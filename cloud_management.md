# Cloud Management

The Cloud Management endpoints allow you to manage cloud accounts, buckets, objects, and perform various cloud-related operations across different cloud providers.

## Endpoints

### Get All Cloud Accounts

Retrieve all cloud accounts associated with the authenticated user.

- **URL**: `/Cloud/accounts`
- **Method**: GET
- **Auth Required**: Yes

#### Query Parameters

- `token`: string (required)

#### Response

An array of CloudAccount objects:

```json
[
  {
    "id": "uuid",
    "accountId": "uuid",
    "userId": "uuid",
    "cloudType": 0,
    "name": "string",
    "accessKey": "string",
    "secretKey": "string",
    "credentialsData": "string",
    "serviceUrl": "string",
    "active": true,
    "shared": false,
    "internalStorage": false,
    "storageTier": 0,
    "deleted": false
  }
]
```

### Get Specific Cloud Account

Retrieve a specific cloud account by ID.

- **URL**: `/Cloud/account`
- **Method**: GET
- **Auth Required**: Yes

#### Query Parameters

- `token`: string (required)
- `id`: uuid (required)

#### Response

A CloudAccount object.

### Add Cloud Account

Add a new cloud account.

- **URL**: `/Cloud/insert_account`
- **Method**: POST
- **Auth Required**: Yes

#### Query Parameters

- `token`: string (required)

#### Request Body

A CloudAccount object:

```json
{
  "accountId": "uuid",
  "userId": "uuid",
  "cloudType": 0,
  "name": "string",
  "accessKey": "string",
  "secretKey": "string",
  "credentialsData": "string",
  "serviceUrl": "string",
  "active": true,
  "shared": false,
  "internalStorage": false,
  "storageTier": 0,
  "deleted": false
}
```

#### Response

The created CloudAccount object.

### Delete Cloud Account

Delete a cloud account.

- **URL**: `/Cloud/delete`
- **Method**: DELETE
- **Auth Required**: Yes

#### Query Parameters

- `id`: uuid (required)
- `token`: string (required)

#### Response

A boolean indicating success or failure.

### List Buckets

List buckets for a given cloud account.

- **URL**: `/Cloud/list_buckets`
- **Method**: POST
- **Auth Required**: Yes

#### Query Parameters

- `includeRegion`: boolean (default: false)

#### Request Body

A CloudAccount object.

#### Response

An array of ICloudStorage objects:

```json
[
  {
    "id": "string",
    "name": "string",
    "region": "string",
    "size": 0,
    "creationDate": "2023-05-05T12:00:00Z"
  }
]
```

### List Objects

List objects in a bucket.

- **URL**: `/Cloud/list_objects`
- **Method**: POST
- **Auth Required**: Yes

#### Request Body

An ObjectsRequestModel:

```json
{
  "account": {
    // CloudAccount object
  },
  "request": {
    "id": "string",
    "bucketName": "string",
    "path": "/",
    "continuationToken": "",
    "count": 1000
  }
}
```

#### Response

An ICloudStorageObjectCollection object.

### Generate Download URL

Generate a download URL for an object.

- **URL**: `/Cloud/generate_download_url`
- **Method**: POST
- **Auth Required**: Yes

#### Query Parameters

- `token`: string (required)

#### Request Body

A GenerateDownloadURLRequest object:

```json
{
  "cloudAccountId": "uuid",
  "id": "string",
  "storageName": "string",
  "key": "string",
  "expireHours": 24,
  "forceToDownload": false
}
```

#### Response

A string containing the download URL.

### Send Download URL

Send a download URL via email.

- **URL**: `/Cloud/send_download_url`
- **Method**: POST
- **Auth Required**: Yes

#### Query Parameters

- `token`: string (required)

#### Request Body

A SendDownloadURLRequest object:

```json
{
  "email": "string",
  "cloudAccountId": "uuid",
  "id": "string",
  "storageName": "string",
  "key": "string",
  "expireHours": 24,
  "note": "string"
}
```

### Delete Object

Delete an object from a bucket.

- **URL**: `/Cloud/delete_object`
- **Method**: POST
- **Auth Required**: Yes

#### Query Parameters

- `token`: string (required)

#### Request Body

A DeleteObjectRequest object:

```json
{
  "cloudAccountId": "uuid",
  "storageName": "string",
  "key": "string",
  "id": "string"
}
```

