# Epismo Skills

Epismo Skills help makes human-AI project operations portable, visible, and repeatable.

## Why This Exists

People using AI repeatedly hit the same problems:

- workflows stay personal and fragile
- know-how is scattered across chats
- prompts are easy to copy, but processes are hard to reuse

## What You Can Do

You can start from any point. Pick the use case and run a prompt.

| Use case                              | Prompt example                                                   |
| ------------------------------------- | ---------------------------------------------------------------- |
| Start from current project state      | "Analyze this project and suggest the best workflow to start."   |
| Find and reuse best practices         | "Import a community workflow and adapt it to this project."      |
| Turn context into executable work     | "Convert this blog post into a reusable workflow for my team."   |
| Split AI execution and human control  | "Run automatable steps with AI and keep final approval with me." |
| Capture successful patterns for reuse | "This worked well. Save it as a reusable workflow template."     |

## Quick Start

> Note: `secretKey` is shown only once. Store it securely.

### Step 1: Create a Secret Key

Choose one option.

**Option A: Create via UI**

1. Go to `Settings > MCP Servers`.
2. Click **Create Secret Key**.
3. Copy the key.

**Option B: Create via API**

1. Request an OTP (sent to your email)

```bash
curl -sX POST https://api.epismo.ai/v1/otp-tokens \
  -H "Content-Type: application/json" \
  -d '{"email":"you@example.com"}'

# => {"otpId":"..."}
```

2. Create (or fetch) a user and get an access token

```bash
curl -sX POST https://api.epismo.ai/v1/users \
  -H "Content-Type: application/json" \
  -d '{"otpId":"...","pin":"YOUR_PIN"}'

# => {"userId":"...","accessToken":"..."}
```

- `accessToken` is short-lived.
- It is only used to issue a `secretKey` via API.

3. Issue a Secret Key

Use `accessToken` in the header and `workspaceId` (optional) in the body.
If `workspaceId` is omitted, the key will be scoped to your personal workspace.
You can get your workspace IDs by calling `GET https://api.epismo.ai/v1/workspaces` with the `accessToken`.

```bash
curl -sX POST https://api.epismo.ai/v1/secret-keys \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -d '{"workspaceId":"..."}'

# => {"secretKey":"..."}
```

### Step 2: Add the MCP server to your client

MCP endpoint:

- URL: `https://mcp.epismo.ai/`
- Protocol: Streamable HTTP (`POST /`, JSON-RPC 2.0)
- Auth header: `Authorization: Bearer YOUR_SECRET_KEY`

Example client config:

```json
{
  "name": "epismo-mcp",
  "url": "https://mcp.epismo.ai/",
  "headers": {
    "Authorization": "Bearer YOUR_SECRET_KEY"
  }
}
```

### Step 3: Set the Skills in your environment

Set up Epismo Skills in the environment your client runs in (for example, agent config, workspace settings, or CI).

## Source of truth

Repository: `https://github.com/epismoai/skills`

If you customize locally, review upstream diffs and merge selectively.
