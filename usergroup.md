# UserGroup Endpoints

This document provides detailed information about the user group management endpoints exposed by the Amove desktop agent's Click API.

> This API is bound to `http://localhost:29123` on a machine running the Amove desktop agent. It is not a hosted service.

## Endpoints

1. [Get All User Groups](#get-all-user-groups)
2. [Get All User Groups With Details](#get-all-user-groups-with-details)
3. [Insert User Group](#insert-user-group)
4. [Update User Group](#update-user-group)
5. [Delete User Group](#delete-user-group)
6. [Get Assigned Users](#get-assigned-users)
7. [Get Assigned User Groups](#get-assigned-user-groups)
8. [Assign Users](#assign-users)
9. [Unassign User](#unassign-user)
10. [Import Users](#import-users)


## Get All User Groups

Returns the paginated list of user groups defined in the current account.

- **URL**: `/usergroup/get_all`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| token | string | — | Session token obtained from the authentication flow |
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "Name" | Field to sort by |
| descending | boolean | false | Sort direction |
| name | string | null | Optional filter on user group name |

### Response

Returns a `DTOCollection<UserGroup>` containing the matching user groups.


## Get All User Groups With Details

Returns user groups together with their assigned members, projects, and shared cloud drives inlined.

- **URL**: `/usergroup/get_all_with_details`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| token | string | — | Session token |
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "Name" | Field to sort by |
| descending | boolean | false | Sort direction |
| name | string | null | Optional filter on user group name |

### Response

Returns a collection of `UserGroupWithDetailsDTO` entries.


## Insert User Group

Creates a new user group in the current account.

- **URL**: `/usergroup/insert`
- **Method**: POST
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | Session token |

### Request Body

```json
{
  "name": "Design Team",
  "description": "Users working on the creative project",
  "active": true
}
```

### Response

Returns the created `UserGroup` object including its generated `id`.


## Update User Group

Updates an existing user group.

- **URL**: `/usergroup/update`
- **Method**: PUT
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | Session token |

### Request Body

```json
{
  "id": "00000000-0000-0000-0000-000000000000",
  "name": "Design Team",
  "description": "Updated description",
  "active": true
}
```

### Response

Returns the updated `UserGroup` object.


## Delete User Group

Soft-deletes a user group.

- **URL**: `/usergroup/delete`
- **Method**: DELETE
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | User group id to delete |
| token | string | Session token |

### Response

Returns HTTP 200 on success.


## Get Assigned Users

Returns the paginated list of users assigned to a given user group.

- **URL**: `/usergroup/get_assigned_users`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| token | string | — | Session token |
| userGroupId | string (uuid) | — | Target user group id |
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "User.Username" | Field to sort by |
| descending | boolean | false | Sort direction |
| username | string | null | Optional filter on username |

### Response

Returns a collection of `UserUserGroupDTO` entries representing user-to-group assignments.


## Get Assigned User Groups

Returns the paginated list of user groups that a given user belongs to.

- **URL**: `/usergroup/get_assigned_usergroups`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| token | string | — | Session token |
| userId | string (uuid) | — | Target user id |
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "UserGroup.Name" | Field to sort by |
| descending | boolean | false | Sort direction |
| name | string | null | Optional filter on user group name |

### Response

Returns a collection of `UserUserGroupDTO` entries.


## Assign Users

Assigns one or more users to user groups.

- **URL**: `/usergroup/assign_users`
- **Method**: POST
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | Session token |

### Request Body

An array of `UserUserGroup` objects:

```json
[
  {
    "userId": "00000000-0000-0000-0000-000000000000",
    "userGroupId": "11111111-1111-1111-1111-111111111111"
  }
]
```

### Response

Returns HTTP 200 on success.


## Unassign User

Removes a user-to-user-group assignment.

- **URL**: `/usergroup/unassign_user`
- **Method**: DELETE
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | Id of the assignment (`UserUserGroup`) to remove |
| token | string | Session token |

### Response

Returns HTTP 200 on success.


## Import Users

Bulk-creates user-to-group relationships. For each entry the service maps the provided user to the provided user groups.

- **URL**: `/usergroup/import_users`
- **Method**: POST
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | Session token |

### Request Body

An array of `UserUserGroupRelation` objects:

```json
[
  {
    "user": {
      "username": "user@example.com",
      "firstName": "Ada",
      "lastName": "Lovelace"
    },
    "userGroups": [
      { "id": "00000000-0000-0000-0000-000000000000", "name": "Design Team" }
    ]
  }
]
```

### Response

Returns HTTP 200 on success.


## Sample Code

### Get All User Groups

<details>
<summary>Python</summary>

```python
import requests

response = requests.get(
    "http://localhost:29123/usergroup/get_all",
    params={"token": "EXAMPLE_TOKEN", "page": 1, "pagesize": 20},
)
print(response.json())
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
const res = await fetch(
  "http://localhost:29123/usergroup/get_all?token=EXAMPLE_TOKEN&page=1&pagesize=20"
);
console.log(await res.json());
```

</details>

<details>
<summary>C#</summary>

```csharp
using System.Net.Http;

using HttpClient client = new HttpClient();
HttpResponseMessage res = await client.GetAsync(
    "http://localhost:29123/usergroup/get_all?token=EXAMPLE_TOKEN&page=1&pagesize=20"
);
string body = await res.Content.ReadAsStringAsync();
Console.WriteLine(body);
```

</details>

### Insert User Group

<details>
<summary>Python</summary>

```python
import requests

response = requests.post(
    "http://localhost:29123/usergroup/insert",
    params={"token": "EXAMPLE_TOKEN"},
    json={"name": "Design Team", "description": "Creative project team", "active": True},
)
print(response.json())
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
const res = await fetch(
  "http://localhost:29123/usergroup/insert?token=EXAMPLE_TOKEN",
  {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      name: "Design Team",
      description: "Creative project team",
      active: true,
    }),
  }
);
console.log(await res.json());
```

</details>


For error handling, see [Error Model](errors.md).
