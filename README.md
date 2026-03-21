# Epismo Skills

Reusable skill packages that give AI agents structured project-operation capabilities through Epismo MCP or CLI.

## Why Skills

AI agents hit the same problems across teams and projects:

- Operational know-how stays trapped in chat histories.
- Prompts copy easily, but multi-step processes don't.
- Every new project restarts from scratch.

Skills solve this by packaging proven operational patterns — intake, planning, coordination, risk handling, asset capture, and workflow release — into portable instruction sets that any agent with Epismo access can follow.

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
Set up Epismo access and Skills from github.com/epismoai/skills. I can use MCP or CLI.
```

The agent will read this page and complete the steps below. Otherwise, follow the steps manually.

Choose one access method:

- `MCP` if your agent connects to Epismo through an MCP server.
- `CLI` if your agent or automation will call the `epismo` command directly.

### Option A: Set Up MCP Access

MCP clients that support OAuth metadata discovery (most standard MCP clients) connect automatically using the standard OAuth authorization flow — no manual token setup required.

For manual or scripted setups, follow the two-step process below: first obtain an **API access token**, then exchange it for an **MCP token**.

#### Step 1: Get an API Access Token

The API access token authenticates you against Epismo's `/v1/*` endpoints. Obtain one using the `otp` grant:

```bash
# Request an OTP
curl -sX POST "https://api.epismo.ai/v1/otp-tokens" \
  -H "Content-Type: application/json" \
  -d '{"email":"you@example.com"}'
# => {"otpId":"<OTP_ID>"}
```

```bash
# Exchange OTP for an API access token
curl -sX POST "https://api.epismo.ai/oauth/token" \
  -H "Content-Type: application/json" \
  -d '{
    "grant_type":"otp",
    "otp_id":"<OTP_ID>",
    "pin":"<PIN>",
    "client_id":"epismo-cli"
  }'
# => {"access_token":"<API_ACCESS_TOKEN>","refresh_token":"<REFRESH_TOKEN>","token_type":"Bearer","expires_in":3600,"scope":"read write offline_access"}
```

> **API access token** (`scope: read write offline_access`) — used to call `/v1/*` REST endpoints, including workspace listing and MCP token issuance. Not directly accepted by the MCP server.

If you need the authenticated user, call `/oauth/userinfo` with the returned token.

#### Step 2: Exchange for an MCP Token

`POST /v1/mcp/tokens` acts as a bridge: it accepts your **API access token** and issues a separate **MCP token** scoped to the MCP server.

If you want a workspace-scoped MCP token, list workspaces and pick one `id`. Otherwise omit `workspaceId`.

```bash
curl -sX GET "https://api.epismo.ai/v1/workspaces" \
  -H "Authorization: Bearer <API_ACCESS_TOKEN>"
```

```bash
curl -sX POST "https://api.epismo.ai/v1/mcp/tokens" \
  -H "Authorization: Bearer <API_ACCESS_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{
    "resource":"https://mcp.epismo.ai/",
    "workspaceId":"<WORKSPACE_ID>"
  }'

# => {
#      "accessToken":"<MCP_ACCESS_TOKEN>",
#      "refreshToken":"<MCP_REFRESH_TOKEN>",
#      "tokenType":"Bearer",
#      "expiresIn":3600,
#      "scope":"mcp"
#    }
```

> **MCP token** (`scope: mcp`) — used exclusively with the MCP server at `https://mcp.epismo.ai/`. Distinct from the API access token above.

To target the personal scope, remove the `workspaceId` line.

Refresh with the returned `refreshToken`:

```bash
curl -sX POST "https://api.epismo.ai/v1/mcp/tokens/refresh" \
  -H "Content-Type: application/json" \
  -d '{
    "resource":"https://mcp.epismo.ai/",
    "refreshToken":"<MCP_REFRESH_TOKEN>"
  }'
```

### Option B: Set Up CLI Access

#### Install

```bash
npm install -g epismo
```

#### Auth

`epismo login` uses the same OTP-to-OAuth flow described above and defaults to the no-browser flow.

```bash
epismo login --email you@example.com
epismo whoami
```

`epismo login --browser` is also available if needed.

Interactive login and manual API usage now follow the same OAuth flow end to end. The CLI uses saved OAuth credentials for protected API access, while direct Bearer token usage applies to manual API calls and MCP token setup shown above.

#### Workspace Selection

```bash
epismo workspace list
epismo workspace use ws_123
```

- `epismo workspace use <workspace-id>` saves the default workspace locally.
- `--workspace-id <id>` overrides the saved default for a single command.
- If neither is set, the CLI uses personal context.

#### Workspace and Project Scope

- `workspace` is the top-level execution scope.
- Projects live inside a workspace.
- `projects[]` narrows search scope and controls project membership inside that workspace.

#### Input Conventions

- `--input <json>` accepts an inline JSON object.
- `--input @path/to/file.json` loads a JSON object from a file.
- `--input -` reads a JSON object from stdin.
- Explicit flags override fields loaded from `--input`.
- Complex fields such as `asset`, `workflow`, `track`, and `filter` also accept JSON strings when passed as direct flags.

### Load Skills into Your Agent

Get the skill files from this repository (clone, download, or copy — use whatever fits your environment) and load them into your agent's context.

Each skill follows this structure:

```
<skill-name>/
  SKILL.md          <- primary instruction file; load this into your agent's context
  references/       <- supplementary references (loaded on demand by SKILL.md)
  templates/        <- structured output templates
```

The setup is complete when your agent can read `SKILL.md` for the skills it needs.

## Source of Truth

Repository: `https://github.com/epismoai/skills`
