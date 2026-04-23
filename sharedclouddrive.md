# SharedCloudDrive Endpoints

This document provides detailed information about the shared cloud drive management endpoints exposed by the Amove desktop agent's Click API. A shared cloud drive is a virtual drive backed by a cloud bucket that is shared across multiple users in the account.

> This API is bound to `http://localhost:29123` on a machine running the Amove desktop agent. It is not a hosted service.

## Endpoints

1. [Get All Shared Cloud Drives](#get-all-shared-cloud-drives)
2. [Get All Shared Cloud Drives With Details](#get-all-shared-cloud-drives-with-details)
3. [Insert Shared Cloud Drive](#insert-shared-cloud-drive)
4. [Update Shared Cloud Drive](#update-shared-cloud-drive)
5. [Delete Shared Cloud Drive](#delete-shared-cloud-drive)


## Get All Shared Cloud Drives

Returns the paginated list of shared cloud drives in the current account.

- **URL**: `/sharedclouddrive/get_all`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| token | string | — | Session token |
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "Name" | Field to sort by |
| descending | boolean | false | Sort direction |
| name | string | null | Optional filter on drive name |

### Response

Returns a `DTOCollection<SharedCloudDrive>`.


## Get All Shared Cloud Drives With Details

Returns shared cloud drives with their assigned projects, users, and user groups inlined.

- **URL**: `/sharedclouddrive/get_all_with_details`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| token | string | — | Session token |
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "Name" | Field to sort by |
| descending | boolean | false | Sort direction |
| name | string | null | Optional filter on drive name |

### Response

Returns a paginated collection of `SharedCloudDriveWithDetailsDTO` entries.


## Insert Shared Cloud Drive

Creates a new shared cloud drive bound to an existing cloud account and bucket.

- **URL**: `/sharedclouddrive/insert`
- **Method**: POST
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | Session token |

### Request Body

```json
{
  "name": "Creative",
  "cloudAccountId": "00000000-0000-0000-0000-000000000000",
  "storageName": "my-shared-bucket",
  "prefix": "projects/creative/",
  "prefixId": "",
  "storageId": "",
  "objectsPerFolder": 10000,
  "allowDelete": false,
  "localCacheEncrypted": false,
  "cloudObjectsEncrypted": false,
  "driveType": 1,
  "syncType": 2,
  "active": true
}
```

Key fields:

| Field | Type | Description |
|-------|------|-------------|
| name | string | Drive display name |
| cloudAccountId | string (uuid) | The cloud account that holds the backing bucket |
| storageName | string | Backing bucket name |
| prefix | string | Optional path prefix within the bucket |
| objectsPerFolder | integer | Max objects listed per folder (default 10000) |
| allowDelete | boolean | When true, local deletions propagate to the cloud |
| localCacheEncrypted | boolean | Encrypt the local cache |
| cloudObjectsEncrypted | boolean | Encrypt objects at rest in the cloud |
| driveType | integer (enum) | `1` VirtualDrive, `2` LocalPath |
| syncType | integer (enum) | `1` Stream (on-demand), `2` Mirror (full copy) |

### Response

Returns the created `SharedCloudDrive` object.


## Update Shared Cloud Drive

Updates an existing shared cloud drive.

- **URL**: `/sharedclouddrive/update`
- **Method**: PUT
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | Session token |

### Request Body

A `SharedCloudDrive` object including its `id`. See [Insert Shared Cloud Drive](#insert-shared-cloud-drive) for the field list.

### Response

Returns the updated `SharedCloudDrive` object.


## Delete Shared Cloud Drive

Soft-deletes a shared cloud drive.

- **URL**: `/sharedclouddrive/delete`
- **Method**: DELETE
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | Shared cloud drive id |
| token | string | Session token |

### Response

Returns HTTP 200 on success.


## Sample Code

### List Shared Cloud Drives

<details>
<summary>Python</summary>

```python
import requests

response = requests.get(
    "http://localhost:29123/sharedclouddrive/get_all",
    params={"token": "EXAMPLE_TOKEN", "page": 1, "pagesize": 20},
)
print(response.json())
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
const res = await fetch(
  "http://localhost:29123/sharedclouddrive/get_all?token=EXAMPLE_TOKEN&page=1&pagesize=20"
);
console.log(await res.json());
```

</details>

### Create a Shared Cloud Drive

<details>
<summary>Python</summary>

```python
import requests

response = requests.post(
    "http://localhost:29123/sharedclouddrive/insert",
    params={"token": "EXAMPLE_TOKEN"},
    json={
        "name": "Creative",
        "cloudAccountId": "00000000-0000-0000-0000-000000000000",
        "storageName": "my-shared-bucket",
        "prefix": "projects/creative/",
        "objectsPerFolder": 10000,
        "allowDelete": False,
        "driveType": 1,
        "syncType": 2,
        "active": True,
    },
)
print(response.json())
```

</details>


For error handling, see [Error Model](errors.md).
