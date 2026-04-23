# Projects Endpoints

This document provides detailed information about the project management endpoints exposed by the Amove desktop agent's Click API. Projects group shared cloud drives and the users/groups that may access them.

> This API is bound to `http://localhost:29123` on a machine running the Amove desktop agent. It is not a hosted service.

## Endpoints

1. [Get All Projects](#get-all-projects)
2. [Get All Projects With Details](#get-all-projects-with-details)
3. [Insert Project](#insert-project)
4. [Update Project](#update-project)
5. [Delete Project](#delete-project)
6. [Get Assigned Shared Cloud Drives](#get-assigned-shared-cloud-drives)
7. [Get Assigned Projects](#get-assigned-projects)
8. [Assign Shared Cloud Drive](#assign-shared-cloud-drive)
9. [Unassign Shared Cloud Drive](#unassign-shared-cloud-drive)


## Get All Projects

Returns the paginated list of projects defined in the current account.

- **URL**: `/projects/get_all`
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
| name | string | null | Optional filter on project name |

### Response

Returns a paginated collection of `Project` entries.


## Get All Projects With Details

Returns projects with their assigned users, user groups, and shared cloud drives inlined.

- **URL**: `/projects/get_all_with_details`
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
| name | string | null | Optional filter on project name |

### Response

Returns a paginated collection of `ProjectWithDetailsDTO` entries.


## Insert Project

Creates a new project in the current account.

- **URL**: `/projects/insert`
- **Method**: POST
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | Session token |

### Request Body

```json
{
  "name": "Campaign 2026",
  "description": "Q1 creative campaign",
  "active": true
}
```

### Response

Returns the created `Project` object including its generated `id`.


## Update Project

Updates an existing project.

- **URL**: `/projects/update`
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
  "name": "Campaign 2026",
  "description": "Updated description",
  "active": true
}
```

### Response

Returns the updated `Project` object.


## Delete Project

Soft-deletes a project.

- **URL**: `/projects/delete`
- **Method**: DELETE
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | Project id to delete |
| token | string | Session token |

### Response

Returns HTTP 200 on success.


## Get Assigned Shared Cloud Drives

Returns the paginated list of shared cloud drives assigned to a given project.

- **URL**: `/projects/get_assigned_sharedclouddrives`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| token | string | — | Session token |
| projectId | string (uuid) | — | Target project id |
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "SharedCloudDrive.Name" | Field to sort by |
| descending | boolean | false | Sort direction |

### Response

Returns a collection of `SharedCloudDriveProjectDTO` entries.


## Get Assigned Projects

Returns the paginated list of projects associated with a given shared cloud drive.

- **URL**: `/projects/get_assigned_projects`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| token | string | — | Session token |
| sharedCloudDriveId | string (uuid) | — | Target shared cloud drive id |
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "Project.Name" | Field to sort by |
| descending | boolean | false | Sort direction |

### Response

Returns a collection of `SharedCloudDriveProjectDTO` entries.


## Assign Shared Cloud Drive

Assigns one or more shared cloud drives to a project.

- **URL**: `/projects/assign_sharedclouddrive`
- **Method**: POST
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | Session token |

### Request Body

An array of `ProjectSharedCloudDrive` objects:

```json
[
  {
    "projectId": "00000000-0000-0000-0000-000000000000",
    "sharedCloudDriveId": "11111111-1111-1111-1111-111111111111"
  }
]
```

### Response

Returns HTTP 200 on success.


## Unassign Shared Cloud Drive

Removes an existing project-to-shared-cloud-drive assignment.

- **URL**: `/projects/unassign_sharedclouddrive`
- **Method**: DELETE
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | Id of the `ProjectSharedCloudDrive` assignment to remove |
| token | string | Session token |

### Response

Returns HTTP 200 on success.


## Sample Code

### Create a Project

<details>
<summary>Python</summary>

```python
import requests

response = requests.post(
    "http://localhost:29123/projects/insert",
    params={"token": "EXAMPLE_TOKEN"},
    json={"name": "Campaign 2026", "description": "Q1 creative", "active": True},
)
print(response.json())
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
const res = await fetch(
  "http://localhost:29123/projects/insert?token=EXAMPLE_TOKEN",
  {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      name: "Campaign 2026",
      description: "Q1 creative",
      active: true,
    }),
  }
);
console.log(await res.json());
```

</details>

### Assign a Shared Cloud Drive to a Project

<details>
<summary>Python</summary>

```python
import requests

response = requests.post(
    "http://localhost:29123/projects/assign_sharedclouddrive",
    params={"token": "EXAMPLE_TOKEN"},
    json=[
        {
            "projectId": "00000000-0000-0000-0000-000000000000",
            "sharedCloudDriveId": "11111111-1111-1111-1111-111111111111",
        }
    ],
)
print(response.status_code)
```

</details>


For error handling, see [Error Model](errors.md).
