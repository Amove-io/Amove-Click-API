# FastrSettings Endpoints

This document provides detailed information about the Fastr-settings endpoints of the Amove desktop agent. Fastr settings are stored per cloud account and control transfer bandwidth caps, checksum verification, and collision behavior. Each endpoint has a "self" variant (acting on the current user) and a matching admin variant (`*_for_user`) that acts on behalf of another user.

> This API is bound to `http://localhost:29123` on a machine running the Amove desktop agent. It is not a hosted service.

## Endpoints

1. [Get All](#get-all)
2. [Get All for User](#get-all-for-user)
3. [Get by Cloud Account](#get-by-cloud-account)
4. [Insert](#insert)
5. [Insert for User](#insert-for-user)
6. [Update](#update)
7. [Update for User](#update-for-user)
8. [Delete](#delete)
9. [Delete for User](#delete-for-user)


## Get All

Returns every Fastr-settings record belonging to the current user.

- **URL**: `/fastrsettings/get_all`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | Authentication token. |

### Response

Returns a `DTOResDesktopObject<FastrSettings>` wrapper containing the list of settings and a total count.


## Get All for User

Returns every Fastr-settings record belonging to a specific user. Admin-only.

- **URL**: `/fastrsettings/get_all/{userId}`
- **Method**: GET
- **Auth Required**: Yes

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| userId | string (uuid) | Id of the user whose settings to retrieve. |

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | Authentication token. |

### Response

Returns a `DTOResDesktopObject<FastrSettings>` wrapper.


## Get by Cloud Account

Returns the Fastr-settings record associated with a specific cloud account for the current user.

- **URL**: `/fastrsettings/get_by_cloud_account/{cloudAccountId}`
- **Method**: GET
- **Auth Required**: Yes

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| cloudAccountId | string (uuid) | Id of the cloud account. |

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | Authentication token. |

### Response

Returns a single `FastrSettings` object, or `null` if no record exists for that cloud account.


## Insert

Creates or updates the Fastr settings for the current user and the cloud account referenced in the body. The pair `(UserId, CloudAccountId)` is unique.

- **URL**: `/fastrsettings/insert`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "CloudAccountId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "UserId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "AccountId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "MaxUploadBytesPerSecond": 0,
  "MaxDownloadBytesPerSecond": 0,
  "MaxServerToServerBytesPerSecond": 0,
  "VerifyChecksum": true,
  "FileExistsAction": 0,
  "Active": true
}
```

| Field | Type | Description |
|-------|------|-------------|
| CloudAccountId | string (uuid) | Cloud account this record applies to. |
| UserId | string (uuid) | User this record applies to. |
| AccountId | string (uuid) | Tenant account id. |
| MaxUploadBytesPerSecond | integer | Upload cap in bytes/sec. `0` = unlimited. |
| MaxDownloadBytesPerSecond | integer | Download cap in bytes/sec. `0` = unlimited. |
| MaxServerToServerBytesPerSecond | integer | Server-to-server cap in bytes/sec. `0` = unlimited. |
| VerifyChecksum | boolean | When `true`, verify SHA-256 after transfer. |
| FileExistsAction | integer (enum) | `0` = Overwrite, `1` = SkipIfSame, `2` = KeepBoth. |
| Active | boolean | Whether this settings record is active. |

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | Authentication token. |

### Response

Returns the saved `FastrSettings` object.


## Insert for User

Creates or updates Fastr settings on behalf of another user. Admin-only. Same body as [Insert](#insert).

- **URL**: `/fastrsettings/insert_for_user`
- **Method**: POST
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | Authentication token. |

### Response

Returns the saved `FastrSettings` object.


## Update

Updates an existing Fastr-settings record for the current user.

- **URL**: `/fastrsettings/update`
- **Method**: PUT
- **Auth Required**: Yes

### Request Body

Full `FastrSettings` object, including its `Id`.

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | Authentication token. |

### Response

Returns the updated `FastrSettings` object.


## Update for User

Updates an existing Fastr-settings record for another user. Admin-only.

- **URL**: `/fastrsettings/update_for_user`
- **Method**: PUT
- **Auth Required**: Yes

### Request Body

Full `FastrSettings` object, including its `Id`.

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | Authentication token. |

### Response

Returns the updated `FastrSettings` object.


## Delete

Deletes a Fastr-settings record belonging to the current user.

- **URL**: `/fastrsettings/delete`
- **Method**: DELETE
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | Id of the settings record to delete. |
| token | string | Authentication token. |

### Response

Returns HTTP 200 on success.


## Delete for User

Deletes a Fastr-settings record belonging to another user. Admin-only.

- **URL**: `/fastrsettings/delete_for_user`
- **Method**: DELETE
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | Id of the settings record to delete. |
| token | string | Authentication token. |

### Response

Returns HTTP 200 on success.


## Sample Code

### Read settings for a cloud account

<details>
<summary>Python</summary>

```python
import requests

cloud_account_id = "00000000-0000-0000-0000-000000000000"
response = requests.get(
    f"http://localhost:29123/fastrsettings/get_by_cloud_account/{cloud_account_id}",
    params={"token": "EXAMPLE_TOKEN"},
)
print(response.json())
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
const cloudAccountId = "00000000-0000-0000-0000-000000000000";
const res = await fetch(
  `http://localhost:29123/fastrsettings/get_by_cloud_account/${cloudAccountId}?token=EXAMPLE_TOKEN`
);
console.log(await res.json());
```

</details>

### Create or update settings

<details>
<summary>Python</summary>

```python
import requests

payload = {
    "CloudAccountId": "00000000-0000-0000-0000-000000000000",
    "UserId": "00000000-0000-0000-0000-000000000000",
    "AccountId": "00000000-0000-0000-0000-000000000000",
    "MaxUploadBytesPerSecond": 0,
    "MaxDownloadBytesPerSecond": 0,
    "MaxServerToServerBytesPerSecond": 0,
    "VerifyChecksum": True,
    "FileExistsAction": 0,
    "Active": True,
}
response = requests.post(
    "http://localhost:29123/fastrsettings/insert",
    params={"token": "EXAMPLE_TOKEN"},
    json=payload,
)
print(response.json())
```

</details>

<details>
<summary>C#</summary>

```csharp
using var client = new HttpClient();
var payload = new
{
    CloudAccountId = "00000000-0000-0000-0000-000000000000",
    UserId = "00000000-0000-0000-0000-000000000000",
    AccountId = "00000000-0000-0000-0000-000000000000",
    MaxUploadBytesPerSecond = 0L,
    MaxDownloadBytesPerSecond = 0L,
    MaxServerToServerBytesPerSecond = 0L,
    VerifyChecksum = true,
    FileExistsAction = 0,
    Active = true
};

var res = await client.PostAsJsonAsync(
    "http://localhost:29123/fastrsettings/insert?token=EXAMPLE_TOKEN", payload);
Console.WriteLine(await res.Content.ReadAsStringAsync());
```

</details>

For error handling, see [Error Model](errors.md).
