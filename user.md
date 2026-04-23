# User Endpoints

This document provides detailed information about the user-management endpoints in the Amove Click API. All endpoints require a `token` query parameter obtained from the [Authentication](authentication.md) flow.

> This API is bound to `http://localhost:29123` on a machine running the Amove desktop agent. It is not a hosted service.


## Endpoints

1. [Get All Users](#get-all-users)
2. [Get All Users With Details](#get-all-users-with-details)
3. [Insert User](#insert-user)
4. [Edit User](#edit-user)
5. [Delete User](#delete-user)
6. [Resend User Email](#resend-user-email)
7. [Set MFA Preference](#set-mfa-preference)
8. [Generate MFA Token](#generate-mfa-token)
9. [Package Inquiry](#package-inquiry)


## Get All Users

Returns a paginated list of users in the signed-in account.

- **URL**: `/user/get_all_users`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| token | string | — | JWT token. |
| page | integer | 1 | Starting page. |
| pagesize | integer | 50 | Page size. |
| sortfield | string | "CreateDate" | Field to sort by. |
| descending | boolean | true | Sort direction. |
| userStatus | integer (enum) | Active \| Inactive \| Pending (7) | Filter by status. Flag enum: `Active=1`, `Inactive=2`, `Pending=4`. |
| userType | integer (enum) | All | Filter by user type. See [UserType](authentication.md#get-user). |
| username | string | null | Substring match on username. |

### Response

```json
{
  "data": [ { "id": "00000000-0000-0000-0000-000000000000", "firstname": "Jane", "lastname": "Doe", "username": "user@example.com", "email": "user@example.com", "userType": 32, "status": 1, "owner": false, "mfa": false, "createDate": "2026-01-01T00:00:00Z" } ],
  "total": 1,
  "options": { "page": 1, "pageSize": 50 }
}
```


## Get All Users With Details

Same as [Get All Users](#get-all-users) but includes each user's assigned groups, projects, and shared cloud drives inline.

- **URL**: `/user/get_all_users_with_details`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

Identical to [Get All Users](#get-all-users).

### Response

Returns a paginated collection of `UserWithDetailsDTO` objects, each extending the base user record with `userGroups`, `projects`, and `sharedCloudDrives` arrays.


## Insert User

Creates a pending user and emails them an activation link. The user has a limited time window to complete sign-up before the token expires.

- **URL**: `/user/insert_user`
- **Method**: POST
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | JWT token. |

### Request Body

```json
{
  "firstname": "Jane",
  "lastname": "Doe",
  "username": "user@example.com",
  "email": "user@example.com",
  "userType": 32
}
```

| Field | Type | Description |
|-------|------|-------------|
| firstname | string | First name. Maximum 50 characters. |
| lastname | string | Last name. Maximum 50 characters. |
| username | string | Unique username. Maximum 50 characters. |
| email | string | Email address. Maximum 50 characters. |
| userType | integer (enum) | Role for the new user. See [UserType](authentication.md#get-user). |

### Response

Returns the created `User` object, including the generated `id`.


## Edit User

Updates an existing user's role or profile fields.

- **URL**: `/user/edit_user`
- **Method**: PUT
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | JWT token. |

### Request Body

A full `User` object, including `id`. Omitted fields keep their prior values on the server; consult the response for the applied state.

### Response

Returns the updated `User` object.


## Delete User

Deletes an account user. The agent also unsubscribes any billing items tied to that user.

- **URL**: `/user/delete_user`
- **Method**: DELETE
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | User id to delete. |
| token | string | JWT token. |

### Response

`200 OK`.


## Resend User Email

Re-sends the activation email for a pending user.

- **URL**: `/user/resend_user_email`
- **Method**: POST
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | JWT token. |

### Request Body

A `User` object identifying the recipient.

### Response

`200 OK`.


## Set MFA Preference

Enables or disables software-token MFA for the signed-in user.

- **URL**: `/user/set_mfa`
- **Method**: POST
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | JWT token. |

### Request Body

```json
{
  "enabled": true,
  "userCode": "123456"
}
```

| Field | Type | Description |
|-------|------|-------------|
| enabled | boolean | `true` to enable MFA, `false` to disable. |
| userCode | string | Current TOTP code from an authenticator app. Required when enabling. |

### Response

`200 OK`.


## Generate MFA Token

Returns a new shared secret for configuring an authenticator app. Call [Set MFA Preference](#set-mfa-preference) afterwards with a fresh code to activate.

- **URL**: `/user/generate_mfa_token`
- **Method**: POST
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | JWT token. |

### Response

A string containing the shared secret (base-32 encoded).


## Package Inquiry

Returns the remaining capacity for the signed-in account across users, storage, and features.

- **URL**: `/user/package_inquiry`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | JWT token. |

### Response

```json
{
  "adminUsers": 1,
  "creativeUsers": 5,
  "standardUsers": 10,
  "storage": 500,
  "connections": 10,
  "projects": 10,
  "teams": 5,
  "drives": 10,
  "syncs": 5,
  "logs": true,
  "sso": false
}
```

Each numeric field is a remaining-count value for the account's current plan.


## Sample Code

### List users

<details>
<summary>Python</summary>

```python
import requests

response = requests.get(
    "http://localhost:29123/user/get_all_users",
    params={"token": "EXAMPLE_TOKEN", "page": 1, "pagesize": 25},
)
print(response.json())
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
const res = await fetch(
  "http://localhost:29123/user/get_all_users?token=EXAMPLE_TOKEN&page=1&pagesize=25"
);
console.log(await res.json());
```

</details>

<details>
<summary>C#</summary>

```csharp
using var client = new HttpClient();
var res = await client.GetAsync(
    "http://localhost:29123/user/get_all_users?token=EXAMPLE_TOKEN&page=1&pagesize=25");
Console.WriteLine(await res.Content.ReadAsStringAsync());
```

</details>


For error handling, see [Error Model](errors.md).
