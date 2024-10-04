# Storage

The Storage endpoints allow you to manage storage keys, create and manage buckets, and perform various storage-related operations.

## Endpoints

### Get Storage Keys

Retrieve all storage keys associated with the authenticated user.

- **URL**: `/Storage/get_storage_key`
- **Method**: GET
- **Auth Required**: Yes

#### Query Parameters

- `token`: string (required)
- `page`: integer (default: 1)
- `pagesize`: integer (default: 50)
- `sortfield`: string (default: "CreateDate")
- `descending`: boolean (default: true)

#### Response

A StorageApiKeyDTOCollection containing an array of StorageApiKey objects:

```json
{
  "data": [
    {
      "id": "uuid",
      "userId": "uuid",
      "accountId": "uuid",
      "name": "string",
      "accessKey": "string",
      "description": "string",
      "region": "string",
      "storageDn": "string",
      "storageTier": 0,
      "createDate": "2023-05-05T12:00:00Z"
    }
  ],
  "total": 0,
  "options": {
    "pageSize": 0,
    "page": 0,
    "sort": [
      {
        "field": "string",
        "descending": true
      }
    ]
  }
}
```

### Create Storage Key

Create a new storage key.

- **URL**: `/Storage/create_storage_key`
- **Method**: POST
- **Auth Required**: Yes

#### Query Parameters

- `token`: string (required)

#### Request Body

A StorageCreateAccessRequest object:

```json
{
  "name": "string",
  "region": "string",
  "storageDn": "string",
  "permission": 0,
  "allBuckets": true,
  "selectedBuckets": [
    "string"
  ],
  "cloudAccountId": "uuid"
}
```

#### Response

A ShowStorageApiKey object containing the created storage key information.

### Create Bucket

Create a new storage bucket.

- **URL**: `/Storage/create_bucket`
- **Method**: POST
- **Auth Required**: Yes

#### Query Parameters

- `token`: string (required)

#### Request Body

A StorageCreateBucketRequest object:

```json
{
  "cloudAccountId": "uuid",
  "bucketName": "string",
  "isPublic": true,
  "isEncrypted": true,
  "versioningEnabled": true,
  "objectLockEnabled": true
}
```

### Get Bucket Status

Retrieve the status of a specific bucket.

- **URL**: `/Storage/bucket_status`
- **Method**: POST
- **Auth Required**: Yes

#### Query Parameters

- `token`: string (required)

#### Request Body

A StorageGetBucketStatusRequest object:

```json
{
  "cloudAccountId": "uuid",
  "bucketName": "string"
}
```

#### Response

A BucketStatus object:

```json
{
  "versioningEnabled": true,
  "encryptionEnabled": true,
  "isPublic": true,
  "objectLockEnabled": true
}
```

## Sample Code

Here's an example of how to create a new storage bucket using Python:

```python
import requests

API_BASE_URL = "https://api.amoveagent.com"
TOKEN = "your_token_here"

def create_bucket(cloud_account_id, bucket_name, is_public=False, is_encrypted=True, versioning_enabled=True, object_lock_enabled=False):
    url = f"{API_BASE_URL}/Storage/create_bucket"
    params = {"token": TOKEN}
    data = {
        "cloudAccountId": cloud_account_id,
        "bucketName": bucket_name,
        "isPublic": is_public,
        "isEncrypted": is_encrypted,
        "versioningEnabled": versioning_enabled,
        "objectLockEnabled": object_lock_enabled
    }
    
    response = requests.post(url, json=data, params=params)
    
    if response.status_code == 200:
        return True
    else:
        print(f"Error: {response.status_code}")
        print(response.text)
        return False

# Usage
success = create_bucket("your_cloud_account_id", "my-new-bucket", is_public=False, is_encrypted=True)
if success:
    print("Bucket created successfully")
else:
    print("Failed to create bucket")
```

For other programming languages, you can use their respective HTTP client libraries to make similar requests to the API endpoints.

