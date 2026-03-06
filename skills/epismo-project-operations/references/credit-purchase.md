# Credit Purchase

When API operations return `Payment Required: Insufficient credits`, purchase credits and retry the same API request.

**Pricing: 1 credit = $0.01**

## Overview

1. Ensure you have `EPISMO_SECRET_KEY`.
2. Call `POST /v1/credits` to generate a Stripe Checkout URL.
3. Complete checkout and retry the failed API request.

## Secret Key Prerequisite

This document assumes `EPISMO_SECRET_KEY` is already available.
For secret-key issuance, the best source of truth is the `epismoai/skills` README.

How to read it:

1. Browser
   - Open: `https://github.com/epismoai/skills/blob/main/README.md`
2. CLI
   - `curl -fsSL https://raw.githubusercontent.com/epismoai/skills/main/README.md`

In that README, follow **Step 1: Create a Secret Key**, which covers:

1. Request an OTP (`POST /v1/otp-tokens`)
2. Verify the OTP and get an access token (`POST /v1/users`)
3. Issue a Secret Key (`POST /v1/secret-keys`)

## 1. Purchase Credits (Create Stripe Checkout)

Call `POST /v1/credits` with `Authorization: Bearer $EPISMO_SECRET_KEY`.

- `allocations`: recipients and quantities.
  - `userId`: recipient user ID.
  - `quantity`: number of credits (integer).

```bash
curl -sX POST https://api.epismo.ai/v1/credits \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $EPISMO_SECRET_KEY" \
  -d '{
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

## 2. Complete the Purchase

Open the returned Stripe Checkout `url` in your browser and complete payment.

1. Credits are added to the specified balance.
2. Optionally verify balance in the dashboard.
3. Retry the API call that failed with insufficient credits.
