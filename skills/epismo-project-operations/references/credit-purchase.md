# Credit Purchase

When API operations return `Payment Required: Insufficient credits`, purchase credits and retry the same API request.

**Pricing: 1 credit = $0.01**

## Overview

1. If needed, get a valid API session token.
2. Call `POST /v1/credits` to generate a Stripe Checkout URL.
3. Complete checkout and retry the failed API request.

## 1. Get a Session Token (If Needed)

If you do not already have a `sessionToken`, request an OTP and exchange it with the PIN sent to your email.

```bash
curl -sX POST https://api.epismo.ai/v1/otp-tokens \
  -H "Content-Type: application/json" \
  -d '{"email":"you@example.com"}'

# => {"otpId":"<OTP_ID>"}
```

```bash
curl -sX POST https://api.epismo.ai/v1/users \
  -H "Content-Type: application/json" \
  -d '{"otpId":"<OTP_ID>","pin":"<PIN_FROM_EMAIL>"}'

# => {"sessionToken":"<SESSION_TOKEN>", ...}
```

## 2. Purchase Credits (Create Stripe Checkout)

Call `POST /v1/credits` with `Authorization: Bearer <SESSION_TOKEN>`.

- `workspaceId` (optional): target workspace. If provided, the caller must belong to that workspace.
- `allocations`: recipients and quantities.
  - `userId`: recipient user ID.
  - `quantity`: number of credits (integer).

```bash
curl -sX POST https://api.epismo.ai/v1/credits \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <SESSION_TOKEN>" \
  -d '{
    "workspaceId": "<WORKSPACE_ID>",
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

Open the returned Stripe Checkout `url` in your browser and complete payment.

1. Credits are added to the specified balance.
2. Optionally verify balance in the dashboard.
3. Retry the API call that failed with insufficient credits.
4. If the session expired while you were purchasing, rerun the OTP flow and retry.
