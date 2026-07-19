---
name: epismo-basics
description: "Shared Epismo operating model: CLI/MCP surface conventions, workspace and project scope, share URL resolution, selective fetch and pack reuse patterns, pack aliases, pack suggestions, credits/payment handling, and common auth or permission errors. Load this alongside any Epismo skill, or trigger on Epismo usage questions, alias/credit issues, share URLs, workspace scope, or setup/auth problems blocking another Epismo task."
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

| Surface | Pattern                                | Example                             |
| ------- | -------------------------------------- | ----------------------------------- |
| `cli`   | `epismo <resource> <action> [--flags]` | `epismo pack search --type context` |
| `mcp`   | `epismo_<resource>_<action>` + JSON    | `epismo_pack_search`                |

MCP tool name = the terminal CLI command name, including `epismo`, with spaces and hyphens replaced by underscores (e.g. `epismo pack get` → `epismo_pack_get`). Conceptual payloads are the same across surfaces; each `--kebab-flag` becomes a camelCase JSON key (`--include-snapshot` → `includeSnapshot`), and a repeatable / comma-separated flag becomes an array (`--status open,applied` → `statuses: ["open", "applied"]`). **Skill docs show CLI forms only** — derive the MCP tool and parameters from these rules rather than re-listing them per page.

**CLI is the full surface; MCP is a subset.** Prefer MCP or CLI and ignore the HTTP API in normal use. Some surfaces are **CLI-only** (no `epismo_*` tool): `login` / `logout` / `whoami`, `workspace`, `project`, `agent`, `credit`, and `token` — use the CLI for these.

Project containers can be listed, created, and updated through the CLI and HTTP API. Project deletion or deactivation is not available on either surface; use the web app for project lifecycle management.

### Pack Aliases

Packs can be fetched by a short alias instead of a full ID. On the CLI, pass the alias (with or without `@` prefix) as the positional `<reference>` to `pack get` — same slot the ID goes in. In MCP, pass it via the `reference` parameter. To create and manage aliases, see [Pack Alias](./references/pack-alias.md).

### Suggestions

Anyone who can read a workflow or context pack can send the owner a text-first improvement suggestion, and the owner reviews and resolves it (`open` → `applied` / `declined` / `archived`). Suggestions never edit the pack directly. The surface is shared across both pack skills — see [Suggestions](./references/suggestions.md) for the lifecycle, listing modes, and CLI/MCP operations.

### CLI Input Conventions

- `--input '<json>'` — inline JSON
- `--input @path/to/file.json` — from file
- `--input -` — from stdin
- Explicit flags override fields in `--input`.

### Workspace Selection

```bash
epismo workspace list
epismo workspace use <workspace-id>                   # save default
epismo workspace current                              # show saved default (no network)
```

Workspace selection is CLI-only. All CLI commands resolve workspace automatically from `EPISMO_TOKEN`, saved default, then personal space. In MCP, workspace scope is implicit in the OAuth token.

### Member Mutation Results

`epismo workspace member upsert/delete` and `epismo project member add/delete` return one result per input position plus an authoritative `memberCount`. Single-target deletes use the same transactional bulk operation. Always inspect every result; command success does not mean every target changed.

- Workspace upsert statuses: `added`, `updated`, `unchanged`, `skipped`, `failed`.
- Project add statuses: `added`, `unchanged`, `skipped`, `failed`.
- Workspace member/project member delete statuses: `deleted`, `skipped`, `failed`.
- Use `code` to explain non-changing outcomes such as `ALREADY_MEMBER`, `DUPLICATE_INPUT`, `USER_NOT_FOUND`, `MEMBER_NOT_FOUND`, `ROLE_NOT_ALLOWED`, or `NOT_WORKSPACE_MEMBER`.
- Treat `inputIndex` as the mapping back to the original user ID list, including duplicate positions.

### Credit Ownership

- Free is personal: included and purchased credits belong to the user.
- Workspace credits are shared by every member; member-level allocation does not exist.
- Each workspace includes 50,000 credits per month and costs $300 per month, independent of member count.
- Only workspace owners can buy workspace credits or change billing. Usage history still attributes consumption to the acting user.
- Additional credits cost $0.01 each, require a minimum purchase of 500 credits, and go directly to the active personal or workspace balance.

