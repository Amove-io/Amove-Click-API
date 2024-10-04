# Cloud Management

## Index of Endpoints

1. [Get All Cloud Accounts](#get-all-cloud-accounts)
2. [Get Specific Cloud Account](#get-specific-cloud-account)
3. [Add Cloud Account](#add-cloud-account)
4. [Delete Cloud Account](#delete-cloud-account)
5. [List Buckets](#list-buckets)
6. [List Objects](#list-objects)
7. [Generate Download URL](#generate-download-url)
8. [Send Download URL](#send-download-url)
9. [Delete Object](#delete-object)
10. [Create Folder](#create-folder)
11. [Upload Object](#upload-object)
12. [Download Object](#download-object)
13. [Add Storage](#add-storage)
14. [Delete Storage](#delete-storage)
15. [Create Bucket](#create-bucket)
16. [Delete Bucket](#delete-bucket)
17. [Get Bucket Info](#get-bucket-info)
18. [Get Cloud Info](#get-cloud-info)
19. [IDrive Add Storage](#idrive-add-storage)
20. [IDrive Delete Storage](#idrive-delete-storage)
21. [IDrive Create Bucket](#idrive-create-bucket)
22. [IDrive Delete Bucket](#idrive-delete-bucket)
23. [IDrive Regions](#idrive-regions)
24. [IDrive Get Daily Usage](#idrive-get-daily-usage)
25. [IDrive Get Usage](#idrive-get-usage)
26. [Generate Dropbox Authorization URL](#generate-dropbox-authorization-url)
27. [Dropbox Authorization Callback](#dropbox-authorization-callback)
28. [Generate Box Authorization URL](#generate-box-authorization-url)
29. [Box Authorization Callback](#box-authorization-callback)
30. [Generate OneDrive Authorization URL](#generate-onedrive-authorization-url)
31. [OneDrive Authorization Callback](#onedrive-authorization-callback)
32. [Generate Google Drive Authorization URL](#generate-google-drive-authorization-url)
33. [Google Drive Authorization Callback](#google-drive-authorization-callback)

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

### IDrive Add Storage

Add IDrive storage to a cloud account.

- **URL**: `/Cloud/idrive_add_storage`
- **Method**: POST
- **Auth Required**: Yes

#### Query Parameters

- `token`: string (required)

#### Request Body

A CreateIDriveStorageRequest object:

```json
{
  "firstname": "string",
  "userEmail": "string",
  "cloudAccountName": "string",
  "bucketTitle": "string",
  "region": "string",
  "defaultPassword": "string",
  "shared": true,
  "storageTier": 0
}
```

#### Response

The created CloudAccount object.

### IDrive Delete Storage

Delete IDrive storage from a cloud account.

- **URL**: `/Cloud/idrive_delete_storage`
- **Method**: DELETE
- **Auth Required**: Yes

#### Query Parameters

- `id`: uuid (required)
- `token`: string (required)

#### Response

A boolean indicating success or failure.

### IDrive Create Bucket

Create a new bucket in IDrive storage.

- **URL**: `/Cloud/idrive_create_bucket`
- **Method**: POST
- **Auth Required**: Yes

#### Query Parameters

- `bucketName`: string (required)
- `cloudAccountId`: uuid (required)
- `token`: string (required)

### IDrive Delete Bucket

Delete a bucket from IDrive storage.

- **URL**: `/Cloud/idrive_delete_bucket`
- **Method**: DELETE
- **Auth Required**: Yes

#### Query Parameters

- `bucketName`: string (required)
- `cloudAccountId`: uuid (required)
- `token`: string (required)

### IDrive Regions

Get available IDrive regions.

- **URL**: `/Cloud/idrive_regions`
- **Method**: GET
- **Auth Required**: Yes

#### Query Parameters

- `token`: string (required)

#### Response

An array of IDriveRegion objects:

```json
[
  {
    "name": "string",
    "code": "string"
  }
]
```

### IDrive Get Daily Usage

Get daily usage for IDrive storage.

- **URL**: `/Cloud/idrive_get_daily_usage`
- **Method**: GET
- **Auth Required**: Yes

#### Query Parameters

- `token`: string (required)
- `storageTier`: integer (enum)

#### Response

A double value representing the daily usage.

### IDrive Get Usage

Get total usage for IDrive storage.

- **URL**: `/Cloud/idrive_get_usage`
- **Method**: GET
- **Auth Required**: Yes

#### Query Parameters

- `token`: string (required)
- `storageTier`: integer (enum)

#### Response

A double value representing the total usage.

### Generate Dropbox Authorization URL

Generate an authorization URL for Dropbox.

- **URL**: `/Cloud/generate_dropbox_authorization_url`
- **Method**: POST
- **Auth Required**: Yes

#### Query Parameters

- `token`: string (required)

#### Request Body

A DropboxGenerateURLRequest object:

```json
{
  "redirectURL": "string",
  "cloudName": "string"
}
```

#### Response

A string containing the authorization URL.

### Dropbox Authorization Callback

Handle the Dropbox authorization callback.

- **URL**: `/Cloud/dropbox_authorization_callback`
- **Method**: POST
- **Auth Required**: Yes

#### Query Parameters

- `token`: string (required)

#### Request Body

A DropboxAuthorizationCallbackRequest object:

```json
{
  "code": "string",
  "state": "string",
  "redirectURL": "string"
}
```

#### Response

The created CloudAccount object.

### Generate Box Authorization URL

Generate an authorization URL for Box.

- **URL**: `/Cloud/generate_box_authorization_url`
- **Method**: POST
- **Auth Required**: Yes

#### Query Parameters

- `token`: string (required)

#### Request Body

A BoxGenerateURLRequest object:

```json
{
  "redirectURL": "string",
  "cloudName": "string"
}
```

#### Response

A string containing the authorization URL.

### Box Authorization Callback

Handle the Box authorization callback.

- **URL**: `/Cloud/box_authorization_callback`
- **Method**: POST
- **Auth Required**: Yes

#### Query Parameters

- `token`: string (required)

#### Request Body

A BoxAuthorizationCallbackRequest object:

```json
{
  "code": "string",
  "state": "string",
  "redirectURL": "string"
}
```

#### Response

The created CloudAccount object.

### Generate OneDrive Authorization URL

Generate an authorization URL for OneDrive.

- **URL**: `/Cloud/generate_onedrive_authorization_url`
- **Method**: POST
- **Auth Required**: Yes

#### Query Parameters

- `token`: string (required)

#### Request Body

A OneDriveGenerateURLRequest object:

```json
{
  "cloudName": "string",
  "redirectURL": "string"
}
```

#### Response

A string containing the authorization URL.

### OneDrive Authorization Callback

Handle the OneDrive authorization callback.

- **URL**: `/Cloud/onedrive_authorization_callback`
- **Method**: POST
- **Auth Required**: Yes

#### Query Parameters

- `token`: string (required)

#### Request Body

A OneDriveAuthorizationCallbackRequest object:

```json
{
  "code": "string",
  "state": "string",
  "error": "string",
  "errorDescription": "string",
  "redirectURL": "string"
}
```

#### Response

The created CloudAccount object.

### Generate Google Drive Authorization URL

Generate an authorization URL for Google Drive.

- **URL**: `/Cloud/generate_googledrive_authorization_url`
- **Method**: POST
- **Auth Required**: Yes

#### Query Parameters

- `token`: string (required)

#### Request Body

A GoogleDriveGenerateURLRequest object:

```json
{
  "cloudName": "string",
  "redirectURL": "string"
}
```

#### Response

A string containing the authorization URL.

### Google Drive Authorization Callback

Handle the Google Drive authorization callback.

- **URL**: `/Cloud/googledrive_authorization_callback`
- **Method**: POST
- **Auth Required**: Yes

#### Query Parameters

- `token`: string (required)

#### Request Body

A GoogleDriveAuthorizationCallbackRequest object:

```json
{
  "code": "string",
  "state": "string",
  "error": "string",
  "errorDescription": "string",
  "redirectURL": "string"
}
```

#### Response

The created CloudAccount object.

### Upload Object from Form

Upload an object to a bucket using a form.

- **URL**: `/Cloud/upload_object_from_form`
- **Method**: POST
- **Auth Required**: Yes

#### Request Body

Multipart form data:

- `CloudAccountJson`: string
- `StorageName`: string
- `UploadPath`: string
- `File`: file
- `Id`: string
- `Name`: string

### Get Object Stream

Get a stream of object data.

- **URL**: `/Cloud/get_object_stream`
- **Method**: GET
- **Auth Required**: Yes

#### Query Parameters

- `path`: string (required)
- `start`: integer (optional)
- `end`: integer (optional)

#### Response

A stream of object data.

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

This code demonstrates how to use the list_buckets endpoint. You can create similar functions for other endpoints using the appropriate HTTP methods and request/response structures.

## Error Handling

The API uses standard HTTP status codes to indicate the success or failure of requests. Common status codes include:

- 200: OK - The request was successful
- 400: Bad Request - The request was invalid or cannot be served
- 401: Unauthorized - Authentication failed or user doesn't have permissions for the requested operation
- 403: Forbidden - The request is not allowed
- 404: Not Found - The requested resource could not be found
- 500: Internal Server Error - The server encountered an unexpected condition

When an error occurs, the response body may contain additional information about the error. Always check the status code and handle errors appropriately in your applications.

## Conclusion

This documentation covers all the Cloud Management endpoints available in the AmoveAgent API. It includes details on managing cloud accounts, buckets, objects, and integrations with various cloud storage providers. Use these endpoints to build robust cloud management features in your applications.

Remember to handle authentication properly and respect rate limits if any are imposed by the API. For the most up-to-date information, always refer to the official API documentation provided by AmoveAgent.

