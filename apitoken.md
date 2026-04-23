# API Token Endpoints

This document provides detailed information about the API-token management endpoints in the AMove Click API. API tokens are long-lived credentials a user can generate for automated callers.

> This API is bound to `http://localhost:29123` on a machine running the Amove desktop agent. It is not a hosted service.


## Endpoints

1. [Get All API Tokens](#get-all-api-tokens)
2. [Insert API Token](#insert-api-token)
3. [Delete API Token](#delete-api-token)


## Get All API Tokens

Returns all API tokens owned by the signed-in user.

- **URL**: `/apitoken/get_all`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | JWT token. |

### Response

```json
[
  {
    "id": "00000000-0000-0000-0000-000000000000",
    "title": "CI pipeline",
    "token": "EXAMPLE_API_TOKEN",
    "expirationDate": "2027-01-01T00:00:00Z",
    "createDate": "2026-01-01T00:00:00Z"
  }
]
```

| Field | Type | Description |
|-------|------|-------------|
| id | string (uuid) | Token identifier. |
| title | string | Title the caller supplied on creation. |
| token | string | The API token value. |
| expirationDate | string (date-time) | When the token expires. |
| createDate | string (date-time) | When the token was created. |


## Insert API Token

Creates a new API token for the signed-in user.

- **URL**: `/apitoken/insert`
- **Method**: POST
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | JWT token. |

### Request Body

```json
{
  "title": "CI pipeline",
  "expirationDate": "2027-01-01T00:00:00Z"
}
```

| Field | Type | Description |
|-------|------|-------------|
| title | string | Friendly title. |
| expirationDate | string (date-time) | Optional expiration date. |

### Response

Returns the created `ApiToken` object, including the generated `token` value.


## Delete API Token

Deletes an API token.

- **URL**: `/apitoken/delete`
- **Method**: DELETE
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | API token id to delete. |
| token | string | JWT token. |

### Response

`200 OK`.


## Sample Code

### Create an API token

<details>
<summary>Python</summary>

```python
import requests

response = requests.post(
    "http://localhost:29123/apitoken/insert",
    params={"token": "EXAMPLE_TOKEN"},
    json={"title": "CI pipeline", "expirationDate": "2027-01-01T00:00:00Z"},
)
print(response.json())
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
const res = await fetch(
  "http://localhost:29123/apitoken/insert?token=EXAMPLE_TOKEN",
  {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      title: "CI pipeline",
      expirationDate: "2027-01-01T00:00:00Z",
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
    "http://localhost:29123/apitoken/insert?token=EXAMPLE_TOKEN",
    new { title = "CI pipeline", expirationDate = "2027-01-01T00:00:00Z" });
Console.WriteLine(await res.Content.ReadAsStringAsync());
```

</details>


For error handling, see [Error Model](errors.md).
