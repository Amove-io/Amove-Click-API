# Account Log Setting Endpoints

This document provides detailed information about the account-log-setting endpoints in the AMove Click API. These endpoints configure an external log destination (for example, Splunk) for the account.

> This API is bound to `http://localhost:29123` on a machine running the Amove desktop agent. It is not a hosted service.


## Endpoints

1. [Setup](#setup)
2. [Get](#get)
3. [Update](#update)
4. [Delete](#delete)


## Setup

Creates the account log setting for the signed-in account.

- **URL**: `/accountlogsetting/setup`
- **Method**: POST
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | JWT token. |

### Request Body

```json
{
  "token": "SPLUNK_HEC_TOKEN",
  "serverAddress": "splunk.example.com",
  "servicePort": "8088",
  "active": true,
  "provider": 1
}
```

| Field | Type | Description |
|-------|------|-------------|
| token | string | Provider-specific ingest token (for example Splunk HEC token). Maximum 100 characters. |
| serverAddress | string | Log provider hostname. Maximum 100 characters. |
| servicePort | string | Log provider port. Maximum 10 characters. |
| active | boolean | `true` to enable log forwarding. |
| provider | integer (enum) | `1` (Splunk). |

### Response

Returns the created `AccountLogSetting` object.


## Get

Returns the account log setting for the signed-in account.

- **URL**: `/accountlogsetting/get`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | JWT token. |

### Response

Returns the `AccountLogSetting` object. Shape matches the [Setup](#setup) request body, plus an `id` field.


## Update

Updates the account log setting.

- **URL**: `/accountlogsetting/update`
- **Method**: PUT
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | JWT token. |

### Request Body

A full `AccountLogSetting` object, including `id`.

### Response

Returns the updated `AccountLogSetting` object.


## Delete

Removes the account log setting.

- **URL**: `/accountlogsetting/delete`
- **Method**: DELETE
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | JWT token. |

### Response

`200 OK`.


## Sample Code

### Configure Splunk forwarding

<details>
<summary>Python</summary>

```python
import requests

response = requests.post(
    "http://localhost:29123/accountlogsetting/setup",
    params={"token": "EXAMPLE_TOKEN"},
    json={
        "token": "SPLUNK_HEC_TOKEN",
        "serverAddress": "splunk.example.com",
        "servicePort": "8088",
        "active": True,
        "provider": 1,
    },
)
print(response.json())
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
const res = await fetch(
  "http://localhost:29123/accountlogsetting/setup?token=EXAMPLE_TOKEN",
  {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      token: "SPLUNK_HEC_TOKEN",
      serverAddress: "splunk.example.com",
      servicePort: "8088",
      active: true,
      provider: 1,
    }),
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
    "http://localhost:29123/accountlogsetting/setup?token=EXAMPLE_TOKEN",
    new
    {
        token = "SPLUNK_HEC_TOKEN",
        serverAddress = "splunk.example.com",
        servicePort = "8088",
        active = true,
        provider = 1
    });
Console.WriteLine(await res.Content.ReadAsStringAsync());
```

</details>


For error handling, see [Error Model](errors.md).