---

## Scope Model

- `workspace` — top-level access boundary. All operations run within the active workspace.
- **Search inputs** use additive `scopes`:
  - `scopes: [{ type: "personal" }]` — search items that target the current user directly.
  - `scopes: [{ type: "projects", ids: [...] }]` — search items in specific projects.
  - Combine both scopes to search personal and project items together. Omit `scopes` to use the default search scope.
  - CLI search flags `--personal` and `--projects` build `scopes`.
- **Mutation inputs** (pack/track create, update, apply) use `scope` plus optional `sharedWith`:
  - `scope: { type: "personal" }` — write to the caller's personal space.
  - `scope: { type: "projects", ids: [...] }` — write to specific projects. `ids` must be non-empty and a subset of the caller's accessible projects (see `references.projects`). `projects` scope is only valid in a workspace context.
  - `sharedWith: { userIds?: [...], emails?: [...] }` — optional, grants write access to specific people. Works with both `personal` and `projects` scope.
  - On create, `scope` is required (no implicit personal fallback). On update, omit `scope`/`sharedWith` to preserve existing ACL bits; passing them replaces.
  - Updating an item that has hidden project shares to `scope: { type: "personal" }` errors out.
  - CLI mutation flags: `--personal`, `--projects <id...>`, `--share-with <userIdOrEmail...>` (values containing `@` are treated as emails).

When the user refers to "my project", resolve both layers before writing:

1. Active workspace
2. Target project(s) via `scope: { type: "projects", ids: [...] }`

---

## Pack References (Resolving Share URLs)

Pack-level commands (`get`, `update`, `like`, `delete`) take a **single `reference`** — the server resolves it. You never follow redirects or extract IDs yourself; pass the user's value as-is.

| Reference form | Example                                                                            |
| -------------- | ---------------------------------------------------------------------------------- |
| Artifact ID    | `abc-123-...`                                                                      |
| Alias          | `@myproject`, `@handle/myproject`                                                  |
| Share URL      | `https://epismo.ai/share/{token}` or `https://{workspace}.epismo.ai/share/{token}` |
| Hub URL        | `https://epismo.ai/hub/workflows/{id}`, `https://epismo.ai/hub/contexts/{id}`      |

- **CLI** — positional `<reference>`: `epismo pack get <reference>`.
- **MCP** — `reference` parameter on the `epismo_pack_*` tools.

Create or retrieve a share URL only when it is needed:

```bash
epismo pack get <reference> --share-url
```

On MCP, call `epismo_pack_get` with `shareUrl: true`. A successful response includes `shareUrl`. Pack create/update responses do not generate share links automatically. Requesting a share URL does not make a private pack public; recipients still need access to private packs.

Share URLs resolve server-side by token, independent of host — a workspace-subdomain share URL works without special handling, and no credentials or redirect-following are needed. The resolved pack type (`workflow` / `context`) comes back in the response.

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
epismo pack get <id> --block-id <block-id>

# workflow pack — fetch specific steps
epismo pack get <id> --step-id <step-id-1>,<step-id-2>
```

## Pack Reuse Scan

Before creating any new pack, scan private and public candidates for the same topic so local work and community patterns can be compared directly:

1. `visibility=["private"]` + `query=<topic>` — your own or workspace-scoped packs
2. `visibility=["public"]` + `query=<topic>` — community packs for the same topic
3. `like="liked"` + `query=<topic>` — bookmarked packs as a quality signal or fallback

If a close match is found, prefer `get pack` over creating new.

## Search Ranking Mode

`search pack` and `search track` accept `searchMode`: `keyword` (default) or `semantic`. Keyword matches terms in title and content; semantic adds vector similarity, so it also surfaces paraphrases and cross-language matches. Default to `keyword`; use `semantic` when the user describes intent in their own words rather than exact terms. In CLI, pass `--search-mode keyword|semantic`.

```bash
epismo pack search --type context --query "how onboarding works" --search-mode semantic
epismo track search --type task --query "login is broken" --search-mode semantic
```

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
