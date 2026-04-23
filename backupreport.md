# BackupReport Endpoints

This document provides detailed information about the backup reporting endpoints of the Amove desktop agent. Each endpoint is scoped to a single cloud backup (identified by its `backupId`) and aggregates the local file-state database, in-flight transfers, and — where noted — the server-side transfer history.

> This API is bound to `http://localhost:29123` on a machine running the Amove desktop agent. It is not a hosted service.

## Endpoints

1. [Get Summary](#get-summary)
2. [Get Sync Health](#get-sync-health)
3. [Get Transfer Activity](#get-transfer-activity)
4. [Get File States](#get-file-states)
5. [Get File Distribution](#get-file-distribution)
6. [Get Historical Trends](#get-historical-trends)
7. [Get Dashboard](#get-dashboard)

Backups are addressed by their configured id. Use the `Management` endpoints to list available backups and obtain their ids.

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

Returns backup identity, configuration, aggregate file statistics, and the files-by-state chart data.

- **URL**: `/backupreport/summary/{backupId}`
- **Method**: GET
- **Auth Required**: No

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| backupId | string | Id of the backup configured on the agent. |

### Response

Returns a `BackupSummaryReport`:

```json
{
  "BackupId": "string",
  "BackupName": "string",
  "Status": "string",
  "UploaderStatus": "string",
  "CloudProvider": "string",
  "StorageBucket": "string",
  "LocalPath": "string",
  "Prefix": "string",
  "Active": true,
  "CloudObjectsEncrypted": false,
  "LocalCacheEncrypted": false,
  "KeepFileAfterUpload": true,
  "SyncIntervalSeconds": 3600,
  "TotalFiles": 0,
  "TotalDirectories": 0,
  "TotalSizeBytes": 0,
  "TotalSizeFormatted": "string",
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

Returns sync-health metrics for the backup: pending uploads, error counts, completion percentage, and the sync-state breakdown chart.

- **URL**: `/backupreport/sync_health/{backupId}`
- **Method**: GET
- **Auth Required**: No

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| backupId | string | Id of the backup configured on the agent. |

### Response

Returns a `BackupSyncHealthReport`:

```json
{
  "BackupId": "string",
  "BackupStatus": "string",
  "FilesPendingUpload": 0,
  "FilesUploading": 0,
  "FilesNormal": 0,
  "FilesWithErrors": 0,
  "FilesDeleting": 0,
  "TotalTrackedFiles": 0,
  "SyncCompletionPercent": 0,
  "ActiveUploadCount": 0,
  "SyncStateBreakdown": [
    { "Label": "string", "Value": 0, "Color": "string" }
  ]
}
```


## Get Transfer Activity

Returns recent transfer history for this backup from the central server, filtered by backup name and paginated. This endpoint calls the remote Amove Web API and therefore requires a token.

- **URL**: `/backupreport/transfer_activity/{backupId}`
- **Method**: GET
- **Auth Required**: Yes

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| backupId | string | Id of the backup configured on the agent. |

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| token | string | — | Authentication token. |
| status | integer | 0 | Transfer status filter (0 = all). |
| page | integer | 1 | Starting page. |
| pageSize | integer | 50 | Page size. |

### Response

Returns a `BackupTransferActivityReport`:

```json
{
  "BackupId": "string",
  "Data": [ { "...": "TransferHistoryDetail" } ],
  "Total": 0,
  "Page": 1,
  "PageSize": 50
}
```


## Get File States

Returns paginated file-state entries from the local LiteDB database for this backup, along with per-state counts.

- **URL**: `/backupreport/file_states/{backupId}`
- **Method**: GET
- **Auth Required**: No

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| backupId | string | Id of the backup configured on the agent. |

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| state | integer (enum) | 0 | FileState filter; `0` returns every state. |
| page | integer | 1 | Starting page. |
| pageSize | integer | 50 | Page size. |

### Response

Returns a `FileStateReport`:

```json
{
  "BackupId": "string",
  "Files": [
    {
      "Path": "string",
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

Returns analytical views over the backup content: file counts and sizes by extension, a size-bucket histogram, the largest files, recently modified files, and per-directory breakdown.

- **URL**: `/backupreport/file_distribution/{backupId}`
- **Method**: GET
- **Auth Required**: No

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| backupId | string | Id of the backup configured on the agent. |

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| topFiles | integer | 20 | Maximum number of entries in `LargestFiles`. |
| topDirectories | integer | 20 | Maximum number of entries in `DirectoryBreakdown`. |

### Response

Returns a `BackupFileDistributionReport` with `FileCountByExtension`, `SizeByExtension`, `SizeDistribution`, `LargestFiles`, `RecentlyModified`, and `DirectoryBreakdown` collections.


## Get Historical Trends

Returns per-day aggregated trends over the last N days: transfers per day, data volume, average speed, and the overall success-vs-failure split. The data is aggregated server-side and this endpoint requires a token.

- **URL**: `/backupreport/historical_trends/{backupId}`
- **Method**: GET
- **Auth Required**: Yes

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| backupId | string | Id of the backup configured on the agent. |

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| token | string | — | Authentication token. |
| days | integer | 30 | Size of the trend window in days. |

### Response

Returns a `BackupHistoricalTrendsReport`:

```json
{
  "BackupId": "string",
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

- **URL**: `/backupreport/dashboard/{backupId}`
- **Method**: GET
- **Auth Required**: No

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| backupId | string | Id of the backup configured on the agent. |

### Response

Returns a `BackupDashboardReport`:

```json
{
  "Summary": { "...": "BackupSummaryReport" },
  "SyncHealth": { "...": "BackupSyncHealthReport" },
  "FileDistribution": { "...": "BackupFileDistributionReport" }
}
```


## Sample Code

### Get the backup dashboard

<details>
<summary>Python</summary>

```python
import requests

backup_id = "00000000-0000-0000-0000-000000000000"
response = requests.get(
    f"http://localhost:29123/backupreport/dashboard/{backup_id}"
)
print(response.json())
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
const backupId = "00000000-0000-0000-0000-000000000000";
const res = await fetch(
  `http://localhost:29123/backupreport/dashboard/${backupId}`
);
console.log(await res.json());
```

</details>

### Get recent transfer activity

<details>
<summary>Python</summary>

```python
import requests

backup_id = "00000000-0000-0000-0000-000000000000"
response = requests.get(
    f"http://localhost:29123/backupreport/transfer_activity/{backup_id}",
    params={"token": "EXAMPLE_TOKEN", "page": 1, "pageSize": 25},
)
print(response.json())
```

</details>

<details>
<summary>C#</summary>

```csharp
using var client = new HttpClient();
string backupId = "00000000-0000-0000-0000-000000000000";
var res = await client.GetAsync(
    $"http://localhost:29123/backupreport/historical_trends/{backupId}" +
    "?token=EXAMPLE_TOKEN&days=30");
Console.WriteLine(await res.Content.ReadAsStringAsync());
```

</details>

For error handling, see [Error Model](errors.md).
