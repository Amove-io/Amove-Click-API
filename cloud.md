# Cloud Endpoints

This document covers the Cloud endpoints of the AMove Click API. They drive cloud-account management, storage (bucket/object) operations, OAuth authorization flows for third-party providers, and Fastr server-to-server / peer-to-peer / client file transfers running on the local agent. Unless noted, every endpoint requires a `token` query parameter obtained from the [Authentication](authentication.md) flow.

> This API is bound to `http://localhost:29123` on a machine running the Amove desktop agent. It is not a hosted service.


## Endpoints

### Cloud Accounts

1. [Get Cloud Accounts](#get-cloud-accounts)
2. [Get Cloud Account](#get-cloud-account)
3. [Insert Cloud Account](#insert-cloud-account)
4. [Manage Shared Cloud Account](#manage-shared-cloud-account)
5. [Delete Cloud Account](#delete-cloud-account)

### Storage Operations

6. [List Buckets](#list-buckets)
7. [List Objects](#list-objects)
8. [Generate Download URL](#generate-download-url)
9. [Generate Upload URL](#generate-upload-url)
10. [Send Download URL](#send-download-url)
11. [Delete Object](#delete-object)
12. [Create Folder](#create-folder)
13. [Upload Object](#upload-object)
14. [Download Object](#download-object)
15. [Upload Object From Form](#upload-object-from-form)

### Storage Management (Storj)

16. [Add Storage](#add-storage)
17. [Delete Storage](#delete-storage)
18. [Create Bucket](#create-bucket)
19. [Delete Bucket](#delete-bucket)
20. [Get Bucket Info](#get-bucket-info)
21. [Get Cloud Info](#get-cloud-info)

### IDrive Storage

22. [IDrive Add Storage](#idrive-add-storage)
23. [IDrive Delete Storage](#idrive-delete-storage)
24. [IDrive Create Bucket](#idrive-create-bucket)
25. [IDrive Delete Bucket](#idrive-delete-bucket)
26. [IDrive Regions](#idrive-regions)
27. [IDrive Get Daily Usage](#idrive-get-daily-usage)
28. [IDrive Get Usage](#idrive-get-usage)

### OAuth Authorization

29. [Generate Dropbox Authorization URL](#generate-dropbox-authorization-url)
30. [Dropbox Authorization Callback](#dropbox-authorization-callback)
31. [Generate Box Authorization URL](#generate-box-authorization-url)
32. [Box Authorization Callback](#box-authorization-callback)
33. [Generate OneDrive Authorization URL](#generate-onedrive-authorization-url)
34. [OneDrive Authorization Callback](#onedrive-authorization-callback)
35. [Generate Google Drive Authorization URL](#generate-google-drive-authorization-url)
36. [Google Drive Authorization Callback](#google-drive-authorization-callback)

### Fastr Server-to-Server Transfers

37. [Initiate Server Transfer](#initiate-server-transfer)
38. [Get Server Transfer Status](#get-server-transfer-status)
39. [Cancel Server Transfer](#cancel-server-transfer)
40. [Get All Server Transfers](#get-all-server-transfers)

### P2P Client Transfers

41. [Initiate P2P Transfer](#initiate-p2p-transfer)
42. [Get P2P Transfer Status](#get-p2p-transfer-status)
43. [Cancel P2P Transfer](#cancel-p2p-transfer)
44. [Pause P2P Transfer](#pause-p2p-transfer)
45. [Resume P2P Transfer](#resume-p2p-transfer)
46. [Get All P2P Transfers](#get-all-p2p-transfers)

### Client File Transfers

47. [Pause Client Transfer](#pause-client-transfer)
48. [Resume Client Transfer](#resume-client-transfer)
49. [Cancel Client Transfer](#cancel-client-transfer)
50. [Get Client Transfer Status](#get-client-transfer-status)
51. [Get All Client Transfers](#get-all-client-transfers)


## Cloud Accounts

### Get Cloud Accounts

Returns all cloud-account entities visible to the signed-in user (owned plus shared).

- **URL**: `/cloud/accounts`
- **Method**: GET
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | JWT token. |

#### Response

Returns an array of `CloudAccount` objects. Access keys and secret keys are returned obfuscated.


### Get Cloud Account

Returns a single cloud account by id.

- **URL**: `/cloud/account`
- **Method**: GET
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | JWT token. |
| id | string (uuid) | Cloud account id. |

#### Response

Returns a `CloudAccount`.


### Insert Cloud Account

Registers a new cloud account. The agent validates the supplied credentials against the selected provider before saving.

- **URL**: `/cloud/insert_account`
- **Method**: POST
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | JWT token. |

#### Request Body

```json
{
  "name": "My AWS",
  "cloudType": 1,
  "region": "us-east-1",
  "endpoint": "",
  "accessKey": "EXAMPLE_ACCESS_KEY",
  "secretKey": "EXAMPLE_SECRET_KEY",
  "serviceUrl": "",
  "shared": false
}
```

| Field | Type | Description |
|-------|------|-------------|
| name | string | Display name. |
| cloudType | integer (enum) | Provider. Flag enum: `AWS=1`, `Wasabi=2`, `Azure=4`, `S3Generic=8`, `Google=16`, `Frame=32`, `Cloudflare=64`, `Dropbox=128`, `Box=256`, `OneDrive=512`, `GoogleDrive=1024`, `LocalHttp=2048`, `FastrServer=4096`, `Atomosphere=8192`. |
| region | string | Region code where applicable. |
| endpoint | string | Custom endpoint for S3-generic / self-hosted providers. |
| accessKey | string | Provider access key. |
| secretKey | string | Provider secret key. |
| serviceUrl | string | Service base URL for providers like LocalHttp / FastrServer. |
| shared | boolean | When true, the account is shared with the whole Amove account. |

#### Response

Returns the stored `CloudAccount`.


### Manage Shared Cloud Account

Updates sharing state on an existing cloud account.

- **URL**: `/cloud/manage_share`
- **Method**: PUT
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | JWT token. |

#### Request Body

The full `CloudAccount` to update. Only sharing-related fields are persisted.

#### Response

HTTP 200 with no body on success.


### Delete Cloud Account

Removes a cloud account owned by the current account.

- **URL**: `/cloud/delete`
- **Method**: DELETE
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | JWT token. |
| id | string (uuid) | Cloud account id. |

#### Response

Returns `true` on success.


## Storage Operations

### List Buckets

Lists buckets / root containers available on a cloud account.

- **URL**: `/cloud/list_buckets`
- **Method**: POST
- **Auth Required**: Yes (token not required on this action — see note)

#### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| includeRegion | boolean | false | When true, the region is resolved and included for each bucket. |

#### Request Body

The full `CloudAccount` whose buckets should be listed.

#### Response

Returns a provider-specific `ICloudStorageCollection`. A typical shape:

```json
{
  "data": [
    { "name": "my-bucket", "region": "us-east-1", "createDate": "2026-01-01T00:00:00Z" }
  ],
  "continuationToken": ""
}
```


### List Objects

Lists the objects under a prefix in a bucket. Supports continuation for large listings.

- **URL**: `/cloud/list_objects`
- **Method**: POST
- **Auth Required**: Yes

#### Request Body

```json
{
  "account": { "id": "00000000-0000-0000-0000-000000000000", "cloudType": 1, "accessKey": "EXAMPLE_ACCESS_KEY", "secretKey": "EXAMPLE_SECRET_KEY" },
  "request": {
    "id": "",
    "bucketName": "my-bucket",
    "path": "photos/",
    "continuationToken": "",
    "count": 1000
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| account | object (`CloudAccount`) | Target cloud account. |
| request.id | string | Optional target bucket or folder id (used by providers such as Box / Frame / GoogleDrive). |
| request.bucketName | string | Bucket or root container name. |
| request.path | string | Prefix to list. Defaults to `/`. |
| request.continuationToken | string | Continuation token from a previous call. |
| request.count | integer | Maximum objects to return (default `1000`). |

#### Response

Returns an `ICloudStorageObjectCollection` containing `data[]` (objects/folders) and a `continuationToken` for the next page.


### Generate Download URL

Builds a presigned download URL for a single object.

- **URL**: `/cloud/generate_download_url`
- **Method**: POST
- **Auth Required**: Yes

#### Request Body

```json
{
  "cloudAccount": { "id": "00000000-0000-0000-0000-000000000000" },
  "id": "",
  "storageName": "my-bucket",
  "key": "photos/file.jpg",
  "expireHours": 24,
  "forceToDownload": false
}
```

| Field | Type | Description |
|-------|------|-------------|
| cloudAccount | object (`CloudAccount`) | Cloud account holding the object. |
| id | string | Optional provider-specific object id. |
| storageName | string | Bucket name. |
| key | string | Object key. |
| expireHours | integer | URL validity in hours (default `24`). |
| forceToDownload | boolean | When true, the URL is generated with `Content-Disposition: attachment`. |

#### Response

Returns the signed URL as a JSON string.


### Generate Upload URL

Builds a presigned upload URL for a single object key.

- **URL**: `/cloud/generate_upload_url`
- **Method**: POST
- **Auth Required**: Yes

#### Request Body

```json
{
  "cloudAccount": { "id": "00000000-0000-0000-0000-000000000000" },
  "id": "",
  "storageName": "my-bucket",
  "key": "uploads/file.bin",
  "expireHours": 24,
  "contentType": "application/octet-stream"
}
```

| Field | Type | Description |
|-------|------|-------------|
| cloudAccount | object (`CloudAccount`) | Cloud account to upload into. |
| storageName | string | Bucket name. |
| key | string | Target object key. |
| expireHours | integer | URL validity in hours (default `24`). |
| contentType | string | Content-Type the client must send with the PUT (e.g. `application/octet-stream`). |

#### Response

Returns the signed URL as a JSON string.


### Send Download URL

Generates a presigned download URL and emails it to the supplied address.

- **URL**: `/cloud/send_download_url`
- **Method**: POST
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | JWT token. |

#### Request Body

```json
{
  "email": "user@example.com",
  "cloudAccountId": "00000000-0000-0000-0000-000000000000",
  "id": "",
  "storageName": "my-bucket",
  "key": "photos/file.jpg",
  "expireHours": 24,
  "note": "Please review"
}
```

#### Response

HTTP 200 on success.


### Delete Object

Deletes an object from cloud storage.

- **URL**: `/cloud/delete_object`
- **Method**: POST
- **Auth Required**: Yes

#### Request Body

```json
{
  "cloudAccount": { "id": "00000000-0000-0000-0000-000000000000" },
  "storageName": "my-bucket",
  "key": "photos/file.jpg",
  "id": ""
}
```

#### Response

HTTP 200 on success.


### Create Folder

Creates a folder (or folder-marker object) in a bucket.

- **URL**: `/cloud/create_folder`
- **Method**: POST
- **Auth Required**: Yes

#### Request Body

```json
{
  "cloudAccount": { "id": "00000000-0000-0000-0000-000000000000" },
  "id": "",
  "storageName": "my-bucket",
  "path": "photos/",
  "folderName": "vacation-2026"
}
```

#### Response

HTTP 200 on success.


### Upload Object

Starts a background upload of a local file to cloud storage. Returns a `transferId` that can be used with the transfer control endpoints.

- **URL**: `/cloud/upload_object`
- **Method**: POST
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | JWT token. Optional — when omitted, the agent uses the currently signed-in user's token. |

#### Request Body

```json
{
  "cloudAccount": { "id": "00000000-0000-0000-0000-000000000000", "cloudType": 1 },
  "storageName": "my-bucket",
  "uploadPath": "uploads",
  "localFilePath": "C:/data/file.bin",
  "id": "",
  "name": ""
}
```

| Field | Type | Description |
|-------|------|-------------|
| cloudAccount | object (`CloudAccount`) | Target cloud account. |
| storageName | string | Bucket name. |
| uploadPath | string | Destination folder in the bucket. |
| localFilePath | string | Absolute local path. The file name is used as the object name. |
| id | string | Optional transfer id. Ignored for `LocalHttp`/`FastrServer` uploads (a new id is always generated). |

#### Response

```json
{
  "transferId": "00000000-0000-0000-0000-000000000000"
}
```

Progress is also broadcast on the `/notification` WebSocket channel. For `LocalHttp` or `FastrServer` targets the transfer is trackable via the [client transfer](#client-file-transfers) endpoints; for other providers the upload is fire-and-forget.


### Download Object

Starts a background download of an object into a local path. Returns a `transferId`.

- **URL**: `/cloud/download_object`
- **Method**: POST
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | JWT token. Optional — when omitted, the agent uses the signed-in user's token. |

#### Request Body

```json
{
  "cloudAccount": { "id": "00000000-0000-0000-0000-000000000000", "cloudType": 1 },
  "storageName": "my-bucket",
  "downloadPath": "C:/downloads",
  "objectKey": "photos/file.jpg",
  "id": "",
  "name": "file.jpg",
  "objectType": 1
}
```

| Field | Type | Description |
|-------|------|-------------|
| cloudAccount | object (`CloudAccount`) | Source cloud account. |
| storageName | string | Bucket name. |
| downloadPath | string | Absolute local folder. |
| objectKey | string | Cloud object key to download. |
| objectType | integer (enum) | `1` = File, `2` = Directory. |

#### Response

```json
{
  "transferId": "00000000-0000-0000-0000-000000000000"
}
```

Progress is broadcast on the `/notification` WebSocket channel.


### Upload Object From Form

Accepts a multipart form upload (browser-side file picker). The file is streamed to disk under the agent's `tmp/{cloudAccountId}/{storageName}/` directory and the upload begins immediately.

- **URL**: `/cloud/upload_object_from_form`
- **Method**: POST
- **Content-Type**: `multipart/form-data`
- **Auth Required**: No token is accepted on this endpoint — the currently signed-in user is used.

#### Form Fields

| Field | Type | Description |
|-------|------|-------------|
| cloudAccountJson | string | JSON-serialized `CloudAccount`. |
| storageName | string | Bucket name. |
| uploadPath | string | Destination folder. |
| file | binary | File content. |
| id | string | Optional transfer id. |

#### Response

HTTP 200 on success. HTTP 400 is returned when `file` is missing or empty.


## Storage Management (Storj)

These endpoints target the built-in Amove-managed storage (Storj).

### Add Storage

Provisions a bucket under the signed-in account.

- **URL**: `/cloud/add_storage`
- **Method**: POST
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | JWT token. |

#### Request Body

```json
{
  "cloudAccountName": "Amove Storage",
  "bucketName": "project-alpha",
  "shared": false
}
```

#### Response

Returns the created `CloudAccount`.


### Delete Storage

Deletes an Amove-managed storage cloud account.

- **URL**: `/cloud/delete_storage`
- **Method**: DELETE
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | JWT token. |
| id | string (uuid) | Cloud account id. |

#### Response

Returns `true` on success.


### Create Bucket

Creates a new bucket on the current user's Amove storage.

- **URL**: `/cloud/create_bucket`
- **Method**: POST
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | JWT token. |
| bucketName | string | Bucket to create. |

#### Response

HTTP 200 on success.


### Delete Bucket

Deletes a bucket from the current user's Amove storage.

- **URL**: `/cloud/delete_bucket`
- **Method**: DELETE
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | JWT token. |
| bucketName | string | Bucket to delete. |

#### Response

HTTP 200 on success.


### Get Bucket Info

Returns metadata (usage, object count, region, etc.) for an Amove-storage bucket.

- **URL**: `/cloud/get_bucket_info`
- **Method**: GET
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | JWT token. |
| bucketName | string | Bucket name. |

#### Response

Returns a `BucketInfoResponse` object.


### Get Cloud Info

Returns account-level Amove-storage metrics (total quota, used bytes, bucket count).

- **URL**: `/cloud/get_cloud_info`
- **Method**: GET
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | JWT token. |

#### Response

Returns a `CloudInfoResponse` object.


## IDrive Storage

Endpoints for managing IDrive e2 storage accounts and buckets.

### IDrive Add Storage

Registers an IDrive storage account in the requested region and (optionally) creates its first bucket.

- **URL**: `/cloud/idrive_add_storage`
- **Method**: POST
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | JWT token. |

#### Request Body

```json
{
  "cloudAccountName": "IDrive e2",
  "bucketName": "project-alpha",
  "regionCode": "us-ca",
  "storageTier": 0,
  "shared": false
}
```

| Field | Type | Description |
|-------|------|-------------|
| storageTier | integer (enum) | `All=0`, `Perform=1`, `Global=2`, `Scale=4`, `Archive=8`, `AmoveConnection=16`. |
| regionCode | string | IDrive region code — see [IDrive Regions](#idrive-regions). |

#### Response

Returns the stored `CloudAccount`.


### IDrive Delete Storage

Removes an IDrive storage account.

- **URL**: `/cloud/idrive_delete_storage`
- **Method**: DELETE
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | JWT token. |
| id | string (uuid) | Cloud account id. |

#### Response

Returns `true` on success.


### IDrive Create Bucket

Creates a bucket inside an existing IDrive account.

- **URL**: `/cloud/idrive_create_bucket`
- **Method**: POST
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | JWT token. |
| cloudAccountId | string (uuid) | IDrive cloud account id. |
| bucketName | string | Bucket to create. |

#### Response

HTTP 200 on success.


### IDrive Delete Bucket

Deletes a bucket inside an IDrive account.

- **URL**: `/cloud/idrive_delete_bucket`
- **Method**: DELETE
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | JWT token. |
| cloudAccountId | string (uuid) | IDrive cloud account id. |
| bucketName | string | Bucket to delete. |

#### Response

HTTP 200 on success.


### IDrive Regions

Returns the list of IDrive regions available to the account.

- **URL**: `/cloud/idrive_regions`
- **Method**: GET
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | JWT token. |

#### Response

```json
[
  { "name": "Los Angeles", "code": "us-la" },
  { "name": "Virginia", "code": "us-va" }
]
```


### IDrive Get Daily Usage

Returns the previous day's IDrive byte usage for the account, optionally filtered by storage tier.

- **URL**: `/cloud/idrive_get_daily_usage`
- **Method**: GET
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| token | string | — | JWT token. |
| storageTier | integer (enum) | 0 (All) | Filter by tier. See [IDrive Add Storage](#idrive-add-storage). |

#### Response

Returns a `double` (bytes).


### IDrive Get Usage

Returns the current cumulative IDrive usage for the account, optionally filtered by storage tier.

- **URL**: `/cloud/idrive_get_usage`
- **Method**: GET
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| token | string | — | JWT token. |
| storageTier | integer (enum) | 0 (All) | Filter by tier. |

#### Response

Returns a `double` (bytes).


## OAuth Authorization

Each third-party provider uses a pair of endpoints: one to obtain the authorization URL the user should visit, and one the client invokes after the redirect with the authorization code.

### Generate Dropbox Authorization URL

- **URL**: `/cloud/generate_dropbox_authorization_url`
- **Method**: POST
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | JWT token. |

#### Request Body

```json
{
  "redirectURL": "http://localhost:3000/oauth/dropbox",
  "cloudName": "My Dropbox",
  "shared": false
}
```

#### Response

Returns the Dropbox authorization URL as a JSON string.


### Dropbox Authorization Callback

Completes the Dropbox OAuth flow and stores a new cloud account.

- **URL**: `/cloud/dropbox_authorization_callback`
- **Method**: POST
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | JWT token. |

#### Request Body

```json
{
  "code": "EXAMPLE_AUTH_CODE",
  "state": "opaque-state",
  "redirectURL": "http://localhost:3000/oauth/dropbox"
}
```

#### Response

Returns the created `CloudAccount`.


### Generate Box Authorization URL

- **URL**: `/cloud/generate_box_authorization_url`
- **Method**: POST
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | JWT token. |

#### Request Body

```json
{
  "redirectURL": "http://localhost:3000/oauth/box",
  "cloudName": "My Box",
  "shared": false
}
```

#### Response

Returns the Box authorization URL as a JSON string.


### Box Authorization Callback

Completes the Box OAuth flow and stores a new cloud account.

- **URL**: `/cloud/box_authorization_callback`
- **Method**: POST
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | JWT token. |

#### Request Body

```json
{
  "code": "EXAMPLE_AUTH_CODE",
  "state": "opaque-state",
  "redirectURL": "http://localhost:3000/oauth/box"
}
```

#### Response

Returns the created `CloudAccount`.


### Generate OneDrive Authorization URL

- **URL**: `/cloud/generate_onedrive_authorization_url`
- **Method**: POST
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | JWT token. |

#### Request Body

```json
{
  "cloudName": "My OneDrive",
  "redirectURL": "http://localhost:3000/oauth/onedrive",
  "shared": false
}
```

#### Response

Returns the OneDrive authorization URL as a JSON string.


### OneDrive Authorization Callback

Completes the OneDrive OAuth flow and stores a new cloud account.

- **URL**: `/cloud/onedrive_authorization_callback`
- **Method**: POST
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | JWT token. |

#### Request Body

```json
{
  "code": "EXAMPLE_AUTH_CODE",
  "state": "opaque-state",
  "error": "",
  "errorDescription": "",
  "redirectURL": "http://localhost:3000/oauth/onedrive"
}
```

#### Response

Returns the created `CloudAccount`.


### Generate Google Drive Authorization URL

- **URL**: `/cloud/generate_googledrive_authorization_url`
- **Method**: POST
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | JWT token. |

#### Request Body

```json
{
  "cloudName": "My Google Drive",
  "redirectURL": "http://localhost:3000/oauth/googledrive",
  "shared": false
}
```

#### Response

Returns the Google Drive authorization URL as a JSON string.


### Google Drive Authorization Callback

Completes the Google Drive OAuth flow and stores a new cloud account.

- **URL**: `/cloud/googledrive_authorization_callback`
- **Method**: POST
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | JWT token. |

#### Request Body

```json
{
  "code": "EXAMPLE_AUTH_CODE",
  "state": "opaque-state",
  "error": "",
  "errorDescription": "",
  "redirectURL": "http://localhost:3000/oauth/googledrive"
}
```

#### Response

Returns the created `CloudAccount`.


## Fastr Server-to-Server Transfers

Server-to-server transfers move an object directly between two Fastr-compatible servers via the local agent, which streams the bytes through. The call is non-blocking: the agent returns a `transferId` and continues in the background.

### Initiate Server Transfer

Starts a server-to-server transfer. The agent HEADs the source URL, then streams the body to the target URL in 8 MB chunks.

- **URL**: `/cloud/server_transfer/initiate`
- **Method**: POST
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | JWT token. |

#### Request Body

```json
{
  "sourceServerUrl": "https://source.example.com",
  "targetServerUrl": "https://target.example.com",
  "sourceBucket": "bucket-a",
  "sourcePath": "files/video.mp4",
  "targetBucket": "bucket-b",
  "targetPath": "files/video.mp4",
  "sourceAccountId": "00000000-0000-0000-0000-000000000000",
  "targetAccountId": "00000000-0000-0000-0000-000000000000",
  "metadata": {}
}
```

All six URL/bucket/path fields are required; the agent rejects the call with HTTP 400 when any are missing.

#### Response

HTTP 202 Accepted:

```json
{
  "transferId": "00000000-0000-0000-0000-000000000000",
  "message": "Server transfer initiated",
  "status": "pending"
}
```


### Get Server Transfer Status

Returns the latest known status of a server transfer.

- **URL**: `/cloud/server_transfer/{transferId}/status`
- **Method**: GET
- **Auth Required**: Yes

#### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| transferId | string | Id returned by [Initiate Server Transfer](#initiate-server-transfer). |

#### Response

Returns a `FastrServerTransferStatus`:

```json
{
  "transferId": "00000000-0000-0000-0000-000000000000",
  "status": "running",
  "sourceServerUrl": "https://source.example.com",
  "targetServerUrl": "https://target.example.com",
  "sourceBucket": "bucket-a",
  "sourcePath": "files/video.mp4",
  "targetBucket": "bucket-b",
  "targetPath": "files/video.mp4",
  "startTime": "2026-01-01T00:00:00Z",
  "endTime": "0001-01-01T00:00:00",
  "bytesTransferred": 12582912,
  "totalBytes": 104857600,
  "progress": 12,
  "errorMessage": ""
}
```

`status` is one of `pending`, `running`, `completed`, `cancelled`, `failed`. Returns HTTP 404 when the id is unknown.


### Cancel Server Transfer

Cancels a running or pending server transfer.

- **URL**: `/cloud/server_transfer/{transferId}`
- **Method**: DELETE
- **Auth Required**: Yes

#### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| transferId | string | Transfer id. |

#### Response

HTTP 200 on success. HTTP 400 is returned when the transfer is already `completed` or `cancelled`.


### Get All Server Transfers

Returns every server transfer tracked by the agent (active and terminal).

- **URL**: `/cloud/server_transfer/all`
- **Method**: GET
- **Auth Required**: Yes

#### Response

Returns an array of `FastrServerTransferStatus`.


## P2P Client Transfers

P2P transfers ask a source Fastr-compatible server to push an object directly to another server using Fastr's peer-to-peer (with TURN fallback) transport. The agent acts as the orchestrator and forwards status updates to WebSocket clients.

### Initiate P2P Transfer

Initiates a peer-to-peer transfer between two servers.

- **URL**: `/cloud/p2p_transfer/initiate`
- **Method**: POST
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | JWT token. |

#### Request Body

```json
{
  "sourceAccount": { "id": "00000000-0000-0000-0000-000000000000", "cloudType": 2048, "serviceUrl": "https://source.example.com", "accessKey": "EXAMPLE_ACCESS_KEY", "secretKey": "EXAMPLE_SECRET_KEY" },
  "targetAccount": { "id": "00000000-0000-0000-0000-000000000000" },
  "sourceBucket": "bucket-a",
  "sourcePath": "files/video.mp4",
  "targetServerUrl": "https://target.example.com",
  "targetBucket": "bucket-b",
  "targetPath": "files/video.mp4",
  "overwrite": false,
  "chunkSizeMB": 64
}
```

| Field | Type | Description |
|-------|------|-------------|
| sourceAccount | object (`CloudAccount`) | Required. Account on the source server. |
| targetAccount | object (`CloudAccount`) | Optional. When provided, its id is used as the target-side credential on TURN fallback. |
| sourceBucket / sourcePath | string | Required. |
| targetServerUrl / targetBucket / targetPath | string | Required. |
| overwrite | boolean | Overwrite the target object if it exists. Default `false`. |
| chunkSizeMB | integer | Chunk size in MB (default `64`). |

#### Response

```json
{
  "transferId": "00000000-0000-0000-0000-000000000000",
  "message": "P2P transfer initiated successfully"
}
```


### Get P2P Transfer Status

Queries the source server for up-to-date P2P transfer status and returns the merged view.

- **URL**: `/cloud/p2p_transfer/{transferId}/status`
- **Method**: GET
- **Auth Required**: Yes

#### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| transferId | string | Transfer id. |

#### Response

Returns a `P2PTransferStatusResponse`:

```json
{
  "transferId": "00000000-0000-0000-0000-000000000000",
  "status": "running",
  "totalBytes": 104857600,
  "transferredBytes": 41943040,
  "progressPercentage": 40,
  "startTime": "2026-01-01T00:00:00Z",
  "endTime": null,
  "errorMessage": null,
  "sourceBucket": "bucket-a",
  "sourcePath": "files/video.mp4",
  "targetServerUrl": "https://target.example.com",
  "targetBucket": "bucket-b",
  "targetPath": "files/video.mp4",
  "checksumVerified": false
}
```

`status` is one of `pending`, `running`, `paused`, `completed`, `failed`, `cancelled`.


### Cancel P2P Transfer

Cancels a P2P transfer on both the local agent and the source server.

- **URL**: `/cloud/p2p_transfer/{transferId}/cancel`
- **Method**: POST
- **Auth Required**: Yes

#### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| transferId | string | Transfer id. |

#### Response

```json
{
  "success": true,
  "message": "Transfer cancelled successfully",
  "serverCancelled": true
}
```


### Pause P2P Transfer

Pauses a running P2P transfer. The agent verifies on the source server that the transfer is paused before returning success.

- **URL**: `/cloud/p2p_transfer/{transferId}/pause`
- **Method**: POST
- **Auth Required**: Yes

#### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| transferId | string | Transfer id. |

#### Response

HTTP 200:

```json
{
  "success": true,
  "message": "Transfer paused successfully",
  "serverPaused": true,
  "serverStatus": "paused"
}
```

Returns HTTP 400 when the transfer is not in `running` or `initiating` state, or when pause could not be confirmed.


### Resume P2P Transfer

Resumes a paused P2P transfer.

- **URL**: `/cloud/p2p_transfer/{transferId}/resume`
- **Method**: POST
- **Auth Required**: Yes

#### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| transferId | string | Transfer id. |

#### Response

HTTP 200:

```json
{
  "success": true,
  "message": "Transfer resumed successfully",
  "serverResumed": true,
  "serverStatus": "running"
}
```

Returns HTTP 400 when the transfer is not paused or the source-server resume cannot be confirmed.


### Get All P2P Transfers

Returns every P2P transfer the agent has tracked in the current process lifetime.

- **URL**: `/cloud/p2p_transfer/all`
- **Method**: GET
- **Auth Required**: Yes

#### Response

Returns an array of `FastrServerTransferStatus`.


## Client File Transfers

Client transfers are the upload/download background jobs started by [Upload Object](#upload-object) and [Download Object](#download-object). They apply only to `LocalHttp` and `FastrServer` targets (those are the cases that register a controllable transfer id).

### Pause Client Transfer

Pauses an in-progress client-side upload or download.

- **URL**: `/cloud/client_transfer/{transferId}/pause`
- **Method**: POST
- **Auth Required**: Yes

#### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| transferId | string | Transfer id returned by upload or download. |

#### Response

HTTP 200 on success. HTTP 400 when the transfer is not in `running` or `initiating` state.


### Resume Client Transfer

Resumes a paused client-side transfer.

- **URL**: `/cloud/client_transfer/{transferId}/resume`
- **Method**: POST
- **Auth Required**: Yes

#### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| transferId | string | Transfer id. |

#### Response

HTTP 200 on success. HTTP 400 when the transfer is not paused.


### Cancel Client Transfer

Cancels a client-side transfer.

- **URL**: `/cloud/client_transfer/{transferId}/cancel`
- **Method**: POST
- **Auth Required**: Yes

#### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| transferId | string | Transfer id. |

#### Response

HTTP 200 on success. HTTP 400 when the transfer is already `completed` or `cancelled`.


### Get Client Transfer Status

Returns the current status of a client-side transfer, fetching the latest counters from the source server when available.

- **URL**: `/cloud/client_transfer/{transferId}/status`
- **Method**: GET
- **Auth Required**: Yes

#### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| transferId | string | Transfer id. |

#### Response

```json
{
  "transferId": "00000000-0000-0000-0000-000000000000",
  "status": "running",
  "transferredBytes": 41943040,
  "totalBytes": 104857600,
  "startTime": "2026-01-01T00:00:00Z",
  "endTime": null,
  "errorMessage": null,
  "pausedDuration": null
}
```


### Get All Client Transfers

Returns every client transfer (upload / download) the agent is tracking.

- **URL**: `/cloud/client_transfer/all`
- **Method**: GET
- **Auth Required**: Yes

#### Response

Returns an array of transfer summary objects — each contains `transferId`, `status`, `transferredBytes`, `totalBytes`, `startTime`, `endTime`, `errorMessage`, `sourceBucket`, `sourcePath`, `targetBucket`, `targetPath`, `progress`.


## Sample Code

### Upload an object

<details>
<summary>Python</summary>

```python
import requests

response = requests.post(
    "http://localhost:29123/cloud/upload_object",
    params={"token": "EXAMPLE_TOKEN"},
    json={
        "cloudAccount": {"id": "00000000-0000-0000-0000-000000000000", "cloudType": 1},
        "storageName": "my-bucket",
        "uploadPath": "uploads",
        "localFilePath": "/home/me/file.bin",
    },
)
print(response.json()["transferId"])
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
const res = await fetch(
  "http://localhost:29123/cloud/upload_object?token=EXAMPLE_TOKEN",
  {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      cloudAccount: { id: "00000000-0000-0000-0000-000000000000", cloudType: 1 },
      storageName: "my-bucket",
      uploadPath: "uploads",
      localFilePath: "C:/data/file.bin",
    }),
  },
);
const { transferId } = await res.json();
console.log(transferId);
```

</details>

<details>
<summary>C#</summary>

```csharp
using System.Net.Http.Json;

using var client = new HttpClient();
var res = await client.PostAsJsonAsync(
    "http://localhost:29123/cloud/upload_object?token=EXAMPLE_TOKEN",
    new
    {
        cloudAccount = new { id = "00000000-0000-0000-0000-000000000000", cloudType = 1 },
        storageName = "my-bucket",
        uploadPath = "uploads",
        localFilePath = @"C:\data\file.bin",
    });
Console.WriteLine(await res.Content.ReadAsStringAsync());
```

</details>


### Initiate a P2P transfer and poll status

<details>
<summary>Python</summary>

```python
import requests
import time

base = "http://localhost:29123/cloud"
token = "EXAMPLE_TOKEN"

init = requests.post(
    f"{base}/p2p_transfer/initiate",
    params={"token": token},
    json={
        "sourceAccount": {
            "id": "00000000-0000-0000-0000-000000000000",
            "cloudType": 2048,
            "serviceUrl": "https://source.example.com",
            "accessKey": "EXAMPLE_ACCESS_KEY",
            "secretKey": "EXAMPLE_SECRET_KEY",
        },
        "sourceBucket": "bucket-a",
        "sourcePath": "files/video.mp4",
        "targetServerUrl": "https://target.example.com",
        "targetBucket": "bucket-b",
        "targetPath": "files/video.mp4",
    },
).json()

transfer_id = init["transferId"]
while True:
    status = requests.get(f"{base}/p2p_transfer/{transfer_id}/status").json()
    print(status["status"], status["progressPercentage"])
    if status["status"] in ("completed", "failed", "cancelled"):
        break
    time.sleep(1)
```

</details>


For error handling, see [Error Model](errors.md).
