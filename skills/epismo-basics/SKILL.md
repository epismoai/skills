---
name: epismo-basics
description: "Shared Epismo operating model: CLI/MCP surface conventions, workspace and project scope, share URL resolution, selective fetch and asset reuse patterns, asset aliases, credits/payment handling, and common auth or permission errors. Load this alongside any Epismo skill, or trigger on Epismo usage questions, alias/credit issues, share URLs, workspace scope, or setup/auth problems blocking another Epismo task."
---

# Epismo Basics

Shared reference for all Epismo skills. Covers connection, surface conventions, scope, share URL resolution, and error handling.

Skills that build on this:
- [Project Tracking](../project-tracking/SKILL.md) — tasks, goals, notes, planning
- [Workflow Hub](../workflow-hub/SKILL.md) — workflow asset discovery and release
- [Context Pack](../context-pack/SKILL.md) — context assets, session handoff

---

## Connection

Choose one surface and stick with it within a session.

- **MCP** — for tool-based agents. Requires an OAuth access token with `scope=mcp` and the correct `resource`. Standard MCP clients connect automatically via OAuth metadata discovery; no manual token setup required.
- **CLI** — for shell-driven agents. Use `epismo login` or set `EPISMO_TOKEN`.

If access is not ready, complete auth setup first — see the `github.com/epismoai/skills` README.

### CLI Auth

```bash
epismo login --email you@example.com   # OTP flow (default, no browser)
epismo login --browser                  # browser-based flow
epismo whoami                           # verify
```

### MCP Token (manual / scripted setup)

```bash
# 1. Request OTP
curl -sX POST https://api.epismo.ai/v1/otp-tokens \
  -H "Content-Type: application/json" \
  -d '{"email":"you@example.com"}'
# => {"otpId":"<OTP_ID>"}

# 2. Exchange OTP for API access token
curl -sX POST https://api.epismo.ai/oauth/token \
  -H "Content-Type: application/json" \
  -d '{"grant_type":"otp","otp_id":"<OTP_ID>","pin":"<PIN>","client_id":"epismo-cli"}'
# => {"access_token":"<API_TOKEN>", ...}

# 3. List workspaces and pick one
curl -sX GET https://api.epismo.ai/v1/workspaces \
  -H "Authorization: Bearer <API_TOKEN>"

# 4. Exchange for MCP token
curl -sX POST https://api.epismo.ai/v1/mcp/tokens \
  -H "Authorization: Bearer <API_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{"resource":"https://mcp.epismo.ai/","workspaceId":"<workspace-id>"}'
# => {"accessToken":"<MCP_TOKEN>","scope":"mcp", ...}

# Refresh
curl -sX POST https://api.epismo.ai/v1/mcp/tokens/refresh \
  -H "Content-Type: application/json" \
  -d '{"resource":"https://mcp.epismo.ai/","refreshToken":"<MCP_REFRESH_TOKEN>"}'
```

---

## Surface Conventions

| Surface | Pattern | Example |
| ------- | ------- | ------- |
| `cli` | `epismo <resource> <action> [--flags]` | `epismo asset search --type context` |
| `mcp` | `epismo_<resource>_<action>` + same parameters | `epismo_asset_search` |

MCP tool name = CLI command with spaces and hyphens replaced by underscores. Parameters are identical across both surfaces.

### Asset Aliases

Assets can be fetched by a short alias instead of a full ID. Pass `--alias <name>` (or `alias` parameter in MCP) wherever `--id` is accepted on `get asset`. To create and manage aliases, see [Asset Alias](./references/asset-alias.md).

### CLI Input Conventions

- `--input '<json>'` — inline JSON
- `--input @path/to/file.json` — from file
- `--input -` — from stdin
- Explicit flags override fields in `--input`.

### Workspace Selection

```bash
epismo workspace list
epismo workspace use --workspace-id <workspace-id>   # save default
epismo workspace current                              # show saved default (no network)
```

