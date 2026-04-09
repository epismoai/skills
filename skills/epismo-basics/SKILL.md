---
name: epismo-basics
description: "Shared Epismo operating model: CLI/MCP surface conventions, workspace and project scope, share URL resolution, selective fetch and pack reuse patterns, pack aliases, credits/payment handling, and common auth or permission errors. Load this alongside any Epismo skill, or trigger on Epismo usage questions, alias/credit issues, share URLs, workspace scope, or setup/auth problems blocking another Epismo task."
---

# Epismo Basics

Shared reference for all Epismo skills. Covers connection, surface conventions, scope, share URL resolution, and error handling.

Skills that build on this:

- [Project Tracking](../project-tracking/SKILL.md) — tasks, goals, notes, planning
- [Workflow Pack](../workflow-pack/SKILL.md) — workflow pack discovery and release
- [Context Pack](../context-pack/SKILL.md) — context packs, session handoff

---

## Connection

CLI and MCP are two interfaces to the **same Epismo service** — same account, same data, same tools. There is one Epismo account; the difference is only how you connect.

**When both are available, use CLI.** If only MCP is available, use MCP. Never use both in the same session.

### Auth

**CLI** (preferred):

```bash
epismo login --email you@example.com   # OTP flow (default, no browser)
epismo login --browser                  # browser-based flow
epismo whoami                           # verify
```

**MCP**: add `https://mcp.epismo.ai` as an MCP server in your client. Authentication is handled automatically via OAuth.

---

## Surface Conventions

| Surface | Pattern                                        | Example                             |
| ------- | ---------------------------------------------- | ----------------------------------- |
| `cli`   | `epismo <resource> <action> [--flags]`         | `epismo pack search --type context` |
| `mcp`   | `epismo_<resource>_<action>` + same parameters | `epismo_pack_search`                |

MCP tool name = CLI command with spaces and hyphens replaced by underscores. Parameters are identical across both surfaces.

### Pack Aliases

Packs can be fetched by a short alias instead of a full ID. Pass `--alias <name>` (or `alias` parameter in MCP) wherever `--id` is accepted on `get pack`. To create and manage aliases, see [Pack Alias](./references/pack-alias.md).

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
const res = await fetch(`https://epismo.ai/share/${token}`, {
  redirect: "manual",
});
const location = res.headers.get("location") ?? "";

const workflowMatch = location.match(/\/hub\/workflows\/([^/?#]+)/);
const contextMatch = location.match(/\/hub\/contexts\/([^/?#]+)/);
// use whichever matches; id = decodeURIComponent(match[1])
```

| Redirect path         | Resource type   |
| --------------------- | --------------- |
| `/hub/workflows/{id}` | `workflow` pack |
| `/hub/contexts/{id}`  | `context` pack  |

Use the resolved `id` with `get pack` on any surface.

---

## Selective Fetch Pattern

`search` always returns outline format (`id`, `title`, and brief metadata) — never full content blocks. Always scan titles first, then fetch only what you need — this keeps session context lean.

1. **Scan** — `search pack` or `search track` to get a title list.
2. **Select** — identify relevant items from the titles. Skip clearly unrelated ones.
3. **Fetch** — `get pack --full` or `get track` for each selected item to load full content.

`get pack` has two modes:

| Mode              | Flag     | Returns                                                   |
| ----------------- | -------- | --------------------------------------------------------- |
| Outline (default) | _(none)_ | `id`, `title`, `view`, `contentIndex` — no content blocks |
| Full              | `--full` | complete content including all blocks / steps             |

To load a single block or step instead of the full pack:

```bash
# context pack — fetch one block
epismo pack get --id <id> --block-id <block-id>

# workflow pack — fetch specific steps
epismo pack get --id <id> --step-id <step-id-1>,<step-id-2>
```

## Pack Reuse Scan Order

Before creating any new pack, scan in this order to avoid duplicating something that already exists:

1. `visibility=["private"]` + `query=<topic>` — your own packs first
2. `like="liked"` + `query=<topic>` — packs you bookmarked
3. `visibility=["public"]` + `query=<topic>` — community packs

If a close match is found, prefer `get pack` over creating new.

## Pagination

All `search` calls return page size 20. For large result sets, iterate `page=1, 2, 3...` and merge results.

```bash
epismo pack search --type context --filter '{"visibility":["public"]}' --page 2
epismo track search --type task --filter '{"status":["todo"]}' --page 2
```

Stop iterating when a page returns fewer than 20 results.

---

## Error Handling

| Error                                    | Action                                                                                                          |
| ---------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| `Payment Required: Insufficient credits` | Stop. Check balance and purchase credits — see [Credit Purchase](./references/credit-purchase.md).              |
| `Permission denied`                      | Re-check accessible projects and resource ownership.                                                            |
| `Unauthorized` / `403`                   | Re-authenticate: run `epismo login` (CLI) or reconnect via MCP OAuth. Verify active workspace and subscription. |
| `Not Found` / `404`                      | Confirm the resource ID. It may have been deleted or the share token may have expired.                          |
| Rate limit / `429`                       | Wait and retry with backoff. Inform user if persistent.                                                         |
| Timeout                                  | Retry once. If persistent, reduce payload size or split the operation.                                          |

## Source of Truth

Repository: `https://github.com/epismoai/skills`
