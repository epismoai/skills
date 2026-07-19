# Credit Purchase

When an operation returns `Payment Required: Insufficient credits`, check the current credit state, purchase credits if needed, and retry the same request or command.

**Pricing: 1 credit = $0.01; minimum purchase = 500 credits ($5)**

Free credits are personal. Workspace credits are shared by every member. Each workspace includes 50,000 credits per month and costs $300 per month. Purchased credits go directly to the selected personal or workspace balance—there is no member allocation step. Only workspace owners can purchase workspace credits.

There is no MCP tool for `credit`. Use the CLI flow below, or use the HTTP API described afterward.

### Step 1. Authenticate (If Needed)

```bash
epismo login --email you@example.com
epismo login --browser
```

### Step 2. Check Credit Status

```bash
epismo credit balance
```

### Step 3. Purchase Credits (Create Stripe Checkout)

```bash
epismo credit checkout --quantity 500
epismo credit checkout --input @checkout.json
```

### Step 4. Complete the Purchase

Open the returned Stripe Checkout URL in your browser and complete payment.

1. Credits are added to the specified balance.
2. Verify the updated balance with `epismo credit balance`.
3. Retry the CLI call that failed with insufficient credits.
4. If your session expired while purchasing, run `epismo login` again and retry.

### API

#### Step 1. Get an OAuth Access Token (If Needed)

If you do not already have an `access_token`, request an OTP and exchange it with the PIN sent to your email.

```bash
curl -sX POST https://api.epismo.ai/v1/otp-tokens \
  -H "Content-Type: application/json" \
  -d '{"email":"you@example.com"}'

# => {"otpId":"<OTP_ID>"}
```

```bash
curl -sX POST https://api.epismo.ai/oauth/token \
  -H "Content-Type: application/json" \
  -d '{"grant_type":"otp","otp_id":"<OTP_ID>","pin":"<PIN_FROM_EMAIL>"}'

# => {"access_token":"<ACCESS_TOKEN>","refresh_token":"<REFRESH_TOKEN>","token_type":"Bearer","expires_in":3600,"scope":"read write offline_access","user_id":"<USER_ID>"}
```

#### Step 2. Check Credit Status

Call `GET /v1/credits` with `Authorization: Bearer <ACCESS_TOKEN>`.

- `workspaceId` (optional query): target workspace. If provided, the caller must belong to that workspace.
- Response: `{ "balance": <NUMBER>, "shortfall": <NUMBER> }`

```bash
curl -sX GET "https://api.epismo.ai/v1/credits?workspaceId=<workspace-id>" \
  -H "Authorization: Bearer <ACCESS_TOKEN>"

# => {"balance":100,"shortfall":0}
```

Omit the `workspaceId` query parameter to check personal scope.

#### Step 3. Purchase Credits (Create Stripe Checkout)

Call `POST /v1/credits` with `Authorization: Bearer <ACCESS_TOKEN>`.

- `workspaceId` (optional query): target Pro workspace. If provided, the caller must be its owner.
- `quantity`: number of credits; a positive integer of at least 500.

```bash
curl -sX POST "https://api.epismo.ai/v1/credits?workspaceId=<workspace-id>" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <ACCESS_TOKEN>" \
  -d '{"quantity":1000}'

# Response:
# {"url":"https://checkout.stripe.com/..."}
```

#### Step 4. Complete the Purchase

Open the returned Stripe Checkout `url` in your browser and complete payment.

1. Credits are added to the specified balance.
2. Verify the updated balance with `GET /v1/credits`.
3. Retry the API call that failed with insufficient credits.
4. If the access token expired while purchasing, rerun the OTP flow and retry.
