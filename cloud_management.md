# Cloud Management

The Cloud Management endpoints allow you to manage cloud accounts, buckets, and perform various cloud-related operations.

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

### Create Bucket

Create a new bucket in a cloud account.

- **URL**: `/Cloud/create_bucket`
- **Method**: POST
- **Auth Required**: Yes

#### Query Parameters

- `token`: string (required)

#### Request Body

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

## Sample Code

Here's an example of how to list buckets using Python:

```python
import requests

API_BASE_URL = "https://api.amoveagent.com"
TOKEN = "your_token_here"

def list_buckets(cloud_account, include_region=False):
    url = f"{API_BASE_URL}/Cloud/list_buckets"
    params = {"token": TOKEN, "includeRegion": include_region}
    
    response = requests.post(url, json=cloud_account, params=params)
    
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

