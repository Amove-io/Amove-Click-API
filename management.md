# Management Endpoints

This document covers the Management endpoints of the Amove Click API. These endpoints drive the desktop agent itself — reading agent status, configuring cloud drives / backups / SMB / NFS mounts, performing local file-system operations, checking for app updates, and broadcasting notifications over WebSocket.

> This API is bound to `http://localhost:29123` on a machine running the Amove desktop agent. It is not a hosted service.

Unlike most other Click API controllers, the majority of `/management/*` endpoints operate on the locally-configured agent state and do not require a `token` query parameter; the few that do are called out explicitly below. The sole exception among commands is `/management/command/start`, which accepts a `token` to load the signed-in user.


## Endpoints

### Agent Status

1. [Get Agent Status](#get-agent-status)
2. [Send Command](#send-command)

### Cloud Backups

3. [Get Cloud Backups](#get-cloud-backups)
4. [Insert Cloud Backup](#insert-cloud-backup)
5. [Update Cloud Backup](#update-cloud-backup)
6. [Delete Cloud Backup](#delete-cloud-backup)
7. [Open Cloud Backup](#open-cloud-backup)

### Cloud Drives

8. [Get Cloud Drives](#get-cloud-drives)
9. [Exist Cloud Drive](#exist-cloud-drive)
10. [Insert Cloud Drive](#insert-cloud-drive)
11. [Update Cloud Drive](#update-cloud-drive)
12. [Change Cloud Drive Cache](#change-cloud-drive-cache)
13. [Open Cloud Drive](#open-cloud-drive)
14. [Delete Cloud Drive](#delete-cloud-drive)
15. [Sync Shared Drive](#sync-shared-drive)
16. [Sync Cloud Drive Objects](#sync-cloud-drive-objects)

### SMB / NFS Mounts

17. [Get SMB Clients](#get-smb-clients)
18. [Insert SMB Client](#insert-smb-client)
19. [Update SMB Client](#update-smb-client)
20. [Delete SMB Client](#delete-smb-client)
21. [Get NFS Clients](#get-nfs-clients)
22. [Insert NFS Client](#insert-nfs-client)
23. [Update NFS Client](#update-nfs-client)
24. [Delete NFS Client](#delete-nfs-client)

### File System Operations

25. [List Local Drives](#list-local-drives)
26. [List Available Local Drives](#list-available-local-drives)
27. [List Local Directories](#list-local-directories)
28. [List Local Files And Directories](#list-local-files-and-directories)
29. [Make Directory](#make-directory)
30. [Get Object Status](#get-object-status)
31. [Get Object Info](#get-object-info)
32. [Get Objects Info](#get-objects-info)
33. [Download Object](#download-object)
34. [Make Object Offline](#make-object-offline)
35. [Remove Object From Cache](#remove-object-from-cache)
36. [Remove Object Offline](#remove-object-offline)
37. [Delete Object](#delete-object)
38. [Read Object](#read-object)
39. [Get Object Stream](#get-object-stream)
40. [Generate Presigned URLs](#generate-presigned-urls)
41. [Get Directory Rename Status](#get-directory-rename-status)

### App Updates

42. [Check Update](#check-update)
43. [Apply Update](#apply-update)

### Notifications

44. [Notify](#notify)


## Agent Status

### Get Agent Status

Returns the current lifecycle state of the agent.

- **URL**: `/management/agent_status`
- **Method**: GET
- **Auth Required**: No

#### Response

Returns the agent status as an integer enum:

```json
1
```

| Value | Name | Meaning |
|-------|------|---------|
| 0 | Created | Agent instance created, user not yet signed in. |
| 1 | Started | Agent running with a signed-in user. |
| 2 | Stopped | Agent stopped. |
| 4 | Paused | Agent paused; drives suspended. |
| 8 | Disposed | Agent disposed; a new login is required. |


### Send Command

Sends a lifecycle command to the agent. `Start` requires a `token` in order to load the signed-in user; the other commands operate on the current in-memory user.

- **URL**: `/management/command/{cmd}`
- **Method**: POST
- **Auth Required**: Only for `Start`

#### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| cmd | integer (enum) | `Start=0`, `Dispose=1`, `Pause=2`, `Resume=4`, `Exit=8`. |

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | Required only for `Start`. JWT token of the signed-in user. |

#### Response

HTTP 200 on success.


## Cloud Backups

A cloud backup is a one-way, upload-only sync from a local folder to a cloud bucket (or prefix).

### Get Cloud Backups

Returns every configured cloud backup on this agent.

- **URL**: `/management/cloudbackup`
- **Method**: GET
- **Auth Required**: No

#### Response

Returns an array of `CloudBackupModel`:

```json
[
  {
    "id": "00000000-0000-0000-0000-000000000000",
    "active": true,
    "name": "Daily photos",
    "localPath": "C:/Users/me/Pictures",
    "prefix": "photos/",
    "storageId": "",
    "storageName": "my-bucket",
    "syncIntervalSeconds": 3600,
    "maximumCacheSize": 0,
    "localCacheEncrypted": false,
    "cloudObjectsEncrypted": false,
    "keepFileAfterUpload": true,
    "sharedDrive": false,
    "rotationalCache": false,
    "permissionType": 2,
    "transferOptions": { "chunkSize": 5242880, "retryCount": 5, "concurrentChunks": 5, "concurrentFiles": 5 },
    "account": { "id": "00000000-0000-0000-0000-000000000000", "cloudType": 1 }
  }
]
```


### Insert Cloud Backup

Registers a new cloud backup. The agent overrides the request with a standard `TransferOptions` profile (5 MB chunks, 5 retries, 5 concurrent chunks and files) and always sets `keepFileAfterUpload = true`.

- **URL**: `/management/cloudbackup`
- **Method**: POST
- **Auth Required**: No

#### Request Body

```json
{
  "name": "Daily photos",
  "localPath": "C:/Users/me/Pictures",
  "prefix": "photos/",
  "storageName": "my-bucket",
  "syncIntervalSeconds": 3600,
  "localCacheEncrypted": false,
  "cloudObjectsEncrypted": false,
  "account": { "id": "00000000-0000-0000-0000-000000000000" }
}
```

#### Response

Returns the stored `CloudBackupModel`.


### Update Cloud Backup

Replaces the configuration of an existing cloud backup.

- **URL**: `/management/cloudbackup/{id}`
- **Method**: PUT
- **Auth Required**: No

#### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | Cloud-backup id. |

#### Request Body

Full `CloudBackupModel`.

#### Response

Returns the updated model.


### Delete Cloud Backup

Removes a cloud backup from the agent.

- **URL**: `/management/cloudbackup/{id}`
- **Method**: DELETE
- **Auth Required**: No

#### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | Cloud-backup id. |

#### Response

HTTP 200 on success.


### Open Cloud Backup

Opens the backup's local folder in the OS file explorer.

- **URL**: `/management/cloudbackup_open/{id}`
- **Method**: POST
- **Auth Required**: No

#### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | Cloud-backup id. |

#### Response

HTTP 200 on success.


## Cloud Drives

A cloud drive is a bidirectional virtual-drive or local-path sync with encryption and cache options.

### Get Cloud Drives

Returns configured cloud drives. When a `path` query is supplied, returns only the drive that owns that path (or an empty array).

- **URL**: `/management/clouddrive`
- **Method**: GET
- **Auth Required**: No

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| path | string | Optional filter — a local filesystem path or mapped drive path. |

#### Response

Returns an array of `CloudDriveModel`:

```json
[
  {
    "id": "00000000-0000-0000-0000-000000000000",
    "sharedDriveId": "",
    "active": true,
    "name": "Team Drive",
    "localPath": "C:/.../cache/00000000-0000-0000-0000-000000000000",
    "permanentLocalPath": "C:/.../cache/00000000-0000-0000-0000-000000000000",
    "prefix": "",
    "storageName": "my-bucket",
    "driveName": "G",
    "syncIntervalSeconds": 60,
    "objectsPerFolder": 1000,
    "maximumCacheSize": 0,
    "allowDelete": true,
    "sharedDrive": false,
    "rotationalCache": false,
    "localCacheEncrypted": false,
    "cloudObjectsEncrypted": false,
    "keepFileAfterUpload": true,
    "permissionType": 2,
    "driveType": 1,
    "syncType": 1,
    "transferOptions": { "chunkSize": 5242880, "retryCount": 5, "concurrentChunks": 5, "concurrentFiles": 5 },
    "account": { "id": "00000000-0000-0000-0000-000000000000", "cloudType": 1 }
  }
]
```

Key enumerations:

- `driveType`: `1 = VirtualDrive`, `2 = LocalPath`.
- `syncType`: `1 = Stream` (files kept in cloud; download on demand), `2 = Mirror` (all files kept locally).
- `permissionType`: `1 = Read`, `2 = ReadWrite`.


### Exist Cloud Drive

Checks whether the provided filesystem path is owned by a configured cloud drive.

- **URL**: `/management/exist_clouddrive`
- **Method**: GET
- **Auth Required**: No

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| path | string | Local filesystem path. |

#### Response

Returns the string `"1"` when a drive owns the path, otherwise `"0"`.


### Insert Cloud Drive

Registers a new cloud drive. The agent resolves and creates the local and permanent cache folders according to the platform:

- **macOS**: `~/Library/Containers/com.amove.AmoveExtension.Ext/Data/Library/Application Support/Cache/{id}`
- **Windows**: `{AgentCwd}/cache/{id}` (when the caller sends `permanentLocalPath = "$$$"` the agent allocates it automatically).

The agent always overrides `keepFileAfterUpload = true` and applies a standard `TransferOptions` profile (5 MB chunks, 5 retries, 5 concurrent chunks and files).

- **URL**: `/management/clouddrive`
- **Method**: POST
- **Auth Required**: No

#### Request Body

```json
{
  "name": "Team Drive",
  "storageName": "my-bucket",
  "prefix": "",
  "driveName": "G",
  "driveType": 1,
  "syncType": 1,
  "permissionType": 2,
  "localCacheEncrypted": false,
  "cloudObjectsEncrypted": false,
  "allowDelete": true,
  "maximumCacheSize": 0,
  "permanentLocalPath": "$$$",
  "account": { "id": "00000000-0000-0000-0000-000000000000" }
}
```

#### Response

Returns the stored `CloudDriveModel` with final `localPath` and `permanentLocalPath` populated.


### Update Cloud Drive

Replaces the configuration of an existing cloud drive.

- **URL**: `/management/clouddrive/{id}`
- **Method**: PUT
- **Auth Required**: No

#### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | Cloud-drive id. |

#### Request Body

Full `CloudDriveModel`.

#### Response

Returns the updated model.


### Change Cloud Drive Cache

Moves or reconfigures the local cache of an existing cloud drive.

- **URL**: `/management/clouddrive_change_cache/{id}`
- **Method**: PUT
- **Auth Required**: No

#### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | Cloud-drive id. |

#### Request Body

`CloudDriveModel` with the new cache fields (`permanentLocalPath`, `localCacheEncrypted`, `maximumCacheSize`, `rotationalCache`).

#### Response

Returns the updated model.


### Open Cloud Drive

Opens the drive's mount point / local folder in the OS file explorer.

- **URL**: `/management/clouddrive_open/{id}`
- **Method**: POST
- **Auth Required**: No

#### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | Cloud-drive id. |

#### Response

HTTP 200 on success.


### Delete Cloud Drive

Unregisters a cloud drive.

- **URL**: `/management/clouddrive/{id}`
- **Method**: DELETE
- **Auth Required**: No

#### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | Cloud-drive id. |

#### Response

HTTP 200 on success.


### Sync Shared Drive

Fetches the list of shared cloud drives assigned to the signed-in user from the Amove Web API and adds/removes local cloud drives to match.

- **URL**: `/management/sync_shareddrive`
- **Method**: POST
- **Auth Required**: No (uses the signed-in user's token)

#### Response

HTTP 200 on success.


### Sync Cloud Drive Objects

Triggers an immediate sync pass on a specific cloud drive starting from the specified path.

- **URL**: `/management/clouddrive/sync/{id}`
- **Method**: GET
- **Auth Required**: No

#### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | Cloud-drive id. |

#### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| path | string | `/` | Prefix within the bucket to sync. When the drive has a configured prefix, the agent uses that instead. |

#### Response

HTTP 200 on success. HTTP 404 when the drive id is unknown.


## SMB / NFS Mounts

### Get SMB Clients

Returns every configured SMB mount on this agent.

- **URL**: `/management/smbclient`
- **Method**: GET
- **Auth Required**: No

#### Response

```json
[
  {
    "id": "00000000-0000-0000-0000-000000000000",
    "name": "Design share",
    "smbShare": "\\\\server\\share",
    "version": "3.0",
    "localPath": "Z:",
    "active": true,
    "isAuthenticated": true,
    "username": "user@example.com",
    "domain": "WORKGROUP"
  }
]
```

Note: `password` is part of the model on write but is omitted from responses.


### Insert SMB Client

Adds a new SMB mount.

- **URL**: `/management/smbclient`
- **Method**: POST
- **Auth Required**: No

#### Request Body

```json
{
  "name": "Design share",
  "smbShare": "\\\\server\\share",
  "version": "3.0",
  "localPath": "Z:",
  "isAuthenticated": true,
  "username": "user@example.com",
  "password": "EXAMPLE_PASSWORD",
  "domain": "WORKGROUP"
}
```

#### Response

Returns the stored `SMBClientModel`.


### Update SMB Client

Replaces the configuration of an SMB mount.

- **URL**: `/management/smbclient/{id}`
- **Method**: PUT
- **Auth Required**: No

#### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | SMB client id. |

#### Request Body

Full `SMBClientModel`.

#### Response

Returns the updated model.


### Delete SMB Client

Removes an SMB mount.

- **URL**: `/management/smbclient/{id}`
- **Method**: DELETE
- **Auth Required**: No

#### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | SMB client id. |

#### Response

HTTP 200 on success.


### Get NFS Clients

Returns every configured NFS mount on this agent.

- **URL**: `/management/nfsclient`
- **Method**: GET
- **Auth Required**: No

#### Response

```json
[
  {
    "id": "00000000-0000-0000-0000-000000000000",
    "name": "Media NAS",
    "nfsShare": "nas.example.com:/media",
    "version": "4",
    "localPath": "/mnt/media",
    "active": true
  }
]
```


### Insert NFS Client

Adds a new NFS mount.

- **URL**: `/management/nfsclient`
- **Method**: POST
- **Auth Required**: No

#### Request Body

```json
{
  "name": "Media NAS",
  "nfsShare": "nas.example.com:/media",
  "version": "4",
  "localPath": "/mnt/media"
}
```

#### Response

Returns the stored `NFSClientModel`.


### Update NFS Client

Replaces the configuration of an NFS mount.

- **URL**: `/management/nfsclient/{id}`
- **Method**: PUT
- **Auth Required**: No

#### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | NFS client id. |

#### Request Body

Full `NFSClientModel`.

#### Response

Returns the updated model.


### Delete NFS Client

Removes an NFS mount.

- **URL**: `/management/nfsclient/{id}`
- **Method**: DELETE
- **Auth Required**: No

#### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | NFS client id. |

#### Response

HTTP 200 on success.


## File System Operations

These endpoints operate on the local filesystem and on objects inside cloud drives. Paths sent to object-level endpoints are validated against the owning drive's cache boundary; a request that escapes the boundary returns HTTP 499 with code `SECURITY`.

### List Local Drives

Returns every logical drive on the machine (Windows drive letters, Unix mount points).

- **URL**: `/management/list_drives`
- **Method**: GET
- **Auth Required**: No

#### Response

```json
["C:\\", "D:\\"]
```


### List Available Local Drives

Returns unused single-letter drive labels — letters `C`–`Z` minus those already in use by the OS or by an existing Amove virtual drive.

- **URL**: `/management/list_available_drives`
- **Method**: GET
- **Auth Required**: No

#### Response

```json
["F", "G", "H"]
```


### List Local Directories

Returns immediate sub-directories of the supplied path. When `path` is empty or `/`, the user's home directory is used; at root, mounted volumes under `/Volumes` are appended.

- **URL**: `/management/list_directories`
- **Method**: GET
- **Auth Required**: No

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| path | string | Local directory path. |

#### Response

```json
["/Users/me/Documents", "/Users/me/Downloads"]
```


### List Local Files And Directories

Returns files and directories at a given path.

- **URL**: `/management/list_files_directories`
- **Method**: GET
- **Auth Required**: No

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| path | string | Local directory path. Empty / `/` falls back to the user home. |

#### Response

```json
[
  { "fileName": "notes.txt", "fullPath": "/Users/me/Documents/notes.txt", "objectType": 1 },
  { "fileName": "Projects", "fullPath": "/Users/me/Documents/Projects", "objectType": 2 }
]
```

| Value | Name |
|-------|------|
| 1 | File |
| 2 | Folder |


### Make Directory

Creates a new folder inside a pre-existing base path. The agent rejects directory names containing path-traversal characters with HTTP 499 / `SECURITY`.

- **URL**: `/management/make_directory`
- **Method**: POST
- **Auth Required**: No

#### Request Body

```json
{
  "path": "/Users/me/Documents",
  "directoryName": "New Folder"
}
```

#### Response

HTTP 200 on success.


### Get Object Status

Returns the lifecycle state of a cached object on a cloud drive.

- **URL**: `/management/object_status`
- **Method**: GET
- **Auth Required**: No

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| path | string | Absolute path to a file inside a cloud drive's cache. |

#### Response

Returns the `FileState` value as an integer:

| Value | Name |
|-------|------|
| 0 | Normal |
| 1 | Uploading |
| 2 | Downloading |
| 3 | Reading |
| 4 | Deleting |
| 5 | Deleted |
| 6 | Ignore |
| 7 | Encrypting |
| 8 | Decrypting |
| 9 | RenameDeleting |
| 10 | Error |

HTTP 404 is returned if the path does not belong to any cloud drive.


### Get Object Info

Returns metadata for a single file in a cloud drive.

- **URL**: `/management/object_info`
- **Method**: GET
- **Auth Required**: No

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| path | string | Absolute path inside a cloud drive. |

#### Response

```json
{
  "driveName": "G",
  "bucketName": "my-bucket",
  "path": "C:/.../folder/file.txt",
  "size": 1234,
  "lastModifiedDate": "2026-01-01T00:00:00Z",
  "state": 0,
  "isDirectory": false,
  "id": ""
}
```


### Get Objects Info

Returns metadata for all objects inside a directory in a cloud drive.

- **URL**: `/management/objects_info`
- **Method**: GET
- **Auth Required**: No

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| path | string | Absolute directory path inside a cloud drive. |

#### Response

Array of `ObjectInfoModel` (same shape as [Get Object Info](#get-object-info)).


### Download Object

Tells the agent to download the specified object from the cloud into the local cache. For folders of known project types (`.postlabteam`, `.fcpbundle`, `.aep`, `.aepx`, `.prproj`) the whole bundle is downloaded.

- **URL**: `/management/object_download`
- **Method**: GET
- **Auth Required**: No

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| path | string | Absolute path inside a cloud drive. |

#### Response

HTTP 200 when the command is queued. HTTP 404 if no drive owns the path.


### Make Object Offline

Marks an object (or all objects in a directory-style bundle) to be downloaded and kept available offline.

- **URL**: `/management/object_make_offline`
- **Method**: GET
- **Auth Required**: No

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| path | string | Absolute path inside a cloud drive. |

#### Response

HTTP 200 on success.


### Remove Object From Cache

Evicts an object from the local cache (when it is in `Temporary` state). The cloud copy is untouched.

- **URL**: `/management/object_remove_from_cache`
- **Method**: POST
- **Auth Required**: No

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| path | string | Absolute path inside a cloud drive. |

#### Response

HTTP 200 on success.


### Remove Object Offline

Reverses "make offline" — removes the local copy, clears any encrypted cache, and re-marks the object as `Temporary`. For directories the operation is applied recursively.

- **URL**: `/management/object_remove_offline`
- **Method**: POST
- **Auth Required**: No

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| path | string | Absolute path inside a cloud drive. |

#### Response

HTTP 200 on success.


### Delete Object

Deletes an object in a cloud drive. The drive must be configured with `allowDelete = true` and the permission type must be `ReadWrite`; otherwise HTTP 403 is returned. Objects currently in `Uploading` state are left in place.

- **URL**: `/management/object_delete`
- **Method**: POST
- **Auth Required**: No

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| path | string | Absolute path inside a cloud drive. |

#### Response

HTTP 200 on success.


### Read Object

Triggers the agent's "read" pipeline on an object — used by the desktop shell integration to materialize a file from cache or request a download.

- **URL**: `/management/read_object`
- **Method**: POST
- **Auth Required**: No

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| path | string | Absolute path inside a cloud drive. |

#### Response

HTTP 200 on success.


### Get Object Stream

Streams a byte range of an object directly from the cloud provider.

- **URL**: `/management/get_object_stream`
- **Method**: GET
- **Auth Required**: No

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| path | string | Absolute path inside a cloud drive. |
| start | integer (long) | First byte to read (inclusive). |
| end | integer (long) | Last byte to read (inclusive). |

#### Response

Streams the bytes as `application/octet-stream`. HTTP 404 when no drive owns the path.


### Generate Presigned URLs

Generates a paired download and upload presigned URL for an object in a cloud drive, each valid for 24 hours.

- **URL**: `/management/generate_presigned_urls`
- **Method**: POST
- **Auth Required**: No

#### Request Body

```json
{
  "driveName": "Team Drive",
  "relativePath": "folder/file.txt",
  "uploadKey": "folder/file.txt"
}
```

| Field | Type | Description |
|-------|------|-------------|
| driveName | string | Human-readable drive name (matches `CloudDriveModel.name`). |
| relativePath | string | Relative path from the root of the drive, used for the download URL. |
| uploadKey | string | Object key within the same bucket, used for the upload URL. |

#### Response

```json
{
  "downloadUrl": "https://...",
  "uploadUrl": "https://..."
}
```

HTTP 404 when no drive matches `driveName`.


### Get Directory Rename Status

Returns `true` when a directory rename in the cloud drive has completed (i.e. the old prefix contains no remaining objects).

- **URL**: `/management/get_directory_rename_status`
- **Method**: GET
- **Auth Required**: No

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| path | string | Absolute directory path inside a cloud drive. |

#### Response

Returns `true` when rename is complete (no remaining objects under the old prefix), `false` otherwise. HTTP 404 when no drive owns the path.


## App Updates

### Check Update

Checks the configured update feed for a newer release of the agent.

- **URL**: `/management/check_update`
- **Method**: GET
- **Auth Required**: No

#### Response

```json
{
  "isUpdateRequired": true,
  "currentVersion": "2.7.0",
  "newVersion": "2.7.1"
}
```


### Apply Update

Triggers the agent's self-update routine. The agent broadcasts an `AppExit` notification to connected clients and terminates so the updater can replace the binary.

- **URL**: `/management/update`
- **Method**: POST
- **Auth Required**: No

#### Response

HTTP 200 once the update has been initiated.


## Notifications

### Notify

Publishes an application notification to every WebSocket client connected to `/notification`. Useful for bridging external UI events into the agent's notification channel.

- **URL**: `/management/notify`
- **Method**: POST
- **Auth Required**: No

#### Request Body

```json
{
  "operation": 2,
  "endpointAddress": "http://localhost:29123/cloud/upload_object",
  "httpMethod": "POST",
  "origion": "my-integration",
  "data": "optional payload string"
}
```

| Field | Type | Description |
|-------|------|-------------|
| operation | integer (enum) | `AppExit=1`, `EndpointExecution=2`. |
| endpointAddress | string | Arbitrary URL describing the origin of the event. |
| httpMethod | string | HTTP verb to report. |
| origion | string | Originating app / integration name (field spelling matches the model). |
| data | string | Optional payload forwarded to WebSocket clients. |

#### Response

HTTP 200 on success.


## Sample Code

### Check agent status

<details>
<summary>Python</summary>

```python
import requests

status = requests.get("http://localhost:29123/management/agent_status").json()
print(status)
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
const res = await fetch("http://localhost:29123/management/agent_status");
console.log(await res.json());
```

</details>


### List configured cloud drives

<details>
<summary>Python</summary>

```python
import requests

drives = requests.get("http://localhost:29123/management/clouddrive").json()
for d in drives:
    print(d["name"], d["localPath"])
```

</details>

<details>
<summary>C#</summary>

```csharp
using var client = new HttpClient();
string json = await client.GetStringAsync("http://localhost:29123/management/clouddrive");
Console.WriteLine(json);
```

</details>


### Start the agent for a signed-in user

<details>
<summary>Python</summary>

```python
import requests

requests.post(
    "http://localhost:29123/management/command/0",
    params={"token": "EXAMPLE_TOKEN"},
)
```

</details>


For error handling, see [Error Model](errors.md).
