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

Complete these three steps in order.

### Step 1: Get a Session Token

```bash
curl -sX POST "https://api.epismo.ai/v1/otp-tokens" \
  -H "Content-Type: application/json" \
  -d '{"email":"you@example.com"}'

# => {"otpId":"<OTP_ID>"}
```

```bash
curl -sX POST "https://api.epismo.ai/v1/users" \
  -H "Content-Type: application/json" \
  -d '{"otpId":"<OTP_ID>","pin":"<PIN_FROM_EMAIL>"}'

# => { "sessionToken":"<SESSION_TOKEN>", ... }
```

### Step 2: Issue an MCP Token

If you want a workspace-scoped MCP token, list workspaces and pick one `id`. Otherwise omit `workspaceId`.

```bash
curl -sX GET "https://api.epismo.ai/v1/workspaces" \
  -H "Authorization: Bearer <SESSION_TOKEN>"
```

```bash
curl -sX POST "https://api.epismo.ai/v1/mcp/tokens" \
  -H "Authorization: Bearer <SESSION_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{
    "resource":"https://mcp.epismo.ai/",
    "workspaceId":"<WORKSPACE_ID>"
  }'

# => {
#      "accessToken":"<ACCESS_TOKEN>",
#      "refreshToken":"<REFRESH_TOKEN>",
#      "tokenType":"Bearer",
#      "expiresIn":3600,
#      "scope":"mcp"
#    }
```

To target the personal scope, remove the `workspaceId` line.

Refresh with the returned `refreshToken`:

```bash
curl -sX POST "https://api.epismo.ai/v1/mcp/tokens/refresh" \
  -H "Content-Type: application/json" \
  -d '{
    "refreshToken":"<REFRESH_TOKEN>",
    "resource":"https://mcp.epismo.ai/"
  }'
```

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
