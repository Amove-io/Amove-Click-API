# Transfer History Endpoints

This document provides detailed information about the transfer-history endpoints in the AMove Click API. Transfer history records every completed or failed transfer the agent has observed, whether it originated from the desktop app, a drive, a sync job, or a Fastr session.

> This API is bound to `http://localhost:29123` on a machine running the Amove desktop agent. It is not a hosted service.


## Endpoints

1. [Get All](#get-all)
2. [Delete](#delete)


## Get All

Returns a paginated list of transfer-history records, joined with user and cloud-account details. Account administrators see transfers for the entire account; other users see only their own.

- **URL**: `/transferhistory/get_all`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| token | string | — | JWT token. |
| page | integer | 1 | Starting page. |
| pagesize | integer | 50 | Page size. |
| sortfield | string | "StartDate" | Field to sort by. |
| descending | boolean | true | Sort direction. |
| origin | integer (enum) | 0 | Filter by origin. Flag enum: `Direct=1`, `Drive=2`, `Sync=4`, `FastrLocal=8`, `FastrServerToServer=16`. `0` means no filter. |
| status | integer (enum) | 0 | Filter by status. Flag enum: `Created=1`, `Running=2`, `Completed=4`, `Canceled=8`, `Error=16`, `Canceling=32`, `Pausing=64`, `Paused=128`. `0` means no filter. |
| direction | integer (enum) | 0 | Filter by direction. `Upload=1`, `Download=2`. `0` means no filter. |

### Response

```json
{
  "data": [
    {
      "id": "00000000-0000-0000-0000-000000000000",
      "userId": "00000000-0000-0000-0000-000000000000",
      "accountId": "00000000-0000-0000-0000-000000000000",
      "userEmail": "user@example.com",
      "userName": "Jane Doe",
      "origin": 1,
      "direction": 1,
      "transferStatus": 4,
      "startDate": "2026-01-01T00:00:00Z",
      "endDate": "2026-01-01T00:05:00Z",
      "sourceCloudAccountId": "00000000-0000-0000-0000-000000000000",
      "sourceCloudProvider": 1,
      "sourceBucket": "src-bucket",
      "sourcePath": "path/to/file",
      "destinationCloudAccountId": "00000000-0000-0000-0000-000000000000",
      "destinationCloudProvider": 2,
      "destinationBucket": "dst-bucket",
      "destinationPath": "path/to/file",
      "totalObjects": 10,
      "transferredObjects": 10,
      "failedObjects": 0,
      "skippedObjects": 0,
      "totalSize": 1048576,
      "transferredSize": 1048576,
      "averageSpeed": 250000.0,
      "errorMessage": null,
      "name": "Nightly backup",
      "sourceCloudAccountName": "My AWS",
      "destinationCloudAccountName": "My Wasabi",
      "totalSizeString": "1.0 MB",
      "transferredSizeString": "1.0 MB"
    }
  ],
  "total": 1,
  "options": { "page": 1, "pageSize": 50 }
}
```


## Delete

Deletes a transfer-history record.

- **URL**: `/transferhistory/delete`
- **Method**: DELETE
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | Transfer-history id to delete. |
| token | string | JWT token. |

### Response

`200 OK`.


## Sample Code

### List completed transfers

<details>
<summary>Python</summary>

```python
import requests

response = requests.get(
    "http://localhost:29123/transferhistory/get_all",
    params={
        "token": "EXAMPLE_TOKEN",
        "page": 1,
        "pagesize": 25,
        "status": 4,  # Completed
    },
)
print(response.json())
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
const params = new URLSearchParams({
  token: "EXAMPLE_TOKEN",
  page: "1",
  pagesize: "25",
  status: "4",
});
const res = await fetch(
  `http://localhost:29123/transferhistory/get_all?${params}`
);
console.log(await res.json());
```

</details>

<details>
<summary>C#</summary>

```csharp
using var client = new HttpClient();
var res = await client.GetAsync(
    "http://localhost:29123/transferhistory/get_all?token=EXAMPLE_TOKEN&page=1&pagesize=25&status=4");
Console.WriteLine(await res.Content.ReadAsStringAsync());
```

</details>


For error handling, see [Error Model](errors.md).
