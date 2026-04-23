# UsersPermission Endpoints

This document provides detailed information about the permission management endpoints exposed by the Amove desktop agent's Click API. Permissions associate users or user groups with projects or shared cloud drives, at one of two levels: `Read` or `ReadWrite`.

> This API is bound to `http://localhost:29123` on a machine running the Amove desktop agent. It is not a hosted service.

All endpoints on this page live under the `/userspermission` route prefix.

## Permission Type

Every permission entry carries a `permissionType` value:

| Value | Meaning |
|---|---|
| `1` | Read |
| `2` | ReadWrite |

## Endpoints

The controller exposes four CRUD groups with five endpoints each. Jump to the relevant section:

- [User to Project](#user-to-project)
  1. [Add User-Project Permission](#add-user-project-permission)
  2. [Edit User-Project Permission](#edit-user-project-permission)
  3. [Delete User-Project Permission](#delete-user-project-permission)
  4. [Get Users Assigned to Project](#get-users-assigned-to-project)
  5. [Get Projects Assigned to User](#get-projects-assigned-to-user)
- [User to SharedCloudDrive](#user-to-sharedclouddrive)
  1. [Add User-SharedCloudDrive Permission](#add-user-sharedclouddrive-permission)
  2. [Edit User-SharedCloudDrive Permission](#edit-user-sharedclouddrive-permission)
  3. [Delete User-SharedCloudDrive Permission](#delete-user-sharedclouddrive-permission)
  4. [Get Users Assigned to Shared Cloud Drive](#get-users-assigned-to-shared-cloud-drive)
  5. [Get Shared Cloud Drives Assigned to User](#get-shared-cloud-drives-assigned-to-user)
- [UserGroup to Project](#usergroup-to-project)
  1. [Add UserGroup-Project Permission](#add-usergroup-project-permission)
  2. [Edit UserGroup-Project Permission](#edit-usergroup-project-permission)
  3. [Delete UserGroup-Project Permission](#delete-usergroup-project-permission)
  4. [Get User Groups Assigned to Project](#get-user-groups-assigned-to-project)
  5. [Get Projects Assigned to User Group](#get-projects-assigned-to-user-group)
- [UserGroup to SharedCloudDrive](#usergroup-to-sharedclouddrive)
  1. [Add UserGroup-SharedCloudDrive Permission](#add-usergroup-sharedclouddrive-permission)
  2. [Edit UserGroup-SharedCloudDrive Permission](#edit-usergroup-sharedclouddrive-permission)
  3. [Delete UserGroup-SharedCloudDrive Permission](#delete-usergroup-sharedclouddrive-permission)
  4. [Get User Groups Assigned to Shared Cloud Drive](#get-user-groups-assigned-to-shared-cloud-drive)
  5. [Get Shared Cloud Drives Assigned to User Group](#get-shared-cloud-drives-assigned-to-user-group)


## User to Project

### Add User-Project Permission

Grants one or more users a permission on a project.

- **URL**: `/userspermission/users_project_permission`
- **Method**: POST
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | Session token |

#### Request Body

An array of `UserProjectPermission` objects:

```json
[
  {
    "userId": "00000000-0000-0000-0000-000000000000",
    "projectId": "11111111-1111-1111-1111-111111111111",
    "permissionType": 2
  }
]
```

#### Response

Returns HTTP 200 on success.


### Edit User-Project Permission

Updates existing user-project permission entries.

- **URL**: `/userspermission/users_project_permission`
- **Method**: PUT
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | Session token |

#### Request Body

An array of `UserProjectPermission` objects (each including its `id`):

```json
[
  {
    "id": "22222222-2222-2222-2222-222222222222",
    "userId": "00000000-0000-0000-0000-000000000000",
    "projectId": "11111111-1111-1111-1111-111111111111",
    "permissionType": 1
  }
]
```

#### Response

Returns HTTP 200 on success.


### Delete User-Project Permission

Removes a user-project permission entry by its id.

- **URL**: `/userspermission/users_project_permission`
- **Method**: DELETE
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | Permission entry id |
| token | string | Session token |

#### Response

Returns HTTP 200 on success.


### Get Users Assigned to Project

Returns the paginated list of users assigned to a project.

- **URL**: `/userspermission/get_users_assigned_to_project`
- **Method**: GET
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| token | string | — | Session token |
| projectId | string (uuid) | — | Target project id |
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "User.Username" | Field to sort by |
| descending | boolean | false | Sort direction |
| username | string | null | Optional filter on username |

#### Response

Returns a collection of `UserProjectPermissionDTO` entries.


### Get Projects Assigned to User

Returns the paginated list of projects a given user is assigned to.

- **URL**: `/userspermission/get_projects_assigned_to_user`
- **Method**: GET
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| token | string | — | Session token |
| userId | string (uuid) | — | Target user id |
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "Project.Name" | Field to sort by |
| descending | boolean | false | Sort direction |
| name | string | null | Optional filter on project name |

#### Response

Returns a collection of `UserProjectPermissionDTO` entries.


## User to SharedCloudDrive

> Note: the route below uses the legacy spelling `users_sharedclouddrive_permision` (one "s" in "permision"). This is the actual URL exposed by the agent.

### Add User-SharedCloudDrive Permission

Grants one or more users a permission on a shared cloud drive.

- **URL**: `/userspermission/users_sharedclouddrive_permision`
- **Method**: POST
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | Session token |

#### Request Body

An array of `UserSharedCloudDrivePermission` objects:

```json
[
  {
    "userId": "00000000-0000-0000-0000-000000000000",
    "sharedCloudDriveId": "11111111-1111-1111-1111-111111111111",
    "permissionType": 2
  }
]
```

#### Response

Returns HTTP 200 on success.


### Edit User-SharedCloudDrive Permission

Updates existing user-shared-cloud-drive permission entries.

- **URL**: `/userspermission/users_sharedclouddrive_permision`
- **Method**: PUT
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | Session token |

#### Request Body

An array of `UserSharedCloudDrivePermission` objects (each including its `id`).

#### Response

Returns HTTP 200 on success.


### Delete User-SharedCloudDrive Permission

Removes a user-shared-cloud-drive permission entry by its id.

- **URL**: `/userspermission/users_sharedclouddrive_permision`
- **Method**: DELETE
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | Permission entry id |
| token | string | Session token |

#### Response

Returns HTTP 200 on success.


### Get Users Assigned to Shared Cloud Drive

Returns the paginated list of users assigned to a shared cloud drive.

- **URL**: `/userspermission/get_users_assigned_to_sharedclouddrive`
- **Method**: GET
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| token | string | — | Session token |
| sharedCloudDriveId | string (uuid) | — | Target shared cloud drive id |
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "User.Username" | Field to sort by |
| descending | boolean | false | Sort direction |
| username | string | null | Optional filter on username |

#### Response

Returns a collection of `UserSharedCloudDrivePermissionDTO` entries.


### Get Shared Cloud Drives Assigned to User

Returns the paginated list of shared cloud drives a given user has access to.

- **URL**: `/userspermission/get_sharedclouddrive_assigned_to_user`
- **Method**: GET
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| token | string | — | Session token |
| userId | string (uuid) | — | Target user id |
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "SharedClouDrive.Name" | Field to sort by |
| descending | boolean | false | Sort direction |
| name | string | null | Optional filter on drive name |

#### Response

Returns a collection of `UserSharedCloudDrivePermissionDTO` entries.


## UserGroup to Project

### Add UserGroup-Project Permission

Grants one or more user groups a permission on a project.

- **URL**: `/userspermission/usergroups_project_permission`
- **Method**: POST
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | Session token |

#### Request Body

An array of `UserGroupProjectPermission` objects:

```json
[
  {
    "userGroupId": "00000000-0000-0000-0000-000000000000",
    "projectId": "11111111-1111-1111-1111-111111111111",
    "permissionType": 2
  }
]
```

#### Response

Returns HTTP 200 on success.


### Edit UserGroup-Project Permission

Updates existing user-group-to-project permission entries.

- **URL**: `/userspermission/usergroups_project_permission`
- **Method**: PUT
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | Session token |

#### Request Body

An array of `UserGroupProjectPermission` objects (each including its `id`).

#### Response

Returns HTTP 200 on success.


### Delete UserGroup-Project Permission

Removes a user-group-to-project permission entry by its id.

- **URL**: `/userspermission/usergroups_project_permission`
- **Method**: DELETE
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | Permission entry id |
| token | string | Session token |

#### Response

Returns HTTP 200 on success.


### Get User Groups Assigned to Project

Returns the paginated list of user groups assigned to a project.

- **URL**: `/userspermission/get_usergroups_assigned_to_project`
- **Method**: GET
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| token | string | — | Session token |
| projectId | string (uuid) | — | Target project id |
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "UserGroup.Name" | Field to sort by |
| descending | boolean | false | Sort direction |
| name | string | null | Optional filter on user group name |

#### Response

Returns a collection of `UserGroupProjectPermissionDTO` entries.


### Get Projects Assigned to User Group

Returns the paginated list of projects a given user group is assigned to.

- **URL**: `/userspermission/get_projects_assigned_to_usergroup`
- **Method**: GET
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| token | string | — | Session token |
| userGroupId | string (uuid) | — | Target user group id |
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "Project.Name" | Field to sort by |
| descending | boolean | false | Sort direction |
| name | string | null | Optional filter on project name |

#### Response

Returns a collection of `UserGroupProjectPermissionDTO` entries.


## UserGroup to SharedCloudDrive

### Add UserGroup-SharedCloudDrive Permission

Grants one or more user groups a permission on a shared cloud drive.

- **URL**: `/userspermission/usergroup_sharedclouddrive_permission`
- **Method**: POST
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | Session token |

#### Request Body

An array of `UserGroupSharedCloudDrivePermission` objects:

```json
[
  {
    "userGroupId": "00000000-0000-0000-0000-000000000000",
    "sharedCloudDriveId": "11111111-1111-1111-1111-111111111111",
    "permissionType": 2
  }
]
```

#### Response

Returns HTTP 200 on success.


### Edit UserGroup-SharedCloudDrive Permission

Updates existing user-group-to-shared-cloud-drive permission entries.

- **URL**: `/userspermission/usergroup_sharedclouddrive_permission`
- **Method**: PUT
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | Session token |

#### Request Body

An array of `UserGroupSharedCloudDrivePermission` objects (each including its `id`).

#### Response

Returns HTTP 200 on success.


### Delete UserGroup-SharedCloudDrive Permission

Removes a user-group-to-shared-cloud-drive permission entry by its id.

- **URL**: `/userspermission/usergroup_sharedclouddrive_permission`
- **Method**: DELETE
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | Permission entry id |
| token | string | Session token |

#### Response

Returns HTTP 200 on success.


### Get User Groups Assigned to Shared Cloud Drive

Returns the paginated list of user groups assigned to a shared cloud drive.

- **URL**: `/userspermission/get_usergroups_assigned_to_sharedclouddrive`
- **Method**: GET
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| token | string | — | Session token |
| sharedCloudDrive | string (uuid) | — | Target shared cloud drive id |
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "UserGroup.Name" | Field to sort by |
| descending | boolean | false | Sort direction |
| name | string | null | Optional filter on user group name |

#### Response

Returns a collection of `UserGroupSharedCloudDrivePermissionDTO` entries.


### Get Shared Cloud Drives Assigned to User Group

Returns the paginated list of shared cloud drives a given user group can access.

- **URL**: `/userspermission/get_sharedclouddrive_assigned_to_usergroup`
- **Method**: GET
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| token | string | — | Session token |
| userGroupId | string (uuid) | — | Target user group id |
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "SharedCloudDrive.Name" | Field to sort by |
| descending | boolean | false | Sort direction |
| name | string | null | Optional filter on drive name |

#### Response

Returns a collection of `UserGroupSharedCloudDrivePermissionDTO` entries.


## Sample Code

### Grant a User ReadWrite access to a Project

<details>
<summary>Python</summary>

```python
import requests

response = requests.post(
    "http://localhost:29123/userspermission/users_project_permission",
    params={"token": "EXAMPLE_TOKEN"},
    json=[
        {
            "userId": "00000000-0000-0000-0000-000000000000",
            "projectId": "11111111-1111-1111-1111-111111111111",
            "permissionType": 2,
        }
    ],
)
print(response.status_code)
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
const res = await fetch(
  "http://localhost:29123/userspermission/users_project_permission?token=EXAMPLE_TOKEN",
  {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify([
      {
        userId: "00000000-0000-0000-0000-000000000000",
        projectId: "11111111-1111-1111-1111-111111111111",
        permissionType: 2,
      },
    ]),
  }
);
console.log(res.status);
```

</details>

### List Users Assigned to a Shared Cloud Drive

<details>
<summary>Python</summary>

```python
import requests

response = requests.get(
    "http://localhost:29123/userspermission/get_users_assigned_to_sharedclouddrive",
    params={
        "token": "EXAMPLE_TOKEN",
        "sharedCloudDriveId": "00000000-0000-0000-0000-000000000000",
        "page": 1,
        "pagesize": 20,
    },
)
print(response.json())
```

</details>


For error handling, see [Error Model](errors.md).
