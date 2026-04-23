# Fastr Endpoints

This document provides detailed information about the Fastr-server management endpoints of the Amove desktop agent. Fastr is Amove's high-performance file transfer product; these endpoints register and manage the Fastr servers associated with the current user's account and support server-to-server migrations.

> This API is bound to `http://localhost:29123` on a machine running the Amove desktop agent. It is not a hosted service.

## Endpoints

1. [Get All Fastr Servers](#get-all-fastr-servers)
2. [Insert Fastr Server](#insert-fastr-server)
3. [Manage Share](#manage-share)
4. [Delete Fastr Server](#delete-fastr-server)
5. [List Server Objects](#list-server-objects)
6. [Migrate](#migrate)


## Get All Fastr Servers

Returns every Fastr server associated with the current user's account.

- **URL**: `/fastr/get_all`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | Authentication token. |

### Response

Returns an array of `FastrServer`:

```json
[
  {
    "Id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "AccountId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "UserId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "Name": "string",
    "ServiceUrl": "string",
    "Port": "string",
    "Username": "string",
    "Password": "string",
    "Active": true,
    "Shared": false,
    "Dedicated": true,
    "Deleted": false
  }
]
```


## Insert Fastr Server

Registers a new Fastr server.

- **URL**: `/fastr/insert`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "Name": "string",
  "ServiceUrl": "string",
  "Port": "string",
  "Username": "string",
  "Password": "string",
  "Active": true,
  "Shared": false,
  "Dedicated": true
}
```

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | Authentication token. |

### Response

Returns the created `FastrServer` with its assigned `Id`.


## Manage Share

Updates the sharing configuration of a Fastr server (for example, enabling or disabling sharing across the account).

- **URL**: `/fastr/manage_share`
- **Method**: PUT
- **Auth Required**: Yes

### Request Body

Full `FastrServer` object, including its `Id`.

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | Authentication token. |

### Response

Returns HTTP 200 on success.


## Delete Fastr Server

Deletes a Fastr server. Only servers belonging to the same account as the current user can be deleted.

- **URL**: `/fastr/delete`
- **Method**: DELETE
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | Id of the Fastr server to delete. |
| token | string | Authentication token. |

### Response

Returns `true` when the server was deleted.


## List Server Objects

Lists files and directories at a given path on a Fastr server.

- **URL**: `/fastr/list_server_objects`
- **Method**: POST
- **Auth Required**: No

### Request Body

```json
{
  "Server": {
    "Id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "Name": "string",
    "ServiceUrl": "string",
    "Port": "string",
    "Username": "string",
    "Password": "string"
  },
  "Path": "string"
}
```

### Response

Returns an array of `FileSystemObject`:

```json
[
  {
    "FileName": "string",
    "FullPath": "string",
    "ObjectType": 1
  }
]
```

`ObjectType` values: `1` = File, `2` = Folder.


## Migrate

Starts a Fastr upload or download migration between the local file system and a Fastr server.

- **URL**: `/fastr/migrate`
- **Method**: POST
- **Auth Required**: No

### Request Body

```json
{
  "Server": {
    "Id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "Name": "string",
    "ServiceUrl": "string",
    "Port": "string",
    "Username": "string",
    "Password": "string"
  },
  "ObjectType": 1,
  "SourcePath": "string",
  "DestinationPath": "string",
  "MigrationType": 1
}
```

`MigrationType` values: `1` = Upload, `2` = Download. `ObjectType` values: `1` = File, `2` = Folder.

### Response

Returns HTTP 200 once the migration has been accepted.


## Sample Code

### List and register servers

<details>
<summary>Python</summary>

```python
import requests

base = "http://localhost:29123"
token = "EXAMPLE_TOKEN"

# List existing servers.
servers = requests.get(f"{base}/fastr/get_all", params={"token": token}).json()
print(servers)

# Register a new Fastr server.
new_server = requests.post(
    f"{base}/fastr/insert",
    params={"token": token},
    json={
        "Name": "example-fastr",
        "ServiceUrl": "fastr.example.com",
        "Port": "29124",
        "Username": "fastr-user",
        "Password": "placeholder-password",
        "Active": True,
        "Shared": False,
        "Dedicated": True,
    },
).json()
print(new_server)
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
const base = "http://localhost:29123";
const token = "EXAMPLE_TOKEN";

const servers = await fetch(`${base}/fastr/get_all?token=${token}`).then(r => r.json());
console.log(servers);

const newServer = await fetch(`${base}/fastr/insert?token=${token}`, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    Name: "example-fastr",
    ServiceUrl: "fastr.example.com",
    Port: "29124",
    Username: "fastr-user",
    Password: "placeholder-password",
    Active: true,
    Shared: false,
    Dedicated: true,
  }),
}).then(r => r.json());
console.log(newServer);
```

</details>

### Start an upload migration

<details>
<summary>C#</summary>

```csharp
using var client = new HttpClient();
var payload = new
{
    Server = new { Id = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" },
    ObjectType = 2,        // Folder
    SourcePath = "/local/source/folder",
    DestinationPath = "/remote/destination/folder",
    MigrationType = 1      // Upload
};

var res = await client.PostAsJsonAsync(
    "http://localhost:29123/fastr/migrate", payload);
Console.WriteLine(res.StatusCode);
```

</details>

For error handling, see [Error Model](errors.md).
