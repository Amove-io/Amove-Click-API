# SSO Endpoints

This document provides detailed information about the single-sign-on endpoints exposed by the Amove desktop agent. They mirror the Web API SSO flow (Okta, SAML, Entra ID) and also expose the anonymous endpoints needed to complete the browser redirect dance during login.

> This API is bound to `http://localhost:29123` on a machine running the Amove desktop agent. It is not a hosted service.

## Endpoints

1. [Get SSO URL](#get-sso-url)
2. [Get SSO URL for Import User](#get-sso-url-for-import-user)
3. [Authenticate](#authenticate)
4. [Get User UserGroups](#get-user-usergroups)
5. [Setup Okta](#setup-okta)
6. [Get Okta](#get-okta)
7. [Update Okta](#update-okta)
8. [Delete Okta](#delete-okta)
9. [Setup Entra ID](#setup-entra-id)
10. [Get Entra ID](#get-entra-id)
11. [Update Entra ID](#update-entra-id)
12. [Delete Entra ID](#delete-entra-id)
13. [Setup SAML](#setup-saml)
14. [Get SAML](#get-saml)
15. [Update SAML](#update-saml)
16. [Delete SAML](#delete-saml)


## Get SSO URL

Resolves the configured SSO identity provider for a user and returns an authorization URL for the browser redirect.

- **URL**: `/sso/sso_url`
- **Method**: POST
- **Auth Required**: No

### Request Body

```json
{
  "CallbackUrl": "string",
  "Username": "string"
}
```

### Response

Returns an `SSOUrlResponse` containing the identifier of the matching SSO configuration and the provider-specific authorization URL.

```json
{
  "Identifier": "string",
  "Url": "string"
}
```


## Get SSO URL for Import User

Resolves the configured SSO provider for an import-user flow (used when an administrator imports a user from their identity provider into Amove).

- **URL**: `/sso/sso_url_import_user`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "CallbackUrl": "string",
  "Username": "string"
}
```

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | Authentication token. |

### Response

Returns an `SSOUrlResponse` with the identifier and authorization URL.


## Authenticate

Exchanges the authorization code returned by the SSO provider for an Amove JWT.

- **URL**: `/sso/authenticate`
- **Method**: POST
- **Auth Required**: No

### Request Body

```json
{
  "Identifier": "string",
  "AuthorizationCode": "string",
  "CallbackUrl": "string"
}
```

### Response

Returns a JWT string that can be used as the `token` query parameter on subsequent Click API calls.


## Get User UserGroups

Retrieves the list of identity-provider users together with their group memberships. Used by the administrator import flow after the user has completed the SSO redirect.

- **URL**: `/sso/get_user_usergroups`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | Authentication token. |
| authorizationCode | string | Authorization code returned by the identity provider. |
| callbackUrl | string | The callback URL originally supplied to `sso_url_import_user`. |

### Response

Returns a list of `UserUserGroupRelation` objects, each containing a `User` and the list of `UserGroup`s it belongs to on the IdP side.


## Setup Okta

Creates the Okta SSO configuration for the current account.

- **URL**: `/sso/setup_okta`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "ClientId": "string",
  "ClientSecret": "string",
  "OpenIdURL": "string",
  "Active": true
}
```

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | Authentication token. |

### Response

Returns the saved `OktaSSO` object (with `Id` assigned).


## Get Okta

Returns the Okta SSO configuration for the current account.

- **URL**: `/sso/get_okta`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | Authentication token. |

### Response

Returns the `OktaSSO` object for the account, or `null` if none is configured.


## Update Okta

Updates the Okta SSO configuration for the current account.

- **URL**: `/sso/update_okta`
- **Method**: PUT
- **Auth Required**: Yes

### Request Body

Full `OktaSSO` object, including its `Id`.

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | Authentication token. |

### Response

Returns the updated `OktaSSO` object.


## Delete Okta

Deletes the Okta SSO configuration for the current account.

- **URL**: `/sso/delete_okta`
- **Method**: DELETE
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | Authentication token. |

### Response

Returns HTTP 200 on success.


## Setup Entra ID

Creates the Microsoft Entra ID (Azure AD) SSO configuration for the current account.

- **URL**: `/sso/setup_entraId`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "ClientId": "string",
  "ClientSecret": "string",
  "OpenIdURL": "string",
  "Active": true
}
```

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | Authentication token. |

### Response

Returns the saved `EntraIDSSO` object.


## Get Entra ID

Returns the Entra ID SSO configuration for the current account.

- **URL**: `/sso/get_entraId`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | Authentication token. |

### Response

Returns the `EntraIDSSO` object, or `null` if none is configured.


## Update Entra ID

Updates the Entra ID SSO configuration for the current account.

- **URL**: `/sso/update_entraId`
- **Method**: PUT
- **Auth Required**: Yes

### Request Body

Full `EntraIDSSO` object, including its `Id`.

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | Authentication token. |

### Response

Returns the updated `EntraIDSSO` object.


## Delete Entra ID

Deletes the Entra ID SSO configuration for the current account.

- **URL**: `/sso/delete_entraId`
- **Method**: DELETE
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | Authentication token. |

### Response

Returns HTTP 200 on success.


## Setup SAML

Creates the SAML SSO configuration for the current account.

- **URL**: `/sso/setup_saml`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "Certificate": "string (X.509 .crt contents)",
  "SPEntityId": "string",
  "IdPSSOURL": "string",
  "Active": true
}
```

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | Authentication token. |

### Response

Returns the saved `SamlSSO` object.


## Get SAML

Returns the SAML SSO configuration for the current account.

- **URL**: `/sso/get_saml`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | Authentication token. |

### Response

Returns the `SamlSSO` object, or `null` if none is configured.


## Update SAML

Updates the SAML SSO configuration for the current account.

- **URL**: `/sso/update_saml`
- **Method**: PUT
- **Auth Required**: Yes

### Request Body

Full `SamlSSO` object, including its `Id`.

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | Authentication token. |

### Response

Returns the updated `SamlSSO` object.


## Delete SAML

Deletes the SAML SSO configuration for the current account.

- **URL**: `/sso/delete_saml`
- **Method**: DELETE
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | Authentication token. |

### Response

Returns HTTP 200 on success.


## Sample Code

### Get SSO URL then Authenticate

<details>
<summary>Python</summary>

```python
import requests

base = "http://localhost:29123"

# 1. Resolve the user's identity provider and get the authorization URL.
resp = requests.post(
    f"{base}/sso/sso_url",
    json={
        "CallbackUrl": "http://localhost:29123/sso/callback",
        "Username": "user@example.com",
    },
)
sso = resp.json()
print("Open in a browser:", sso["Url"])

# 2. After the IdP redirects back with an authorization code, exchange it for a JWT.
resp = requests.post(
    f"{base}/sso/authenticate",
    json={
        "Identifier": sso["Identifier"],
        "AuthorizationCode": "CODE_FROM_IDP_REDIRECT",
        "CallbackUrl": "http://localhost:29123/sso/callback",
    },
)
jwt = resp.json()
print("Token:", jwt)
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
const base = "http://localhost:29123";

// 1. Resolve the user's identity provider.
const ssoRes = await fetch(`${base}/sso/sso_url`, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    CallbackUrl: "http://localhost:29123/sso/callback",
    Username: "user@example.com",
  }),
});
const sso = await ssoRes.json();
console.log("Open in a browser:", sso.Url);

// 2. Exchange the authorization code for a JWT.
const authRes = await fetch(`${base}/sso/authenticate`, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    Identifier: sso.Identifier,
    AuthorizationCode: "CODE_FROM_IDP_REDIRECT",
    CallbackUrl: "http://localhost:29123/sso/callback",
  }),
});
const jwt = await authRes.json();
console.log("Token:", jwt);
```

</details>

### Configure Okta

<details>
<summary>Python</summary>

```python
import requests

requests.post(
    "http://localhost:29123/sso/setup_okta",
    params={"token": "EXAMPLE_TOKEN"},
    json={
        "ClientId": "okta-client-id-placeholder",
        "ClientSecret": "okta-client-secret-placeholder",
        "OpenIdURL": "https://your-tenant.okta.com/.well-known/openid-configuration",
        "Active": True,
    },
)
```

</details>

<details>
<summary>C#</summary>

```csharp
using var client = new HttpClient();
var payload = new
{
    ClientId = "okta-client-id-placeholder",
    ClientSecret = "okta-client-secret-placeholder",
    OpenIdURL = "https://your-tenant.okta.com/.well-known/openid-configuration",
    Active = true
};
var res = await client.PostAsJsonAsync(
    "http://localhost:29123/sso/setup_okta?token=EXAMPLE_TOKEN", payload);
Console.WriteLine(await res.Content.ReadAsStringAsync());
```

</details>

For error handling, see [Error Model](errors.md).
