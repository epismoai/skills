---
name: workflow-pack
description: "Discover, reuse, and release workflow packs in Epismo. Trigger on: 'find a workflow', 'get <id>', 'read <alias>', 'new workflow', 'create workflow', 'capture pattern', 'adapt a workflow', 'reuse this pattern', 'update workflow', 'organize steps', 'release this as a workflow', 'publish a workflow', 'deprecate workflow', 'suggest a change to a workflow', 'review suggestions', 'resolve a suggestion', or any intent to search community workflows or capture a proven execution pattern for reuse."
---

# Workflow Pack

Discover, adapt, and release reusable workflow packs in Epismo — from finding a community pattern to publishing a proven execution structure.

**Core principle: reuse before creating. Search private and public candidates together, then compare with liked/bookmarked workflows before building anything new.**

> For connection setup, surface conventions, scope model, share URL resolution, and error handling, see [Epismo Basics](../epismo-basics/SKILL.md).
> For task/goal tracking, see [Project Tracking](../project-tracking/SKILL.md).
> For workflow query patterns and loading, see [Search & Discovery](./references/search.md).
> For sending and resolving improvement suggestions, see [Suggestions](../epismo-basics/references/suggestions.md).

## Commands

| Command                 | Natural language triggers                                             | →                     |
| ----------------------- | --------------------------------------------------------------------- | --------------------- |
| `new`                   | create workflow, capture pattern, adapt a pattern, new workflow       | [NEW](#new)           |
| `get <id\|alias>`       | get this ID, load this ID, read alias, open alias, load workflow      | [GET](#get)           |
| `find <query>`          | find a workflow, search workflows, what workflows do I have           | [FIND](#find)         |
| `update [<id\|alias>]`  | edit steps, update workflow, modify steps                             | [UPDATE](#update)     |
| `organize`              | reorganize steps, reorder, clean up workflow                          | [ORGANIZE](#organize) |
| `release [<id\|alias>]` | publish this, make public, release pattern, update release, deprecate | [RELEASE](#release)   |
| `suggest`               | suggest a change, propose an edit, review suggestions, resolve a suggestion | [SUGGEST](#suggest) |
| —                       | unclear intent                                                        | Ask once              |

## Operations

| Operation      | CLI                                                                             |
| -------------- | ------------------------------------------------------------------------------- |
| `search pack`  | `epismo pack search --type workflow --filter '{...}'`                           |
| `get pack`     | `epismo pack get <reference> [--full] [--step-id <ids>]`                        |
| `create pack`  | `epismo pack create --input '<json>'`                                           |
| `update pack`  | `epismo pack update <reference> --input '<json>'`                               |
| `delete pack`  | `epismo pack delete <reference>`                                                |
| `like pack`    | `epismo pack like <reference> --liked`                                          |
| `upsert alias` | `epismo alias upsert @<name> --id <id>`                                         |
| `get alias`    | `epismo alias get @<name>`                                                      |
| `list aliases` | `epismo alias list --type workflow`                                             |
| `delete alias` | `epismo alias delete @<name>`                                                   |
| `create suggestion`  | `epismo suggestion create <reference> --title <t> --content <c>`          |
| `get suggestion`     | `epismo suggestion get <id> [--include-snapshot]`                         |
| `list suggestions`   | `epismo suggestion list [--owner] [--reference <ref>] [--status <s>]`     |
| `update suggestion`  | `epismo suggestion update <id> --title <t> --content <c>`                 |
| `resolve suggestion` | `epismo suggestion resolve <id> --status <applied\|declined\|archived>`   |

CLI forms are terminal commands (`epismo ...`). On MCP, derive the tool name mechanically from the full command name (`epismo pack get` → `epismo_pack_get`). See [surface conventions](../epismo-basics/SKILL.md#surface-conventions).

`<reference>` is any pack reference — an ID, `@alias`, share URL, or hub URL — resolved server-side. See [Pack References](../epismo-basics/SKILL.md#pack-references-resolving-share-urls).

---

## NEW

**Goal:** create a reusable workflow as a private pack. Reuse before building — scan for an existing pattern first.

### Step 1 — Reuse scan

Infer 2–4 topic keywords from the user's intent. Search private and public scopes together so local patterns and community patterns can be compared before choosing a base.

```bash
# Private workflows in the active workspace
epismo pack search --type workflow --query "<topic>" --filter '{"visibility":["private"]}'

# Public community workflows
epismo pack search --type workflow --query "<topic>" --filter '{"visibility":["public"]}'

# Bookmarked workflows, useful as a quality signal or fallback
epismo pack search --type workflow --query "<topic>" --filter '{"like":"liked"}'
```

| Result        | Action                                                                     |
| ------------- | -------------------------------------------------------------------------- |
| Strong match  | Fetch full content → adapt steps → [Step 3 — Create](#step-3--create)      |
| Partial match | Fetch → use as base, modify steps → [Step 3 — Create](#step-3--create)     |
| No match      | Design steps from scratch → [Step 2 — Design steps](#step-2--design-steps) |

### Step 2 — Design steps

Each step should:

- Have a clear title and self-contained content
- Assign an owner: `human` for decisions/reviews, AI agent name for generation/research
- Declare `dependsOn` only when execution genuinely requires a prior step

### Adapting a found workflow

Before materializing:

1. Identify steps to keep, modify, or remove.
2. Strip project-specific names, IDs, and one-off constraints.
3. Reassign owners as appropriate for the new context.
4. Confirm destination: private pack for future reuse, or project tracks for immediate execution.

**To materialize a workflow pack as project tracks**, use `pack run` — it converts the pack in one call (CLI `epismo pack run`, MCP `epismo_pack_run`). It creates a root goal (the run's objective and retrieval anchor) plus one `todo` task per step, resolves `dependsOn`/`parentId` step ids to track UUIDs, converts relative due dates to absolute dates from `--start-date` (default today), and stamps `workflow:<id>` provenance sources:

```bash
epismo pack run @release-review \
  --title "Ship CSV export" \
  --projects pj_123 \
  --assignee human=<user-id> \
  --context @repo-conventions
```

- `--title` / `--content` set the goal's objective; omitted, they default to the pack's title/content.
- `--assignee token=id` maps pack assignee tokens; map `human` to a user id (agent ids resolve as-is). Unmapped `human` steps are left unassigned and reported in `warnings` — always check them.
- The response returns `goal.id` and a `stepMap` (step id → track UUID). Fetch the whole run later with `epismo track search --type task --filter '{"goalId":["<goal-id>"]}'`.

Use `epismo track apply` directly only when building a task tree from scratch without a pack.

Use [Workflow Patterns — Discovery](./templates/patterns.md#2-workflow-discovery) for structured adaptation reports.

### Step 3 — Create

Default `private`. Use [RELEASE](#release) to publish publicly.

`create pack`:

```json
{
  "title": "<title>",
  "content": "<description>",
  "type": "workflow",
  "visibility": "private",
  "steps": [
    {
      "id": "s001",
      "title": "<step title>",
      "content": "<step instructions>",
      "dueDate": "",
      "dependsOn": [],
      "parentId": "",
      "assignee": "human"
    }
  ]
}
```

| Who needs it    | `visibility` | Scope / share                                    |
| --------------- | ------------ | ------------------------------------------------ |
| Just me         | `"private"`  | `scope: { type: "personal" }`                    |
| My team         | `"private"`  | `scope: { type: "projects", ids: ["pj_xxx"] }`   |
| Specific people | `"private"`  | any `scope` + `sharedWith: { userIds / emails }` |

```
workflow  <workflow-id>  <workflow-title>
step      <step-id>      <step-title>
```

---

## GET

**Goal:** fetch a workflow by ID, alias, or explicit read/open target.

### Route intent before resolving

Prefer **alias-first** unless the input is obviously a search query.

| Input pattern                                                          | Route         | Why                                           |
| ---------------------------------------------------------------------- | ------------- | --------------------------------------------- |
| `get ...`, `use ...`, `load ...`, `open ...`, `read ...`               | [GET](#get)   | Explicit retrieval                            |
| `find ...`, `search ...`, `what workflows do I have`, `show workflows` | [FIND](#find) | Explicit discovery                            |
| Bare UUID                                                              | [GET](#get)   | IDs are unambiguous                           |
| Bare alias-shaped token                                                | [GET](#get)   | Try alias first, then search if it misses     |
| Short ambiguous phrase                                                 | [GET](#get)   | Try alias first, then search if it misses     |
| Obvious search query, question, or long descriptive phrase             | [FIND](#find) | Discovery intent is clearer than alias intent |

### Resolve the input

1. UUID → `get pack <id>`.
2. `@<alias>`, `@<handle>/<alias>`, or one compact token like `prd-review` → `get pack @<alias>`; if it misses, run [FIND](#find) with the same text.
3. Short phrase like `deployment checklist` → try `get pack @<alias>` first; if it misses, run [FIND](#find).
4. Question, explicit search wording, or long descriptive text → [FIND](#find) first.
5. `get/read/open/use <target>` always counts as retrieval intent.

`get pack` — default returns outline only; pass `--full` for all steps, or `--step-id` to load specific steps.

```bash
epismo pack get <pack-id> --full
epismo pack get @<alias> --full
epismo pack get <pack-id> --step-id <step-id-1>,<step-id-2>
```

---

## FIND

**Goal:** discover the right workflow when the ID is not known.

`search pack` — scan titles only (no step content). Use this for natural-language discovery requests and multi-keyword topic phrases. Search private and public scopes for the same topic, then compare results before fetching full content:

1. `type: workflow`, `query: <topic>`, `filter: { visibility: ["private"] }`
2. `type: workflow`, `query: <topic>`, `filter: { visibility: ["public"] }`
3. `type: workflow`, `query: <topic>`, `filter: { like: "liked" }`

Present the title list; if the match is clear, `get pack` immediately.

For filter keys and search recipes, see [Search & Discovery](./references/search.md).

---

## UPDATE

**Goal:** edit the steps or metadata of an existing workflow.

### ID or alias known

Always fetch before writing.

`get pack` → inspect current steps and metadata → `update pack` with `id` and step operations using `op` fields.

For `update <alias> based on this conversation`:

- modify the steps where the new guidance belongs
- add new steps only for new execution work
- keep `dependsOn`, ownership, and ordering coherent

### ID unknown

1. `search pack` — `type: workflow`, `query: <inferred topic>`, run both `filter: { visibility: ["private"] }` and `filter: { visibility: ["public"] }`
2. **One clear match** → fetch and update without asking.
3. **Multiple candidates** → show titles and IDs, ask the user to pick.
4. **No match** → fall back to [NEW](#new).

**"Last workflow" shortcut:** search private workflow packs and take the first result from the default recent-first outline.

```
workflow  <workflow-id>  <workflow-title>
step      <step-id>      <step-title>    # when a specific step was updated
```

---

## ORGANIZE

**Goal:** improve the step structure of an existing workflow.

`get pack --full` → review all steps → apply changes → `update pack`.

| Situation                                       | Action                                                              |
| ----------------------------------------------- | ------------------------------------------------------------------- |
| A step is too broad or covers multiple concerns | **Split** — divide into two focused steps                           |
| Two steps overlap or one has become redundant   | **Merge** — combine into one coherent step, remove the other        |
| A step is no longer relevant                    | **Remove** — delete the step from the workflow                      |
| Steps are out of execution order                | **Reorder** — update `dependsOn` chains to reflect correct sequence |
| The whole workflow is obsolete                  | **Delete** — requires explicit user approval                        |

All changes are applied via `update pack` using `op` fields. Use `"op": "add"` for new steps, `"op": "update"` for existing ones, `"op": "remove"` to delete a step by ID. `delete pack` requires explicit user approval.

---

## RELEASE

**Goal:** publish a workflow publicly, update an existing release, or deprecate an obsolete one.

Two paths:

| Situation                               | Path                                             |
| --------------------------------------- | ------------------------------------------------ |
| Existing private workflow worth sharing | Promote — change visibility to `public`          |
| New content written for community reuse | Create new public workflow via [NEW](#new) first |

### Step 1 — Quality Gate

Run [Quality Gate](./references/quality.md) before any release decision. All 8 criteria must pass for public release. A single fail means: fix or keep private.

### Step 2 — Release decision

| Decision       | When                                                                     |
| -------------- | ------------------------------------------------------------------------ |
| `release`      | Execution proven, structure generalizable, no superior equivalent exists |
| `update`       | Improves an existing released pattern without breaking consumers         |
| `keep private` | Evidence incomplete or quality gate has open items                       |
| `deprecate`    | Obsolete, unsafe, or superseded                                          |

Run the pre-decision checks in [Release](./references/release.md) before choosing one of these paths.

### Step 3 — Confirm and update

**Confirm with user before writing:** state the workflow title, ID, and target visibility.

`update pack`:

```json
{
  "id": "<id>",
  "visibility": "public",
  "category": "<category>"
}
```

| Category       | When to use                                             |
| -------------- | ------------------------------------------------------- |
| `productivity` | Daily rhythms, time management, operating norms         |
| `programming`  | Code review, deployment, testing, development workflows |
| `design`       | Component reviews, UX research, design system processes |
| `marketing`    | Campaign planning, content creation, launch workflows   |
| `operations`   | Incident response, onboarding, deployment pipelines     |
| `learning`     | Onboarding guides, skill-building, training workflows   |
| `life`         | Personal productivity, non-work routines                |

See [Release](./references/release.md) for the approval boundary and required output format.

```
workflow  <workflow-id>  <workflow-title>
share     https://epismo.ai/hub/workflows/<id>
```

---

## SUGGEST

**Goal:** propose an improvement to a workflow you don't own, or review and resolve suggestions on a workflow you do own. Suggestions are text-first — they never edit the workflow directly.

Full lifecycle, listing modes, and the CLI/MCP surface are in [Suggestions](../epismo-basics/references/suggestions.md). Decide direction first:

| Intent                                              | Action                                                                 |
| --------------------------------------------------- | ---------------------------------------------------------------------- |
| Propose a change to someone else's workflow         | `create suggestion` with the pack reference, a short title, and content describing the proposed edit |
| See suggestions I've received (inbox)               | `list suggestions --owner` (filter with `--status open`)               |
| See suggestions for one workflow                    | `list suggestions --reference @<alias>`                                |
| See suggestions I submitted                         | `list suggestions` (no flags)                                          |
| Revise my own suggestion                            | `update suggestion <id>` (author only)                                 |
| Act on a suggestion I received                      | `resolve suggestion <id> --status applied\|declined\|archived` (owner only) |

When you **apply** a suggestion, make the actual change via [UPDATE](#update) or [ORGANIZE](#organize) first, then `resolve --status applied`. Resolving alone does not edit the workflow.

```
suggestion  <suggestion-id>  <suggestion-title>  <status>
```

---

## Write Safety

1. **Default private** — `new` and `update` always write private. Use [RELEASE](#release) to go public.
2. **No silent writes** — if scope is unclear, ask once before writing.
3. **Approval required** — public publication, deprecation, and overwriting another owner's workflow require explicit user confirmation before writing.

For the full approval matrix, see [Visibility & Sharing — Approval Boundary](./references/visibility.md#approval-boundary).

---

## Operation Output

After every operation, return in this order:

1. **Workflow and step** — return both ID and title at each level affected
2. **What changed** — one sentence
3. **Next action** — single most useful next step (omit if none)

```
workflow  <workflow-id>  <workflow-title>
step      <step-id>      <step-title>    # when a specific step was created or updated
```

Skip analysis and evidence unless the user asked for it.

## Source of Truth

Repository: `https://github.com/epismoai/skills`
