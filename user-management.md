# User Management

The User Management endpoints allow you to manage users, user groups, and user permissions within the system.

## Endpoints

### Get All Users

Retrieve all users in the system.

- **URL**: `/User/get_all_users`
- **Method**: GET
- **Auth Required**: Yes

#### Query Parameters

- `token`: string (required)
- `page`: integer (default: 1)
- `pagesize`: integer (default: 50)
- `sortfield`: string (default: "CreateDate")
- `descending`: boolean (default: true)
- `userStatus`: integer (enum)
- `userType`: integer (enum)
- `username`: string (optional)

#### Response

A UserDTOResDesktopObject containing an array of User objects:

```json
{
  "data": [
    {
      "id": "uuid",
      "accountId": "uuid",
      "userType": 0,
      "authProvider": 0,
      "createDate": "2023-05-05T12:00:00Z",
      "signupDate": "2023-05-05T12:00:00Z",
      "firstname": "string",
      "lastname": "string",
      "username": "string",
      "email": "string",
      "mfa": true,
      "owner": false,
      "migration": false,
      "deleted": false,
      "status": 0
    }
  ],
  "total": 0,
  "options": {
    "pageSize": 0,
    "page": 0
  }
}
```

### Create User

Create a new user in the system.

- **URL**: `/User/insert_user`
- **Method**: POST
- **Auth Required**: Yes

#### Query Parameters

- `token`: string (required)

#### Request Body

A User object:

```json
{
  "accountId": "uuid",
  "userType": 0,
  "authProvider": 0,
  "firstname": "string",
  "lastname": "string",
  "username": "string",
  "email": "string",
  "mfa": true,
  "owner": false,
  "migration": false,
  "status": 0
}
```

#### Response

The created User object.

### Update User

Update an existing user's information.

- **URL**: `/User/edit_user`
- **Method**: PUT
- **Auth Required**: Yes

#### Query Parameters

- `token`: string (required)

#### Request Body

A User object with updated information.

#### Response

The updated User object.

### Delete User

Delete a user from the system.

- **URL**: `/User/delete_user`
- **Method**: DELETE
- **Auth Required**: Yes

#### Query Parameters

- `id`: uuid (required)
- `token`: string (required)

### Set MFA Preference

Set the Multi-Factor Authentication preference for a user.

- **URL**: `/User/set_mfa`
- **Method**: POST
- **Auth Required**: Yes

#### Query Parameters

- `token`: string (required)

#### Request Body

An MFAPreference object:

```json
{
  "enabled": true,
  "userCode": "string"
}
```

## Sample Code

Here's an example of how to create a new user using Python:

```python
import requests

API_BASE_URL = "https://api.amoveagent.com"
TOKEN = "your_token_here"

def create_user(account_id, firstname, lastname, username, email, user_type):
    url = f"{API_BASE_URL}/User/insert_user"
    params = {"token": TOKEN}
    data = {
        "accountId": account_id,
        "userType": user_type,
        "authProvider": 0,
        "firstname": firstname,
        "lastname": lastname,
        "username": username,
        "email": email,
        "mfa": False,
        "owner": False,
        "migration": False,
        "status": 1
    }
    
    response = requests.post(url, json=data, params=params)
    
    if response.status_code == 200:
        return response.json()
    else:
        print(f"Error: {response.status_code}")
        print(response.text)
        return None

# Usage
new_user = create_user("account_id", "John", "Doe", "johndoe", "john.doe@example.com", 1)
if new_user:
    print(f"Created user: {new_user['username']} with ID: {new_user['id']}")
```

For other programming languages, you can use their respective HTTP client libraries to make similar requests to the API endpoints.

