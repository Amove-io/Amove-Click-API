# WebSockets

This document describes the real-time WebSocket channels exposed by the Amove desktop agent. They are the primary way for a client application to receive progress updates, user-facing notifications, application lifecycle events, and file-state changes without polling.

> This API is bound to `http://localhost:29123` on a machine running the Amove desktop agent. It is not a hosted service.

## Overview

The agent exposes two independent WebSocket upgrade endpoints:

| Path | Stream | What's pushed |
|------|--------|---------------|
| `/notification` | `Notification` | `Progress`, `UserNotification`, `AppNotification`, `AppUpdateProgress` |
| `/statechange` | `StateChangeModel` | File state change events (path, storage name, new state) |

These are plain WebSocket endpoints: a client performs a standard WebSocket upgrade and the server pushes JSON text frames as events occur. There is no request/response protocol on these channels — the client only reads.

When the agent needs to close a connection (for example, on shutdown) it performs a clean close with `WebSocketCloseStatus.NormalClosure` (code `1000`).

A request to either path that is not a valid WebSocket upgrade is rejected with HTTP 400.

## Authentication

The WebSocket endpoints do not require a `token` query parameter. They are reachable on the same localhost binding as the REST API and — like the REST API — rely on the loopback interface and configured CORS origins to restrict access.

## Base URL

Use the `ws://` scheme against the same host/port as the REST API:

```
ws://localhost:29123/notification
ws://localhost:29123/statechange
```

## `/notification` — Notifications Channel

Each frame is a JSON `Notification` envelope:

```json
{
  "Type": 1,
  "Data": { "...": "payload depends on Type" }
}
```

`Type` is one of the `NotificationType` values:

| Value | Name | `Data` payload |
|-------|------|----------------|
| 1 | Progress | Per-file transfer progress event |
| 2 | UserNotification | Human-readable message raised by the agent |
| 4 | AppNotification | Emitted after every non-GET REST call — describes the endpoint, HTTP method, and origin |
| 8 | AppUpdateProgress | Auto-updater progress event |

### Progress payload (Type = 1)

```json
{
  "TransferMode": "Upload | Download | Encrypt | Decrypt",
  "Status": "InProgress | Completed | Failed | ...",
  "StorageType": "string",
  "Path": "string",
  "Bytes": 0,
  "Size": 0,
  "Error": "string | null",
  "Name": "string",
  "StorageName": "string",
  "Speed": 0,
  "Duration": "00:00:00",
  "TransferId": "string",
  "StartTimeUtc": "2026-04-22T12:00:00Z",
  "ProgressTimeUtc": "2026-04-22T12:00:01Z"
}
```

### UserNotification payload (Type = 2)

A human-readable object describing the notification. Shape is driven by the event being reported (e.g. drive lifecycle messages). Treat `Data` as an opaque object that you can render.

### AppNotification payload (Type = 4)

```json
{
  "Operation": "string",
  "EndpointAddress": "string",
  "HttpMethod": "POST | PUT | DELETE",
  "Origin": "string (X-App-Origin header value, may be empty)"
}
```

### AppUpdateProgress payload (Type = 8)

Auto-updater progress reported by NetSparkle — includes download progress and state transitions of the update check.


## `/statechange` — File State Channel

Each frame is a JSON `StateChangeModel`:

```json
{
  "Path": "string",
  "StorageName": "string",
  "State": 0
}
```

`State` uses the `FileState` enumeration:

| Value | Name |
|-------|------|
| 0 | Normal |
| 1 | Temporary |
| 2 | Downloading |
| 4 | Uploading |
| 8 | Deleting |
| 16 | Ignore |
| 32 | RenameDeleting |
| 64 | Deleted |
| 128 | Reading |
| 256 | Error |

The channel fires whenever the agent's local file-state manager transitions a tracked file to a new state. Use this channel when you need to reflect sync progress per file in a UI.


## Sample Code

### Notifications channel

<details>
<summary>Python</summary>

```python
import asyncio
import json
import websockets

async def main():
    async with websockets.connect("ws://localhost:29123/notification") as ws:
        async for frame in ws:
            event = json.loads(frame)
            if event["Type"] == 1:
                p = event["Data"]
                print(f"[progress] {p['Path']} {p['Bytes']}/{p['Size']}")
            elif event["Type"] == 2:
                print(f"[user] {event['Data']}")
            elif event["Type"] == 4:
                print(f"[app] {event['Data']['HttpMethod']} {event['Data']['EndpointAddress']}")
            elif event["Type"] == 8:
                print(f"[update] {event['Data']}")

asyncio.run(main())
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
const ws = new WebSocket("ws://localhost:29123/notification");

ws.addEventListener("message", (msg) => {
  const event = JSON.parse(msg.data);
  switch (event.Type) {
    case 1:
      console.log("progress", event.Data.Path, event.Data.Bytes, "/", event.Data.Size);
      break;
    case 2:
      console.log("user notification", event.Data);
      break;
    case 4:
      console.log("app notification", event.Data.HttpMethod, event.Data.EndpointAddress);
      break;
    case 8:
      console.log("app update progress", event.Data);
      break;
  }
});

ws.addEventListener("close", (e) => {
  console.log("closed", e.code, e.reason);
});
```

</details>

### State-change channel

<details>
<summary>Python</summary>

```python
import asyncio
import json
import websockets

async def main():
    async with websockets.connect("ws://localhost:29123/statechange") as ws:
        async for frame in ws:
            change = json.loads(frame)
            print(f"[{change['StorageName']}] {change['Path']} -> state {change['State']}")

asyncio.run(main())
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
const ws = new WebSocket("ws://localhost:29123/statechange");

ws.addEventListener("message", (msg) => {
  const change = JSON.parse(msg.data);
  console.log(`[${change.StorageName}] ${change.Path} -> state ${change.State}`);
});

ws.addEventListener("close", (e) => {
  console.log("closed", e.code, e.reason);
});
```

</details>

For error handling, see [Error Model](errors.md).
