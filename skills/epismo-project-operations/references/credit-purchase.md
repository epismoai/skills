# Credit Purchase

When the MCP server returns an error indicating depleted credits, you will need to purchase additional credits via the API to continue.

**Pricing: 1 credit = $0.01**

## Overview

1. **Acquire Access Token**: Use the email OTP flow to get an `accessToken`.
2. **Create Checkout Session**: Call `POST /v1/credits` to generate a Stripe Checkout URL.
3. **Complete Purchase**: Open the URL, finalize the payment, and retry your MCP request.

## 1. Acquire an Access Token (Email OTP)

To purchase credits, you need a valid `accessToken`. If you don't have an active one, follow these steps to obtain it.

### Step 1: Request an OTP ID

Provide your email to receive a PIN.

```bash
curl -sX POST https://api.epismo.ai/v1/otp-tokens \
  -H "Content-Type: application/json" \
  -d '{"email":"user@example.com"}'

# Response:
# {"otpId":"..."}
```

### Step 2: Exchange PIN for an Access Token

Use the `otpId` from Step 1 and the PIN you received via email.

```bash
curl -sX POST https://api.epismo.ai/v1/users \
  -H "Content-Type: application/json" \
  -d '{"otpId":"...","pin":"YOUR_PIN"}'

# Response:
# {"userId":"...","accessToken":"..."}
```

## 2. Purchase Credits (Create Stripe Checkout)

With your `accessToken` ready, call the `POST /v1/credits` endpoint to generate a Stripe Checkout URL.

- `workspaceId` (Optional): Target workspace for the credits. If omitted, credits apply to your personal workspace. (Use the MCP resource `context:current_user` to find your `workspace.id`).
- `allocations` (Required): An array specifying who receives the credits.
  - `userId`: The recipient's user ID. (Use the MCP resource `context:users` to find this).
  - `quantity`: Number of credits to purchase (integer).

```bash
curl -sX POST https://api.epismo.ai/v1/credits \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -d '{
    "workspaceId": "...",
    "allocations": [
      {
        "userId": "...",
        "quantity": 1000
      }
    ]
  }'

# Response:
# {"url":"https://checkout.stripe.com/..."}
```

## 3. Complete the Purchase

Open the returned Stripe Checkout `url` in your browser to process the payment. Once completed:

1. Credits will be immediately added to the specified balance.
2. You can verify your updated balance in the dashboard if desired.
3. You can now retry the MCP request that previously failed due to insufficient credits.
