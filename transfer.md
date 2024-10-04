# Transfer

The Transfer endpoints allow you to manage and monitor data transfer operations between cloud storage accounts or buckets.

## Endpoints

### Get All Transfers

Retrieve all transfer operations.

- **URL**: `/Transfer/get_all`
- **Method**: GET
- **Auth Required**: Yes

#### Query Parameters

- `token`: string (required)
- `page`: integer (default: 1)
- `pagesize`: integer (default: 50)
- `sortfield`: string (default: "RequestDate")
- `descending`: boolean (default: true)
- `type`: integer (default: 15)

#### Response

A TransferDTOCollection containing an array of Transfer objects:

```json
{
  "data": [
    {
      "id": "uuid",
      "userId": "uuid",
      "syncCycleId": "uuid",
      "sourceCloudAccountId": "uuid",
      "sourceBucket": "string",
      "sourceRegion": "string",
      "sourcePath": "string",
      "destinationCloudAccountId": "uuid",
      "destinationBucket": "string",
      "destinationRegion": "string",
      "destinationPath": "string",
      "requestDate": "2023-05-05T12:00:00Z",
      "startDate": "2023-05-05T12:00:00Z",
      "endDate": "2023-05-05T12:00:00Z",
      "transferStatus": 0,
      "transferType": 0,
      "allowSkip": true,
      "allowDelete": false,
      "total": 0,
      "failed": 0,
      "deleted": 0,
      "skipped": 0,
      "skippedSize": 0,
      "transferred": 0,
      "totalSize": 0,
      "transferredSize": 0,
      "percent": 0,
      "averageSpeed": 0,
      "transferredCost": 0
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

### Create Transfer

Create a new transfer operation.

- **URL**: `/Transfer/transfer`
- **Method**: POST
- **Auth Required**: Yes

#### Query Parameters

- `token`: string (required)

#### Request Body

A TransferRequest object:

```json
{
  "sourceCloudAccountId": "uuid",
  "sourceBucket": "string",
  "sourcePath": "string",
  "destinationCloudAccountId": "uuid",
  "destinationBucket": "string",
  "destinationPath": "string",
  "allowSkip": true
}
```

### Cancel Transfer

Cancel an ongoing transfer operation.

- **URL**: `/Transfer/cancel_transfer`
- **Method**: POST
- **Auth Required**: Yes

#### Query Parameters

- `token`: string (required)

#### Request Body

A GuidEntityReuest object:

```json
{
  "id": "uuid"
}
```

## Sample Code

Here's an example of how to create a new transfer operation using Python:

```python
import requests

API_BASE_URL = "https://api.amoveagent.com"
TOKEN = "your_token_here"

def create_transfer(source_account_id, source_bucket, source_path, dest_account_id, dest_bucket, dest_path, allow_skip=True):
    url = f"{API_BASE_URL}/Transfer/transfer"
    params = {"token": TOKEN}
    data = {
        "sourceCloudAccountId": source_account_id,
        "sourceBucket": source_bucket,
        "sourcePath": source_path,
        "destinationCloudAccountId": dest_account_id,
        "destinationBucket": dest_bucket,
        "destinationPath": dest_path,
        "allowSkip": allow_skip
    }
    
    response = requests.post(url, json=data, params=params)
    
    if response.status_code == 200:
        return True
    else:
        print(f"Error: {response.status_code}")
        print(response.text)
        return False

# Usage
success = create_transfer(
    "source_account_id",
    "source-bucket",
    "/source/path",
    "dest_account_id",
    "dest-bucket",
    "/dest/path"
)
if success:
    print("Transfer operation created successfully")
else:
    print("Failed to create transfer operation")
```

For other programming languages, you can use their respective HTTP client libraries to make similar requests to the API endpoints.

