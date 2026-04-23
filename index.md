# Amove Click API Documentation

## Introduction

The Click API is a local HTTP service exposed by the Amove desktop agent on the machine where it is installed. It allows scripted integration with drives, backups, transfers, and related desktop-side operations.

> This API is bound to `http://localhost:29123` on a machine running the Amove desktop agent. It is not a hosted service and cannot be reached remotely.

## Table of Contents

1. [Base URL](#base-url)
2. [Authentication](#authentication)
3. [API Endpoints](#api-endpoints)
4. [Error Model](#error-model)
5. [WebSockets](#websockets)
6. [Routes](#routes)
7. [Differences from the Web API](#differences-from-the-web-api)
8. [Getting Started](#getting-started)

## Base URL

```
http://localhost:29123
```

The port is configurable via the agent's `appsettings.HostUrl` configuration; `29123` is the default.

## Authentication

Protected endpoints accept a token as a **query-string parameter** (`?token=...`), not an `Authorization` header.

```
GET http://localhost:29123/user/get_all_users?token=YOUR_TOKEN
```

To obtain a token, use the [Authentication](authentication.md) endpoints — for example, `POST /authentication/login` with username and password. The agent uses the same JWT-based session model as the Amove Web API.

Some endpoints are public (no token required) — primarily signup and initial login flows. These are listed with "Auth Required: No".

## API Endpoints

Endpoints are grouped by controller. Each file below documents one controller:

- [Authentication](authentication.md) — sign up, login, MFA, profile management
- [User](user.md) — user account management
- [UserGroup](usergroup.md) — user group management
- [UsersPermission](userspermission.md) — permissions for projects and shared drives
- [Projects](projects.md) — project management
- [SharedCloudDrive](sharedclouddrive.md) — shared cloud drive management
- [Cloud](cloud.md) — cloud accounts, storage operations, OAuth, P2P transfers
- [Storage](storage.md) — storage access keys and bucket administration
- [Transfer](transfer.md) — cloud-to-cloud transfers
- [TransferHistory](transferhistory.md) — historical transfer records
- [Billing](billing.md) — subscription status and plan management
- [SSO](sso.md) — single sign-on configuration (Okta, SAML, Entra ID)
- [Fastr](fastr.md) — Fastr P2P server management
- [FastrSettings](fastr_settings.md) — Fastr settings per cloud account
- [AdSignal](adsignal.md) — material management and comparison
- [ApiToken](apitoken.md) — API token management
- [AccountLogSetting](accountlogsetting.md) — account log configuration
- [BackupReport](backupreport.md) — backup reporting endpoints
- [DriveReport](drivereport.md) — drive reporting endpoints
- [Management](management.md) — local agent management (drives, backups, mounts, file system)

Cross-cutting documents:

- [WebSockets](websockets.md) — real-time notification channels
- [Error Model](errors.md) — HTTP status codes and application error codes

## Error Model

The Click API uses **HTTP 499** for application-level validation errors, **401 Unauthorized** for authentication failures, and **500 Internal Server Error** for unexpected server errors. See [errors.md](errors.md) for full details and the list of application error codes.

## WebSockets

Two WebSocket endpoints push real-time events from the agent:

| Path | Purpose |
|---|---|
| `/notification` | User-level notifications (progress, announcements, app lifecycle) |
| `/statechange` | File state change events (cache, offline status, sync state) |

See [websockets.md](websockets.md) for frame formats and sample clients.

## Routes

Click API routes are of the form `/<controller>/<action>`. There is no `/api/v1/` prefix.

## Differences from the Web API

| Aspect | Web API (`api.amove.io`) | Click API (`localhost:29123`) |
|---|---|---|
| Host | Hosted service | Local desktop agent |
| Path prefix | `/api/v1/...` | `/<controller>/...` |
| Auth transport | `Authorization: Bearer` header | `?token=` query parameter |
| CORS | Broadly open | Restricted to configured origins |
| Real-time | SignalR hub at `/api/rm` | Native WebSockets at `/notification` and `/statechange` |

## Getting Started

1. Install and sign in to the Amove desktop agent on the target machine.
2. Confirm the agent is running — a GET to `http://localhost:29123/management/agent_status` returns a status object without requiring a token.
3. Obtain a token via the [Authentication](authentication.md) flow.
4. Call any protected endpoint with the token as a query parameter.

```python
import requests

token = "YOUR_TOKEN"
response = requests.get(
    "http://localhost:29123/user/get_all_users",
    params={"token": token, "page": 1, "pagesize": 10},
)
print(response.json())
```
