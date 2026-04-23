# Error Model

## Overview

The Click API uses the same error conventions as the Amove Web API, with one additional behavior specific to the local agent: authentication failures are surfaced as `401 Unauthorized` rather than `499`.

| Status | Meaning |
|---|---|
| 200 OK | Request succeeded. |
| 401 Unauthorized | Authentication failed: missing or invalid `token`, or the underlying session was rejected. |
| 499 | Application-level validation error. Body contains a `ValidationProblemDetails` object with a specific error code. |
| 500 Internal Server Error | Unexpected server error. No body returned. |

## HTTP 499 Response Body

```json
{
  "type": "DUPLICATE",
  "status": 499,
  "errors": {
    "DUPLICATE": [
      "A resource with the same name already exists."
    ]
  }
}
```

An `error-code` response header duplicates the value in `type`.

## Error Codes

The full list of application error codes — same as the Web API — is documented in the main API error reference. Common codes you may encounter on the Click API include:

| Code | Meaning |
|---|---|
| AUTH | Authentication failure (returned as HTTP 401 on the Click API) |
| TOKEN | Token is invalid, expired, or already used |
| SESSION | Session is invalid or expired |
| NOT_FOUND | Resource not found |
| DUPLICATE | A resource with the same unique field already exists |
| ACCESS | Access denied to the requested resource |
| DB_INSERT / DB_UPDATE / DB_DELETE | Database operation failed |
| TRANSFER | Transfer operation failed |
| SYNC | Sync operation failed |
| DOWNLOAD_OBJECT | Object download failed |
| CHECKSUM_MISMATCH | File checksum verification failed |
| FILESYSTEM_ACCESS | Local filesystem access denied |
| INTERNAL_ERROR | Internal server error surfaced as a validation error |

## Handling errors in client code

<details>
<summary>Python</summary>

```python
import requests

response = requests.get(
    "http://localhost:29123/user/get_all_users",
    params={"token": "YOUR_TOKEN"},
)

if response.status_code == 200:
    print(response.json())
elif response.status_code == 401:
    print("Authentication required; refresh your token.")
elif response.status_code == 499:
    body = response.json()
    code = body["type"]
    print(f"Application error {code}: {body['errors'][code][0]}")
elif response.status_code == 500:
    print("Server error; try again later.")
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
const res = await fetch(
  "http://localhost:29123/user/get_all_users?token=YOUR_TOKEN"
);

if (res.status === 200) {
  console.log(await res.json());
} else if (res.status === 401) {
  console.log("Authentication required.");
} else if (res.status === 499) {
  const body = await res.json();
  const code = body.type;
  console.log(`Application error ${code}: ${body.errors[code][0]}`);
} else if (res.status === 500) {
  console.log("Server error; try again later.");
}
```

</details>
