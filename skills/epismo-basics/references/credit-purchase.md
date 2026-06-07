# Credit Purchase

When an operation returns `Payment Required: Insufficient credits`, check the current credit state, purchase credits if needed, and retry the same request or command.

**Pricing: 1 credit = $0.01**

**Minimum purchase: 500 credits ($5.00) total per checkout**

Credits are **CLI-only** — there is no MCP tool for `credit`. Run the steps below with `epismo`.

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
epismo credit checkout --allocations '[{"userId":"<USER_ID>","quantity":500}]'
epismo credit checkout --input @checkout.json
```

### Step 4. Complete the Purchase

Open the returned Stripe Checkout URL in your browser and complete payment.

1. Credits are added to the specified balance.
2. Verify the updated balance with `epismo credit balance`.
3. Retry the CLI call that failed with insufficient credits.
4. If your session expired while purchasing, run `epismo login` again and retry.
