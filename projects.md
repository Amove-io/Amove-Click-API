# Projects

The Projects endpoints allow you to manage projects, assign users and shared cloud drives to projects, and handle project permissions.

## Endpoints

### Get All Projects

Retrieve all projects.

- **URL**: `/Projects/get_all`
- **Method**: GET
- **Auth Required**: Yes

#### Query Parameters

- `token`: string (required)
- `page`: integer (default: 1)
- `pagesize`: integer (default: 50)
- `sortfield`: string (default: "Name")
- `descending`: boolean (default: false)
- `name`: string (optional)

#### Response

A ProjectDTOResDesktopObject containing an array of Project objects:

```json
{
  "data": [
    {
      "id": "uuid",
      "accountId": "uuid",
      "name": "string",
      "description": "string",
      "active": true,
      "deleted": false
    }
  ],
  "total": 0,
  "options": {
    "pageSize": 0,
    "page": 0
  }
}
```

### Create Project

Create a new project.

- **URL**: `/Projects/insert`
- **Method**: POST
- **Auth Required**: Yes

#### Query Parameters

- `token`: string (required)

#### Request Body

A Project object:

```json
{
  "accountId": "uuid",
  "name": "string",
  "description": "string",
  "active": true,
  "deleted": false
}
```

#### Response

The created Project object.

### Update Project

Update an existing project.

- **URL**: `/Projects/update`
- **Method**: PUT
- **Auth Required**: Yes

#### Query Parameters

- `token`: string (required)

#### Request Body

A Project object with updated information.

#### Response

The updated Project object.

### Delete Project

Delete a project.

- **URL**: `/Projects/delete`
- **Method**: DELETE
- **Auth Required**: Yes

#### Query Parameters

- `id`: uuid (required)
- `token`: string (required)

### Assign Shared Cloud Drive to Project

Assign one or more shared cloud drives to a project.

- **URL**: `/Projects/assign_sharedclouddrive`
- **Method**: POST
- **Auth Required**: Yes

#### Query Parameters

- `token`: string (required)

#### Request Body

An array of ProjectSharedCloudDrive objects:

```json
[
  {
    "projectId": "uuid",
    "sharedCloudDriveId": "uuid"
  }
]
```

## Sample Code

Here's an example of how to create a new project using Python:

```python
import requests

API_BASE_URL = "https://api.amoveagent.com"
TOKEN = "your_token_here"

def create_project(name, description, account_id):
    url = f"{API_BASE_URL}/Projects/insert"
    params = {"token": TOKEN}
    data = {
        "accountId": account_id,
        "name": name,
        "description": description,
        "active": True,
        "deleted": False
    }
    
    response = requests.post(url, json=data, params=params)
    
    if response.status_code == 200:
        return response.json()
    else:
        print(f"Error: {response.status_code}")
        print(response.text)
        return None

# Usage
new_project = create_project("My New Project", "This is a sample project", "your_account_id")
if new_project:
    print(f"Created project: {new_project['name']} with ID: {new_project['id']}")
```

For other programming languages, you can use their respective HTTP client libraries to make similar requests to the API endpoints.

