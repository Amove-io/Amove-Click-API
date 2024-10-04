# Authentication

The AmoveAgent API uses various authentication methods depending on the endpoint. This document outlines the available authentication endpoints and their usage.

## Endpoints

### Login

Authenticate a user and receive a JWT token.

- **URL**: `/Authentication/login`
- **Method**: POST
- **Auth Required**: No

#### Request Body

```json
{
  "username": "string",
  "password": "string",
  "type": 0,
  "requestToken": "string"
}
```

#### Response

```json
{
  "jwtToken": "string",
  "challenge": 0,
  "session": "string",
  "requestToken": "string"
}
```

### Logout

Log out a user.

- **URL**: `/Authentication/logout`
- **Method**: POST
- **Auth Required**: Yes

#### Request Body

```json
{
  "requestToken": "string",
  "jwtToken": "string"
}
```

### Multi-Factor Authentication Response

Respond to a multi-factor authentication challenge.

- **URL**: `/Authentication/mfa_respond`
- **Method**: POST
- **Auth Required**: No

#### Request Body

```json
{
  "userCode": "string",
  "session": "string",
  "requestToken": "string"
}
```

#### Response

```json
{
  "jwtToken": "string",
  "challenge": 0,
  "session": "string",
  "requestToken": "string"
}
```

### Get User Information

Retrieve information about the authenticated user.

- **URL**: `/Authentication/user`
- **Method**: GET
- **Auth Required**: Yes

#### Query Parameters

- `token`: string (required)

#### Response

```json
{
  "accountName": "string",
  "accountId": "string",
  "userId": "string",
  "firstName": "string",
  "lastName": "string",
  "username": "string",
  "userType": 0,
  "jwtToken": "string",
  "owner": true,
  "migration": true,
  "subscriptionType": 0,
  "mfa": true,
  "authProvider": 0
}
```

## Sample Code

Here's an example of how to log in using Python:

```python
import requests

API_BASE_URL = "https://api.amoveagent.com"

def login(username, password, user_type):
    url = f"{API_BASE_URL}/Authentication/login"
    data = {
        "username": username,
        "password": password,
        "type": user_type
    }
    
    response = requests.post(url, json=data)
    
    if response.status_code == 200:
        return response.json()
    else:
        print(f"Error: {response.status_code}")
        print(response.text)
        return None

# Usage
login_result = login("your_username", "your_password", 1)
if login_result:
    print(f"JWT Token: {login_result['jwtToken']}")
    print(f"Session: {login_result['session']}")
```

For other programming languages, you can use their respective HTTP client libraries to make similar requests to the API endpoints.

