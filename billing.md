# Billing Endpoints

This document provides detailed information about the billing and subscription endpoints in the AMove Click API.

> This API is bound to `http://localhost:29123` on a machine running the Amove desktop agent. It is not a hosted service.


## Endpoints

1. [Billing Status](#billing-status)
2. [Billing Subscribe](#billing-subscribe)
3. [Billing Unsubscribe](#billing-unsubscribe)
4. [Active Subscriptions](#active-subscriptions)


## Billing Status

Returns the primary subscription for the signed-in user.

- **URL**: `/billing/billing_status`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | JWT token. |

### Response

```json
{
  "status": "active",
  "currentPeriodStart": "2026-01-01T00:00:00Z",
  "currentPeriodEnd": "2026-02-01T00:00:00Z",
  "priceId": "price_example",
  "paymentMethod": 1,
  "subscriptionType": 1,
  "subscriptionTitle": "App",
  "price": 19.99
}
```

| Field | Type | Description |
|-------|------|-------------|
| status | string | Subscription status string from the payment provider. |
| currentPeriodStart | string (date-time) | Start of the current billing period. |
| currentPeriodEnd | string (date-time) | End of the current billing period. |
| priceId | string | Payment-provider price identifier. |
| paymentMethod | integer (enum) | `1` (Stripe) or `2` (DirectTransfer). |
| subscriptionType | integer (enum) | Product type. See the `SubscriptionType` enum. Common values: `1` (App), `2` (Storage), `4` (AdminUser), `8` (CreativeUser), `16` (StandardUser). |
| subscriptionTitle | string | Human-readable subscription name. |
| price | number | Amount billed per period. |


## Billing Subscribe

Starts a subscription for the supplied price and returns a URL the client can open to complete payment.

- **URL**: `/billing/billing_subscribe`
- **Method**: POST
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | JWT token. |

### Request Body

```json
{
  "baseAddress": "http://localhost:29123",
  "priceId": "price_example"
}
```

| Field | Type | Description |
|-------|------|-------------|
| baseAddress | string | Return URL base the payment provider should redirect to after checkout. |
| priceId | string | Payment-provider price identifier for the plan to subscribe to. |

### Response

A string containing the checkout URL to open in a browser.


## Billing Unsubscribe

Cancels the active subscription for the signed-in user.

- **URL**: `/billing/billing_unsubscribe`
- **Method**: POST
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | JWT token. |

### Response

Boolean. `true` when the cancellation succeeded.


## Active Subscriptions

Returns all active subscriptions for the signed-in user, including add-ons such as storage or user-seat packs.

- **URL**: `/billing/active_subscriptions`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | JWT token. |

### Response

Array of subscription objects in the same shape as [Billing Status](#billing-status).


## Sample Code

### Start a subscription

<details>
<summary>Python</summary>

```python
import requests

response = requests.post(
    "http://localhost:29123/billing/billing_subscribe",
    params={"token": "EXAMPLE_TOKEN"},
    json={
        "baseAddress": "http://localhost:29123",
        "priceId": "price_example",
    },
)
print(response.text)  # checkout URL
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
const res = await fetch(
  "http://localhost:29123/billing/billing_subscribe?token=EXAMPLE_TOKEN",
  {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      baseAddress: "http://localhost:29123",
      priceId: "price_example",
    }),
  }
);
console.log(await res.text()); // checkout URL
```

</details>

<details>
<summary>C#</summary>

```csharp
using System.Net.Http.Json;

using var client = new HttpClient();
HttpResponseMessage res = await client.PostAsJsonAsync(
    "http://localhost:29123/billing/billing_subscribe?token=EXAMPLE_TOKEN",
    new { baseAddress = "http://localhost:29123", priceId = "price_example" });
Console.WriteLine(await res.Content.ReadAsStringAsync()); // checkout URL
```

</details>


For error handling, see [Error Model](errors.md).
