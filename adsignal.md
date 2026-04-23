# AdSignal Endpoints

This document provides detailed information about the AdSignal endpoints in the AMove Click API. AdSignal is a material-management feature for registering video assets and running frame/audio comparisons between them.

> This API is bound to `http://localhost:29123` on a machine running the Amove desktop agent. It is not a hosted service.


## Endpoints

1. [Add Material](#add-material)
2. [Get Materials](#get-materials)
3. [Compare Materials](#compare-materials)


## Add Material

Registers a video object in an existing Amove cloud account as a new AdSignal material.

- **URL**: `/adsignal/add_material`
- **Method**: POST
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | JWT token. |

### Request Body

```json
{
  "title": "Campaign spot A",
  "cloudAccountId": "00000000-0000-0000-0000-000000000000",
  "storageName": "bucket-name",
  "key": "videos/spot-a.mp4",
  "expectedDuration": 30.0
}
```

| Field | Type | Description |
|-------|------|-------------|
| title | string | Display title for the material. |
| cloudAccountId | string (uuid) | Cloud account that hosts the object. |
| storageName | string | Bucket (or container) name. |
| key | string | Object key within the bucket. |
| expectedDuration | number | Expected duration of the video in seconds. |

### Response

```json
{
  "material": {
    "id": 1,
    "identifier": "example-id",
    "title": "Campaign spot A",
    "detectedDuration": 30.1,
    "expectedDuration": 30.0,
    "createdAt": "2026-01-01T00:00:00Z",
    "updatedAt": "2026-01-01T00:00:00Z"
  }
}
```


## Get Materials

Returns a paginated list of materials registered for the signed-in account.

- **URL**: `/adsignal/get_materials`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| token | string | — | JWT token. |
| page | integer | 1 | Starting page. |
| title | string | "" | Substring match on the material title. |

### Response

```json
{
  "data": [
    {
      "id": 1,
      "countryCode": "US",
      "identifier": "example-id",
      "title": "Campaign spot A",
      "detectedDuration": 30.1,
      "expectedDuration": 30.0,
      "createdAt": "2026-01-01T00:00:00Z",
      "updatedAt": "2026-01-01T00:00:00Z",
      "url": "https://..."
    }
  ],
  "total": 1,
  "options": { "page": 1, "pageSize": 20 }
}
```


## Compare Materials

Runs a frame and audio comparison between two registered materials.

- **URL**: `/adsignal/compare_materials`
- **Method**: POST
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | JWT token. |

### Request Body

```json
{
  "matrialId1": 1,
  "matrialId2": 2
}
```

| Field | Type | Description |
|-------|------|-------------|
| matrialId1 | integer | Id of the first material. |
| matrialId2 | integer | Id of the second material. |

### Response

```json
{
  "frameComparisons": [],
  "audioComparisons": [],
  "materials": [],
  "maxFrameDistance": 0.0,
  "maxAudioDistance": 0.0
}
```

| Field | Type | Description |
|-------|------|-------------|
| frameComparisons | array | Frame-level comparison entries. |
| audioComparisons | array | Audio-level comparison entries. |
| materials | array | Echo of the materials that were compared. |
| maxFrameDistance | number | Largest frame-distance metric observed. |
| maxAudioDistance | number | Largest audio-distance metric observed. |


## Sample Code

### Register and compare two materials

<details>
<summary>Python</summary>

```python
import requests

base = "http://localhost:29123"
params = {"token": "EXAMPLE_TOKEN"}

a = requests.post(
    f"{base}/adsignal/add_material",
    params=params,
    json={
        "title": "Spot A",
        "cloudAccountId": "00000000-0000-0000-0000-000000000000",
        "storageName": "bucket",
        "key": "videos/a.mp4",
        "expectedDuration": 30.0,
    },
).json()

b = requests.post(
    f"{base}/adsignal/add_material",
    params=params,
    json={
        "title": "Spot B",
        "cloudAccountId": "00000000-0000-0000-0000-000000000000",
        "storageName": "bucket",
        "key": "videos/b.mp4",
        "expectedDuration": 30.0,
    },
).json()

result = requests.post(
    f"{base}/adsignal/compare_materials",
    params=params,
    json={"matrialId1": a["material"]["id"], "matrialId2": b["material"]["id"]},
).json()

print(result)
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
const base = "http://localhost:29123";
const token = "EXAMPLE_TOKEN";

const res = await fetch(
  `${base}/adsignal/compare_materials?token=${token}`,
  {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ matrialId1: 1, matrialId2: 2 }),
  }
);
console.log(await res.json());
```

</details>

<details>
<summary>C#</summary>

```csharp
using System.Net.Http.Json;

using var client = new HttpClient();
HttpResponseMessage res = await client.PostAsJsonAsync(
    "http://localhost:29123/adsignal/compare_materials?token=EXAMPLE_TOKEN",
    new { matrialId1 = 1, matrialId2 = 2 });
Console.WriteLine(await res.Content.ReadAsStringAsync());
```

</details>


For error handling, see [Error Model](errors.md).