### Create Folder

Create a new folder in a bucket.

- **URL**: `/Cloud/create_folder`
- **Method**: POST
- **Auth Required**: Yes

#### Query Parameters

- `token`: string (required)

#### Request Body

A CreateFolderRequest object:

```json
{
  "cloudAccountId": "uuid",
  "id": "string",
  "storageName": "string",
  "path": "string",
  "folderName": "string"
}
```

### Upload Object

Upload an object to a bucket.

- **URL**: `/Cloud/upload_object`
- **Method**: POST
- **Auth Required**: Yes

#### Request Body

An UploadFileRequest object:

```json
{
  "cloudAccount": {
    // CloudAccount object
  },
  "storageName": "string",
  "uploadPath": "string",
  "localFilePath": "string",
  "id": "string",
  "name": "string"
}
```

### Download Object

Download an object from a bucket.

- **URL**: `/Cloud/download_object`
- **Method**: POST
- **Auth Required**: Yes

#### Request Body

A DownloadFileRequest object:

```json
{
  "cloudAccount": {
    // CloudAccount object
  },
  "storageName": "string",
  "downloadPath": "string",
  "objectKey": "string",
  "id": "string",
  "name": "string"
}
```

### Add Storage

Add a new storage to a cloud account.

- **URL**: `/Cloud/add_storage`
- **Method**: POST
- **Auth Required**: Yes

#### Query Parameters

- `token`: string (required)

#### Request Body

An AddStorageRequest object:

```json
{
  "cloudAccountName": "string",
  "bucketName": "string",
  "shared": true
}
```

#### Response

The updated CloudAccount object.

### Delete Storage

Delete a storage from a cloud account.

- **URL**: `/Cloud/delete_storage`
- **Method**: DELETE
- **Auth Required**: Yes

#### Query Parameters

- `id`: uuid (required)
- `token`: string (required)

#### Response

A boolean indicating success or failure.

### Create Bucket

Create a new bucket in a cloud account.

- **URL**: `/Cloud/create_bucket`
- **Method**: POST
- **Auth Required**: Yes

#### Query Parameters

- `bucketName`: string (required)
- `token`: string (required)

### Delete Bucket

Delete a bucket from a cloud account.

- **URL**: `/Cloud/delete_bucket`
- **Method**: DELETE
- **Auth Required**: Yes

#### Query Parameters

- `bucketName`: string (required)
- `token`: string (required)

### Get Bucket Info

Get information about a specific bucket.

- **URL**: `/Cloud/get_bucket_info`
- **Method**: GET
- **Auth Required**: Yes

#### Query Parameters

- `bucketName`: string (required)
- `token`: string (required)

#### Response

A BucketInfoResponse object:

```json
{
  "bucketName": "string",
  "usedSpaceSize": 0,
  "objectCount": 0
}
```

### Get Cloud Info

Get information about all buckets in a cloud account.

- **URL**: `/Cloud/get_cloud_info`
- **Method**: GET
- **Auth Required**: Yes

#### Query Parameters

- `token`: string (required)

#### Response

A CloudInfoResponse object:

```json
{
  "bucketsInfo": [
    {
      "bucketName": "string",
      "usedSpaceSize": 0,
      "objectCount": 0
    }
  ],
  "totalUsedSpaceSize": 0,
  "totalObjectCount": 0
}
```

## Sample Code

Here's an example of how to list buckets using Python:

```python
import requests

API_BASE_URL = "https://api.amoveagent.com"
TOKEN = "your_token_here"

def list_buckets(cloud_account, include_region=False):
    url = f"{API_BASE_URL}/Cloud/list_buckets"
    params = {"includeRegion": include_region}
    headers = {"Authorization": f"Bearer {TOKEN}"}
    
    response = requests.post(url, json=cloud_account, params=params, headers=headers)
    
    if response.status_code == 200:
        return response.json()
    else:
        print(f"Error: {response.status_code}")
        print(response.text)
        return None

# Usage
cloud_account = {
    "id": "your_cloud_account_id",
    "cloudType": 1,
    "name": "My Cloud Account"
}

buckets = list_buckets(cloud_account, include_region=True)
if buckets:
    for bucket in buckets:
        print(f"Bucket: {bucket['name']}, Region: {bucket['region']}")
```

For other programming languages, you can use their respective HTTP client libraries to make similar requests to the API endpoints.