Workspace selection is CLI-only. In MCP, workspace scope is implicit in the OAuth token.

---

## Scope Model

- `workspace` — top-level access boundary. All operations run within the active workspace.
- `projects[]` — narrows scope to specific projects within the active workspace.
- Omitting `projects[]` on a search means "all accessible projects in the active workspace".

When the user refers to "my project", resolve both layers before writing:
1. Active workspace
2. Target project(s) via `projects[]`

---

## Resolving Share URLs

`epismo.ai/share/{token}` resolves to a resource without credentials by following the HTTP redirect.

```bash
# curl
curl -s -o /dev/null -w "%{redirect_url}" "https://epismo.ai/share/${TOKEN}"
# → https://epismo.ai/hub/workflows/{id}
# → https://epismo.ai/hub/contexts/{id}
```

```typescript
// fetch (Node.js)
const res = await fetch(`https://epismo.ai/share/${token}`, { redirect: "manual" });
const location = res.headers.get("location") ?? "";

const workflowMatch = location.match(/\/hub\/workflows\/([^/?#]+)/);
const contextMatch  = location.match(/\/hub\/contexts\/([^/?#]+)/);
// use whichever matches; id = decodeURIComponent(match[1])
```

| Redirect path | Resource type |
| ------------- | ------------- |
| `/hub/workflows/{id}` | `workflow` asset |
| `/hub/contexts/{id}` | `context` asset |

Use the resolved `id` with `get asset` on any surface.

---

## Selective Fetch Pattern

`search` always returns outline format (`id`, `title`, and brief metadata) — never full content blocks. Always scan titles first, then fetch only what you need — this keeps session context lean.

1. **Scan** — `search asset` or `search track` to get a title list.
2. **Select** — identify relevant items from the titles. Skip clearly unrelated ones.
3. **Fetch** — `get asset --full` or `get track` for each selected item to load full content.

`get asset` has two modes:

| Mode | Flag | Returns |
| ---- | ---- | ------- |
| Outline (default) | _(none)_ | `id`, `title`, `view`, `contentIndex` — no content blocks |
| Full | `--full` | complete content including all blocks / steps |

To load a single block or step instead of the full asset:

```bash
# context asset — fetch one block
epismo asset get --id <id> --block-id <block-id>

# workflow asset — fetch specific steps
epismo asset get --id <id> --step-id <step-id-1>,<step-id-2>
```

## Asset Reuse Scan Order

Before creating any new asset, scan in this order to avoid duplicating something that already exists:

1. `visibility=["private"]` + `query=<topic>` — your own assets first
2. `like="liked"` + `query=<topic>` — assets you bookmarked
3. `visibility=["public"]` + `query=<topic>` — community assets

If a close match is found, prefer `get asset` over creating new.

## Pagination

All `search` calls return page size 20. For large result sets, iterate `page=1, 2, 3...` and merge results.

```bash
epismo asset search --type context --filter '{"visibility":["public"]}' --page 2
epismo track search --type task --filter '{"status":["todo"]}' --page 2
```

Stop iterating when a page returns fewer than 20 results.

---

## Error Handling

| Error | Action |
| ----- | ------ |
| `Payment Required: Insufficient credits` | Stop. Check balance and purchase credits — see [Credit Purchase](./references/credit-purchase.md). |
| `Permission denied` | Re-check accessible projects and resource ownership. |
| `Unauthorized` / `403` | Verify MCP token or `EPISMO_TOKEN` (CLI), active workspace, and subscription. |
| `Not Found` / `404` | Confirm the resource ID. It may have been deleted or the share token may have expired. |
| Rate limit / `429` | Wait and retry with backoff. Inform user if persistent. |
| Timeout | Retry once. If persistent, reduce payload size or split the operation. |

## Source of Truth

Repository: `https://github.com/epismoai/skills`
