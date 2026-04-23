# DriveReport Endpoints

This document provides detailed information about the cloud-drive reporting endpoints of the Amove desktop agent. Each endpoint is scoped to a single cloud drive (identified by its `driveId`) and aggregates the local file-state database, in-flight transfers, and — where noted — the server-side transfer history.

> This API is bound to `http://localhost:29123` on a machine running the Amove desktop agent. It is not a hosted service.

## Endpoints

1. [Get Summary](#get-summary)
2. [Get Sync Health](#get-sync-health)
3. [Get Transfer Activity](#get-transfer-activity)
4. [Get File States](#get-file-states)
5. [Get File Distribution](#get-file-distribution)
6. [Get Historical Trends](#get-historical-trends)
7. [Get Dashboard](#get-dashboard)

Drives are addressed by their configured id. Use the `Management` endpoints to list available drives and obtain their ids.

### File State Values

`state` and the `State` field returned in responses use the `FileState` enumeration:

| Value | Name |
|-------|------|
| 0 | Normal |
| 1 | Temporary |
| 2 | Downloading |
| 4 | Uploading |
| 8 | Deleting |
| 16 | Ignore |
| 32 | RenameDeleting |
| 64 | Deleted |
| 128 | Reading |
| 256 | Error |


## Get Summary

Returns drive identity, configuration, aggregate file statistics, local-cache size, and the files-by-state chart data.

- **URL**: `/drivereport/summary/{driveId}`
- **Method**: GET
- **Auth Required**: No

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| driveId | string | Id of the drive configured on the agent. |

### Response

Returns a `DriveSummaryReport`:

```json
{
  "DriveId": "string",
  "DriveName": "string",
  "Name": "string",
  "Status": "string",
  "UploaderStatus": "string",
  "DownloaderStatus": "string",
  "CloudProvider": "string",
  "StorageBucket": "string",
  "LocalPath": "string",
  "Prefix": "string",
  "Active": true,
  "DriveType": "string",
  "SyncType": "string",
  "SharedDrive": false,
  "SharedDriveId": "string",
  "PermissionType": "string",
  "AllowDelete": true,
  "MaximumCacheSize": 0,
  "MaximumCacheSizeFormatted": "string",
  "RotationalCache": false,
  "CloudObjectsEncrypted": false,
  "LocalCacheEncrypted": false,
  "KeepFileAfterUpload": false,
  "SyncIntervalSeconds": 60,
  "TotalFiles": 0,
  "TotalDirectories": 0,
  "TotalSizeBytes": 0,
  "TotalSizeFormatted": "string",
  "CacheSizeBytes": 0,
  "CacheSizeFormatted": "string",
  "FilesByState": [
    { "Label": "string", "Value": 0, "Color": "string" }
  ],
  "TransferOptions": {
    "ChunkSize": 0,
    "RetryCount": 0,
    "ConcurrentChunks": 0,
    "ConcurrentFiles": 0
  }
}
```


## Get Sync Health

Returns sync-health metrics for the drive: pending uploads and downloads, error count, completion percentages (per direction and overall), and the sync-state breakdown chart. Counts are reconciled against in-flight transfers (stale entries older than 30 minutes with no bytes transferred are filtered out).

- **URL**: `/drivereport/sync_health/{driveId}`
- **Method**: GET
- **Auth Required**: No

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| driveId | string | Id of the drive configured on the agent. |

### Response

Returns a `DriveSyncHealthReport`:

```json
{
  "DriveId": "string",
  "DriveStatus": "string",
  "FilesPendingUpload": 0,
  "FilesUploading": 0,
  "FilesPendingDownload": 0,
  "FilesDownloading": 0,
  "FilesNormal": 0,
  "FilesTemporary": 0,
  "FilesWithErrors": 0,
  "FilesDeleting": 0,
  "TotalTrackedFiles": 0,
  "UploadCompletionPercent": 0,
  "DownloadCompletionPercent": 0,
  "OverallSyncCompletionPercent": 0,
  "ActiveUploadCount": 0,
  "ActiveDownloadCount": 0,
  "SyncStateBreakdown": [
    { "Label": "string", "Value": 0, "Color": "string" }
  ]
}
```


## Get Transfer Activity

Returns recent transfer history for this drive from the central server, filtered by drive name and paginated. This endpoint calls the remote Amove Web API and therefore requires a token. Direction filtering is applied client-side after the server returns the page.

- **URL**: `/drivereport/transfer_activity/{driveId}`
- **Method**: GET
- **Auth Required**: Yes

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| driveId | string | Id of the drive configured on the agent. |

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| token | string | — | Authentication token. |
| direction | integer | 0 | `0` = both directions, `1` = uploads only, `2` = downloads only. |
| status | integer | 0 | Transfer status filter (0 = all). |
| page | integer | 1 | Starting page. |
| pageSize | integer | 50 | Page size. |

### Response

Returns a `DriveTransferActivityReport`:

```json
{
  "DriveId": "string",
  "Direction": "All | Upload | Download",
  "Data": [ { "...": "TransferHistoryDetail" } ],
  "Total": 0,
  "Page": 1,
  "PageSize": 50
}
```


## Get File States

Returns paginated file-state entries from the local LiteDB database for this drive, along with per-state counts. File paths are returned relative to the drive's local path.

- **URL**: `/drivereport/file_states/{driveId}`
- **Method**: GET
- **Auth Required**: No

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| driveId | string | Id of the drive configured on the agent. |

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| state | integer (enum) | 0 | FileState filter; `0` returns every state. |
| page | integer | 1 | Starting page. |
| pageSize | integer | 50 | Page size. |

### Response

Returns a `DriveFileStateReport`:

```json
{
  "DriveId": "string",
  "Files": [
    {
      "Path": "string (relative)",
      "FileName": "string",
      "SizeBytes": 0,
      "SizeFormatted": "string",
      "State": "string",
      "LastModifiedDateUtc": "2026-01-01T00:00:00Z"
    }
  ],
  "Total": 0,
  "Page": 1,
  "PageSize": 50,
  "StateCounts": [
    { "State": "string", "Count": 0, "TotalSize": 0, "TotalSizeFormatted": "string" }
  ]
}
```


## Get File Distribution

Returns analytical views over the drive content: file counts and sizes by extension, a size-bucket histogram, the largest files, recently modified files, and per-directory breakdown.

- **URL**: `/drivereport/file_distribution/{driveId}`
- **Method**: GET
- **Auth Required**: No

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| driveId | string | Id of the drive configured on the agent. |

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| topFiles | integer | 20 | Maximum number of entries in `LargestFiles`. |
| topDirectories | integer | 20 | Maximum number of entries in `DirectoryBreakdown`. |

### Response

Returns a `DriveFileDistributionReport` with `FileCountByExtension`, `SizeByExtension`, `SizeDistribution`, `LargestFiles`, `RecentlyModified`, and `DirectoryBreakdown` collections.


## Get Historical Trends

Returns per-day aggregated trends over the last N days: transfers per day, data volume, average speed, and the overall success-vs-failure split. The data is aggregated server-side and this endpoint requires a token.

- **URL**: `/drivereport/historical_trends/{driveId}`
- **Method**: GET
- **Auth Required**: Yes

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| driveId | string | Id of the drive configured on the agent. |

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| token | string | — | Authentication token. |
| days | integer | 30 | Size of the trend window in days. |

### Response

Returns a `DriveHistoricalTrendsReport`:

```json
{
  "DriveId": "string",
  "TransfersPerDay": [ { "Date": "2026-04-01", "Value": 0 } ],
  "DataVolumePerDay": [ { "Date": "2026-04-01", "Value": 0 } ],
  "AverageSpeedPerDay": [ { "Date": "2026-04-01", "Value": 0 } ],
  "SuccessVsFailureRate": [
    { "Label": "Completed", "Value": 0, "Color": "#22c55e" },
    { "Label": "Failed",    "Value": 0, "Color": "#ef4444" },
    { "Label": "Canceled",  "Value": 0, "Color": "#f59e0b" }
  ],
  "CumulativeDataTransferred": 0,
  "CumulativeDataTransferredFormatted": "string"
}
```


## Get Dashboard

Returns the combined dashboard view built from `summary`, `sync_health`, and `file_distribution`. Transfer activity and historical trends are not included here and must be fetched from their own endpoints.

- **URL**: `/drivereport/dashboard/{driveId}`
- **Method**: GET
- **Auth Required**: No

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| driveId | string | Id of the drive configured on the agent. |

### Response

Returns a `DriveDashboardReport`:

```json
{
  "Summary": { "...": "DriveSummaryReport" },
  "SyncHealth": { "...": "DriveSyncHealthReport" },
  "FileDistribution": { "...": "DriveFileDistributionReport" }
}
```


## Sample Code

### Get the drive dashboard

<details>
<summary>Python</summary>

```python
import requests

drive_id = "00000000-0000-0000-0000-000000000000"
response = requests.get(
    f"http://localhost:29123/drivereport/dashboard/{drive_id}"
)
print(response.json())
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
const driveId = "00000000-0000-0000-0000-000000000000";
const res = await fetch(
  `http://localhost:29123/drivereport/dashboard/${driveId}`
);
console.log(await res.json());
```

</details>

### Get download-only transfer activity

<details>
<summary>Python</summary>

```python
import requests

drive_id = "00000000-0000-0000-0000-000000000000"
response = requests.get(
    f"http://localhost:29123/drivereport/transfer_activity/{drive_id}",
    params={
        "token": "EXAMPLE_TOKEN",
        "direction": 2,
        "page": 1,
        "pageSize": 25,
    },
)
print(response.json())
```

</details>

<details>
<summary>C#</summary>

```csharp
using var client = new HttpClient();
string driveId = "00000000-0000-0000-0000-000000000000";
var res = await client.GetAsync(
    $"http://localhost:29123/drivereport/historical_trends/{driveId}" +
    "?token=EXAMPLE_TOKEN&days=14");
Console.WriteLine(await res.Content.ReadAsStringAsync());
```

</details>

For error handling, see [Error Model](errors.md).
