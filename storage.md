# Storage Endpoints

This document provides detailed information about the storage administration endpoints exposed by the Amove desktop agent's Click API. These endpoints manage Amove-issued storage access keys and operate on IDrive-backed buckets.

> This API is bound to `http://localhost:29123` on a machine running the Amove desktop agent. It is not a hosted service.

## Endpoints

1. [Get Storage Keys](#get-storage-keys)
2. [Create Storage Key](#create-storage-key)
3. [Delete Storage Key](#delete-storage-key)
4. [Create Bucket](#create-bucket)
5. [Bucket Status](#bucket-status)
6. [Update Bucket](#update-bucket)
7. [Delete Bucket](#delete-bucket)


## Get Storage Keys

Returns the paginated list of storage access keys issued for the current account.

- **URL**: `/storage/get_storage_key`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| token | string | — | Session token |
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "CreateDate" | Field to sort by |
| descending | boolean | true | Sort direction |

### Response

Returns a `DTOCollection<StorageApiKey>`. Secret keys are never returned on this endpoint — only the public `accessKey`, `name`, `region`, `storageDn`, `storageTier`, and `createDate`.


## Create Storage Key

Creates a new storage access key. The response is the only time the secret key is returned to the caller.

- **URL**: `/storage/create_storage_key`
- **Method**: POST
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | Session token |

### Request Body

```json
{
  "name": "ci-deploy",
  "region": "us-west-1",
  "storageDn": "example.storage.amove.io",
  "permission": 2,
  "allBuckets": false,
  "selectedBuckets": ["my-bucket-1", "my-bucket-2"],
  "cloudAccountId": "00000000-0000-0000-0000-000000000000"
}
```

| Field | Type | Description |
|-------|------|-------------|
| name | string | Human-readable key name |
| region | string | Storage region |
| storageDn | string | Storage endpoint DN |
| permission | integer (enum) | `0` Read, `1` Write, `2` ReadWrite |
| allBuckets | boolean | When true, the key applies to all buckets under the account |
| selectedBuckets | array | Bucket name allow-list when `allBuckets` is false |
| cloudAccountId | string (uuid) | Cloud account id the key belongs to |

### Response

Returns a `ShowStorageApiKey` object which includes the plain-text `secretKey` in addition to the fields of `StorageApiKey`. Store the secret immediately — it cannot be retrieved again.


## Delete Storage Key

Deletes a previously issued storage access key.

- **URL**: `/storage/delete_storage_key`
- **Method**: DELETE
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | Storage key id |
| token | string | Session token |

### Response

Returns HTTP 200 on success.


## Create Bucket

Creates a bucket inside the IDrive storage associated with the given cloud account.

- **URL**: `/storage/create_bucket`
- **Method**: POST
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | Session token |

### Request Body

```json
{
  "cloudAccountId": "00000000-0000-0000-0000-000000000000",
  "bucketName": "my-new-bucket",
  "isPublic": false,
  "isEncrypted": true,
  "versioningEnabled": false,
  "objectLockEnabled": false
}
```

### Response

Returns HTTP 200 on success.


## Bucket Status

Retrieves the current status flags of an existing bucket.

- **URL**: `/storage/bucket_status`
- **Method**: POST
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | Session token |

### Request Body

```json
{
  "cloudAccountId": "00000000-0000-0000-0000-000000000000",
  "bucketName": "my-bucket"
}
```

### Response

Returns a `BucketStatus` object describing encryption, versioning, object-lock, and public-access state for the bucket.


## Update Bucket

Updates configuration flags on an existing bucket.

- **URL**: `/storage/update_bucket`
- **Method**: PUT
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | Session token |

### Request Body

```json
{
  "cloudAccountId": "00000000-0000-0000-0000-000000000000",
  "bucketName": "my-bucket",
  "isPublic": false,
  "isEncrypted": true,
  "versioningEnabled": true
}
```

### Response

Returns HTTP 200 on success.


## Delete Bucket

Deletes a bucket from the IDrive storage associated with the given cloud account.

- **URL**: `/storage/delete_bucket`
- **Method**: DELETE
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| cloudAccountId | string (uuid) | Cloud account id that owns the bucket |
| bucketName | string | Name of the bucket to delete |
| token | string | Session token |

### Response

Returns HTTP 200 on success.


## Sample Code

### Create and Retrieve a Storage Key

<details>
<summary>Python</summary>

```python
import requests

response = requests.post(
    "http://localhost:29123/storage/create_storage_key",
    params={"token": "EXAMPLE_TOKEN"},
    json={
        "name": "ci-deploy",
        "region": "us-west-1",
        "storageDn": "example.storage.amove.io",
        "permission": 2,
        "allBuckets": True,
        "selectedBuckets": [],
        "cloudAccountId": "00000000-0000-0000-0000-000000000000",
    },
)
key = response.json()
print("Access key:", key["accessKey"])
print("Secret key:", key["secretKey"])  # Store this now; cannot be retrieved again.
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
const res = await fetch(
  "http://localhost:29123/storage/create_storage_key?token=EXAMPLE_TOKEN",
  {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      name: "ci-deploy",
      region: "us-west-1",
      storageDn: "example.storage.amove.io",
      permission: 2,
      allBuckets: true,
      selectedBuckets: [],
      cloudAccountId: "00000000-0000-0000-0000-000000000000",
    }),
  }
);
const key = await res.json();
console.log(key.accessKey, key.secretKey);
```

</details>

### Create a Bucket

<details>
<summary>Python</summary>

```python
import requests

response = requests.post(
    "http://localhost:29123/storage/create_bucket",
    params={"token": "EXAMPLE_TOKEN"},
    json={
        "cloudAccountId": "00000000-0000-0000-0000-000000000000",
        "bucketName": "my-new-bucket",
        "isPublic": False,
        "isEncrypted": True,
        "versioningEnabled": False,
        "objectLockEnabled": False,
    },
)
print(response.status_code)
```

</details>


For error handling, see [Error Model](errors.md).
