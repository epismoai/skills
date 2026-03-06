# Epismo Skills

Reusable skill packages that give AI agents structured project-operation capabilities through Epismo MCP.

## Why Skills

AI agents hit the same problems across teams and projects:

- Operational know-how stays trapped in chat histories.
- Prompts copy easily, but multi-step processes don't.
- Every new project restarts from scratch.

Skills solve this by packaging proven operational patterns — intake, planning, coordination, risk handling, workflow release — into portable instruction sets that any MCP-connected agent can follow.

## What You Can Do

Pick a use case and run a prompt. Skills handle the rest.

| Use case                              | Example prompt                                                   |
| ------------------------------------- | ---------------------------------------------------------------- |
| Start from current project state      | "Analyze this project and suggest the best next action."         |
| Find and reuse best practices         | "Find a community workflow that fits this project and adapt it." |
| Turn context into executable work     | "Convert this blog post into a reusable workflow for my team."   |
| Split AI execution and human control  | "Run automatable steps with AI and keep final approval with me." |
| Capture successful patterns for reuse | "This worked well. Save it as a reusable workflow."              |

## Quick Start

If you're working with an AI agent, simply say:

```
Set up MCP and Skills from github.com/epismoai/skills
```

The agent will read this page and complete the steps below. Otherwise, follow the steps manually.

---

Complete these three steps in order. Each step depends on output from the previous one.

### Step 1: Create a Secret Key

> You can also create one from the UI: `Settings > MCP Servers > Create Secret Key`. If you do, skip to Step 2.

**1-1. Request an OTP**

Send your email address. A one-time PIN will be sent to that address, and the response returns an `otpId`.

```bash
curl -sX POST https://api.epismo.ai/v1/otp-tokens \
  -H "Content-Type: application/json" \
  -d '{"email":"you@example.com"}'

# => {"otpId":"<OTP_ID>"}
```

Save: `OTP_ID` from the response.

**1-2. Verify the OTP and get an access token**

Use the `otpId` from step 1-1 and the PIN delivered to your email.

```bash
curl -sX POST https://api.epismo.ai/v1/users \
  -H "Content-Type: application/json" \
  -d '{"otpId":"<OTP_ID>","pin":"<PIN_FROM_EMAIL>"}'

# => {"userId":"<USER_ID>","accessToken":"<ACCESS_TOKEN>"}
```

Save: `ACCESS_TOKEN` from the response. It is short-lived and used only to issue a Secret Key in the next step.

**1-3. Issue a Secret Key**

Use the `ACCESS_TOKEN` from step 1-2. The key will be scoped to a workspace.

- **Personal workspace (default):** omit `workspaceId` from the request body.
- **Specific workspace:** first fetch your workspace IDs, then include the target `workspaceId`.

```bash
# Optional: fetch workspace IDs
curl -sX GET https://api.epismo.ai/v1/workspaces \
  -H "Authorization: Bearer <ACCESS_TOKEN>"

# => [{"workspaceId":"<WORKSPACE_ID>","name":"..."},...]
```

```bash
curl -sX POST https://api.epismo.ai/v1/secret-keys \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <ACCESS_TOKEN>" \
  -d '{"workspaceId":"<WORKSPACE_ID>"}'   # omit workspaceId to use personal workspace

# => {"secretKey":"<SECRET_KEY>"}
```

Save: `SECRET_KEY` from the response. **It is shown only once.** Export it as an environment variable before continuing:

```bash
export EPISMO_SECRET_KEY=<SECRET_KEY>
```

> `EPISMO_SECRET_KEY` authenticates most Epismo API endpoints (`Authorization: Bearer $EPISMO_SECRET_KEY`). Only the endpoints used in this step, require the short-lived `ACCESS_TOKEN`.

---

### Step 2: Add the MCP Server to Your Client

Use `EPISMO_SECRET_KEY` from Step 1 to configure your MCP client.

MCP endpoint:

- URL: `https://mcp.epismo.ai/`
- Protocol: Streamable HTTP (`POST /`, JSON-RPC 2.0)
- Auth header: `Authorization: Bearer <SECRET_KEY>`

Add the following to your client configuration:

```json
{
  "name": "epismo-mcp",
  "url": "https://mcp.epismo.ai/",
  "headers": {
    "Authorization": "Bearer <SECRET_KEY>"
  }
}
```

Verify the connection by calling any MCP tool. If the server responds, the setup is complete.

---

### Step 3: Load Skills into Your Agent

Get the skill files from this repository (clone, download, or copy — use whatever fits your environment) and load them into your agent's context.

Each skill follows this structure:

```
<skill-name>/
  SKILL.md          <- primary instruction file; load this into your agent's context
  references/       <- supplementary references (loaded on demand by SKILL.md)
  templates/        <- structured output templates
```

The setup is complete when your agent can read `SKILL.md` for the skills it needs.

---

## Source of Truth

Repository: `https://github.com/epismoai/skills`
