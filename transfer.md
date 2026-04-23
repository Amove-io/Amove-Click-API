# Transfer Endpoints

This document provides detailed information about the cloud-to-cloud transfer endpoints exposed by the Amove desktop agent's Click API. Use these endpoints to list existing transfers, queue a new transfer, or cancel a running one.

> This API is bound to `http://localhost:29123` on a machine running the Amove desktop agent. It is not a hosted service.

## Endpoints

1. [Get All Transfers](#get-all-transfers)
2. [Transfer](#transfer)
3. [Cancel Transfer](#cancel-transfer)


## Get All Transfers

Returns the paginated list of transfers associated with the current user's account, filtered by transfer type.

- **URL**: `/transfer/get_all`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| token | string | — | Session token |
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "RequestDate" | Field to sort by |
| descending | boolean | true | Sort direction |
| type | integer (flags) | 15 | Bitwise OR of `TransferType` values (see below) |

The `type` parameter is a bitmask over `TransferType`:

| Value | Meaning |
|---|---|
| `1` | Transfer |
| `2` | Sync |
| `4` | SyncInitTransfer |
| `8` | ManualTransfer |

The default `15` (`1 + 2 + 4 + 8`) returns every transfer type.

### Response

Returns a `DTOCollection<Transfer>`.


## Transfer

Queues a new cloud-to-cloud transfer from a source cloud account/bucket/path to a destination cloud account/bucket/path.

- **URL**: `/transfer/transfer`
- **Method**: POST
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | Session token |

### Request Body

```json
{
  "sourceCloudAccountId": "00000000-0000-0000-0000-000000000000",
  "sourceBucket": "source-bucket",
  "sourceBucketId": "",
  "sourcePath": "folder/file.mov",
  "sourceId": "",
  "destinationCloudAccountId": "11111111-1111-1111-1111-111111111111",
  "destinationBucket": "destination-bucket",
  "destinationBucketId": "",
  "destinationPath": "archive/folder/",
  "destinationId": "",
  "allowSkip": true,
  "keepSourceTree": false
}
```

| Field | Type | Description |
|-------|------|-------------|
| sourceCloudAccountId | string (uuid) | Cloud account the source objects live in |
| sourceBucket | string | Source bucket name |
| sourceBucketId | string | Optional source bucket id (used by parent-style providers) |
| sourcePath | string | Path (key prefix) of the source object or folder |
| sourceId | string | Optional source object id (used by parent-style providers) |
| destinationCloudAccountId | string (uuid) | Cloud account the destination lives in |
| destinationBucket | string | Destination bucket name |
| destinationBucketId | string | Optional destination bucket id |
| destinationPath | string | Path under which the source objects will be placed |
| destinationId | string | Optional destination object id |
| allowSkip | boolean | Skip transfer when a newer version already exists at the destination |
| keepSourceTree | boolean | Preserve the source folder tree under the destination path |

### Response

Returns HTTP 200 when the transfer has been queued. Progress and completion are delivered out-of-band via the WebSocket notification channel.


## Cancel Transfer

Sends a cancel request to a running transfer.

- **URL**: `/transfer/cancel_transfer`
- **Method**: POST
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | Session token |

### Request Body

```json
{
  "id": "00000000-0000-0000-0000-000000000000"
}
```

| Field | Type | Description |
|-------|------|-------------|
| id | string (uuid) | Transfer id to cancel |

### Response

Returns HTTP 200 on success.


## Sample Code

### Queue a Transfer

<details>
<summary>Python</summary>

```python
import requests

response = requests.post(
    "http://localhost:29123/transfer/transfer",
    params={"token": "EXAMPLE_TOKEN"},
    json={
        "sourceCloudAccountId": "00000000-0000-0000-0000-000000000000",
        "sourceBucket": "source-bucket",
        "sourcePath": "folder/",
        "destinationCloudAccountId": "11111111-1111-1111-1111-111111111111",
        "destinationBucket": "destination-bucket",
        "destinationPath": "archive/folder/",
        "allowSkip": True,
        "keepSourceTree": False,
    },
)
print(response.status_code)
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
const res = await fetch(
  "http://localhost:29123/transfer/transfer?token=EXAMPLE_TOKEN",
  {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      sourceCloudAccountId: "00000000-0000-0000-0000-000000000000",
      sourceBucket: "source-bucket",
      sourcePath: "folder/",
      destinationCloudAccountId: "11111111-1111-1111-1111-111111111111",
      destinationBucket: "destination-bucket",
      destinationPath: "archive/folder/",
      allowSkip: true,
      keepSourceTree: false,
    }),
  }
);
console.log(res.status);
```

</details>

### Cancel a Running Transfer

<details>
<summary>Python</summary>

```python
import requests

response = requests.post(
    "http://localhost:29123/transfer/cancel_transfer",
    params={"token": "EXAMPLE_TOKEN"},
    json={"id": "00000000-0000-0000-0000-000000000000"},
)
print(response.status_code)
```

</details>


For error handling, see [Error Model](errors.md).
