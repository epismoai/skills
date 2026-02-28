# Epismo Skills

Epismo Skills help makes human-AI project operations portable, visible, and repeatable.

## Why This Exists

Teams using AI repeatedly hit the same problems:

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

1. Create a Secret Key (choose one)

> Note: `secretKey` is shown only once. Store it securely.

**Option A: via UI**

- A-1. Open Epismo and go to `Settings > MCP Servers`.
- A-2. Click **Create Secret Key**.
- A-3. Copy the key

**Option B: via API**

- B-1. Request an OTP (sent to your email)

```bash
curl -sX POST https://api.epismo.ai/v1/otp-tokens \
  -H "Content-Type: application/json" \
  -d '{"email":"you@example.com"}'

# => {"otpId":"..."}
```

- B-2. Create (or fetch) a user and get an access token

```bash
curl -sX POST https://api.epismo.ai/v1/users \
  -H "Content-Type: application/json" \
  -d '{"email":"you@example.com","otpId":"...","pin":"YOUR_PIN"}'

# => {"userId":"...","accessToken":"..."}
```

`accessToken` is a short-lived access token used only for issuing `secretKey` via API.

- B-3. Issue a Secret Key (use accessToken in header, userId in body)

```bash
curl -sX POST https://api.epismo.ai/v1/secret-keys \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -d '{"userId":"..."}'

# => {"secretKey":"..."}
```

2. Add the MCP server to your client

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

3. Set the Skills in your environment

## Source of Truth

- Source repository: [github.com/epismoai/skills](https://github.com/epismoai/skills)

If you customize locally, review upstream diffs and merge selectively.
