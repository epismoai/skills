---
name: context-pack
description: "Pack, share, and load context using Epismo context packs. Trigger on: 'pack this', 'new pack', 'get <id>', 'read <alias>', 'load my context', 'what context do I have', 'restore session', 'save this context', 'share with my team', 'pack this up', 'hand this off', 'publish this guide', 'organize my packs', or any intent to persist or retrieve knowledge across tools or sessions."
---

# Context Pack

Store and retrieve context through Epismo context packs — so knowledge survives tool switches, session ends, and team handoffs.

The unit of organization is a **block** inside a pack. The goal is one well-structured pack per topic, grown over time by adding and refining blocks — not many small packs.

> For Epismo connection setup, CLI/MCP surface conventions, scope model, share URL resolution, and error handling, see [Epismo Basics](../epismo-basics/SKILL.md).

> **Surface selection:** CLI and MCP connect to the same Epismo service. Use CLI if available; fall back to MCP if not. Never use both in the same session.

## Commands

| Command                | Natural language triggers                                             | →                     |
| ---------------------- | --------------------------------------------------------------------- | --------------------- |
| `pack`                 | pack this, save this, summarize session                               | [PACK](#pack)         |
| `new`                  | new pack, start fresh, hand off a task, share project status          | [NEW](#new)           |
| `publish [<id>]`       | publish this, make this public, publish a guide, share with community | [PUBLISH](#publish)   |
| `get <id\|alias>`      | get this ID, load this ID, restore from ID, read alias, open alias    | [GET](#get)           |
| `find <query>`         | what context do I have, load my context, search                       | [FIND](#find)         |
| `update [<id\|alias>]` | edit this pack, update my last pack                                   | [UPDATE](#update)     |
| `organize`             | reorganize blocks, split this, clean up                               | [ORGANIZE](#organize) |
| —                      | unclear intent                                                        | Ask once              |

## Operations (context packs)

| Operation      | CLI                                                                                         | MCP                  |
| -------------- | ------------------------------------------------------------------------------------------- | -------------------- |
| `search pack`  | `epismo pack search --type context [--query <keywords>] [--filter '{...}']`                 | `epismo_pack_search` |
| `get pack`     | `epismo pack get --id <id> [--full] [--block-id <id>]`<br>`epismo pack get --alias <alias>` | `epismo_pack_get`    |
| `upsert pack`  | `epismo pack upsert --input '<json>'`                                                       | `epismo_pack_upsert` |
| `delete pack`  | `epismo pack delete --id <id>`                                                              | `epismo_pack_delete` |
| `like pack`    | `epismo pack like --id <id> --liked`                                                        | `epismo_pack_like`   |
| `upsert alias` | `epismo alias upsert --type context --id <id> --alias <name>`                               | —                    |
| `get alias`    | `epismo alias get --alias <name>`                                                           | —                    |
| `list aliases` | `epismo alias list --type context`                                                          | —                    |
| `delete alias` | `epismo alias delete --alias <name>`                                                        | —                    |

---

## PACK

**Goal:** save current context as a block inside the right pack. Find an existing pack and add or refine a block; create a new pack only when nothing relevant exists.

### Step 1 — Find the right pack

Infer 2–4 topic keywords from the current session. Search private packs (titles only — one fast call).

`search pack` — `type: context`, `query: <inferred topic>`, `filter: { visibility: ["private"] }`, `sort: updatedAt:desc`

| Result              | Action                                    |
| ------------------- | ----------------------------------------- |
| One clear match     | Fetch it → add or update a block → upsert |
| Multiple candidates | Show titles and IDs → ask user to pick    |
| No match            | Proceed as [NEW](#new)                    |

### Step 2 — Identify the right block

`get pack` — fetch the pack and read its existing blocks. Decide:

- **Update an existing block** — if the new content belongs to a block that already exists (e.g., decisions, next steps), integrate it there. Rewrite the block so it reads as a coherent whole, not as an append.
- **Add a new block** — if the new content is a distinct topic not covered by any existing block.

The pack should read well from scratch at any point. Do not leave stale or contradicted information in place.

### Step 3 — Upsert and return

`upsert pack`:

```json
{
  "id": "<existing-id>",
  "content": "<top-level summary or intro — does not include block content>",
  "blocks": [
    {
      "id": "<existing-block-id>",
      "title": "<block-title>",
      "content": "<updated block content>"
    },
    { "title": "<new-block-title>", "content": "<new block content>" }
  ]
}
```

- Pass `"id"` on existing blocks to update them in place. Omit `"id"` to add a new block.
- `"content"` at the top level is a brief intro or summary. Block content lives in `"blocks[]"`, not in the top-level `"content"`.

```
context  <context-id>  <context-title>
block    <block-id>    <block-title>
```

---

## NEW

**Goal:** create a fresh pack with an initial block structure. Use when no relevant pack exists, or when the content is clearly a separate topic.

### Step 1 — Design the block structure

Before writing, decide how the content divides into blocks. Each block should be focused enough to update independently and broad enough to stay relevant over time.

**Design for selective fetch.** Because blocks can be loaded individually with `--block-id`, a well-split pack lets future sessions load only what they need without pulling the whole pack. Aim for blocks where any one block answers a distinct question on its own.

| Use case                              | Starting blocks                                             |
| ------------------------------------- | ----------------------------------------------------------- |
| Session / tool handoff                | Context, Decisions, Next Steps                              |
| Source summary (YouTube, blog, paper) | Source, Summary, Key Takeaways, Reuse Notes                 |
| Project memory / working context      | Purpose, Constraints, Decisions, References                 |
| Prompt or instruction pack            | When To Use This, Inputs, Instructions, Output Expectations |
| Project status                        | Summary, Progress, Blockers                                 |
| Team rules or plan                    | Purpose, Rules, Plan                                        |
| Task handoff                          | Current State, What Remains, Context for Recipient          |

Good block boundaries: topic changes, audience changes, or update frequency changes (e.g., "Decisions" is stable; "Next Steps" changes every session — keep them separate).

For templates, see [Content Templates](./templates/content.md).

### Step 2 — Write

Content rules:

- Each block is self-contained — no "see above" references
- Prefer scannable structure: bullets, short paragraphs, and small tables only when they help
- Include IDs, URLs, and precise names so the reader can act without searching
- **Title:** infer from session topic. Do not ask unless genuinely ambiguous.

### Step 3 — Publish

Default `private`. Use [PUBLISH](#publish) to go public.

`upsert pack`:

```json
{
  "title": "<title>",
  "content": "<top-level summary or intro>",
  "type": "context",
  "visibility": "private",
  "blocks": [
    { "title": "<block-title>", "content": "<block content>" },
    { "title": "<block-title>", "content": "<block content>" }
  ]
}
```

- Blocks are passed as a separate `"blocks[]"` array — **not** embedded in the top-level `"content"` string.
- `"content"` at the top level is a brief intro only. All substantive content belongs in blocks.

| Who needs it | `visibility` | `projects[]` |
| ------------ | ------------ | ------------ |
| Just me      | `"private"`  | omit         |
| My team      | `"private"`  | `["pj_xxx"]` |

```
context  <context-id>  <context-title>
block    <block-id>    <block-title>
```

---

## PUBLISH

**Goal:** make a pack publicly discoverable. Always requires explicit user confirmation before writing.

Two paths:

| Situation                                | Path                                    |
| ---------------------------------------- | --------------------------------------- |
| Existing private pack worth sharing      | Promote — change visibility to `public` |
| New content written for a broad audience | Create new public pack                  |

### Promote: private → public

1. `get pack` — fetch the pack. Run the [Public Review Gate](./references/visibility.md#public-review-gate). Flag issues before proceeding.
2. Confirm with user: **"Publish '{title}' (`{id}`) as public under category `{category}`?"**
3. `upsert pack`:

```json
{
  "id": "<id>",
  "visibility": "public",
  "category": "<category>"
}
```

### Create new public

Write for an external reader using the public guide template in [Content Templates](./templates/content.md). Content must be self-contained and pass the [Public Review Gate](./references/visibility.md#public-review-gate).

Choose category carefully — it determines how people find this pack. See [Category Reference](./references/visibility.md#category-reference).

Confirm with user, then `upsert pack`:

```json
{
  "title": "<title>",
  "content": "<content>",
  "type": "context",
  "visibility": "public",
  "category": "<category>"
}
```

Return ID, title, and share URL. For share URL resolution, see [Epismo Basics — Resolving Share URLs](../epismo-basics/SKILL.md#resolving-share-urls).

```
context  <context-id>  <context-title>
share    https://epismo.ai/hub/contexts/<id>
```

---

## GET

**Goal:** fetch a pack by ID, alias, or explicit read/open target.

### Route intent before resolving

Prefer **alias-first** unless the input is obviously a search query.

| Input pattern                                                           | Route         | Why                                           |
| ----------------------------------------------------------------------- | ------------- | --------------------------------------------- |
| `get ...`, `use ...`, `load ...`, `open ...`, `read ...`, `restore ...` | [GET](#get)   | Explicit retrieval                            |
| `find ...`, `search ...`, `what context do I have`, `show my packs`     | [FIND](#find) | Explicit discovery                            |
| Bare UUID                                                               | [GET](#get)   | IDs are unambiguous                           |
| Short ambiguous phrase                                                  | [GET](#get)   | Try alias first, then search if it misses     |
| Obvious search query, question, or long descriptive phrase              | [FIND](#find) | Discovery intent is clearer than alias intent |

### Resolve the input

1. UUID → `get pack --id`.
2. `@handle/name` or one compact token like `weekly-handoff` → `get pack --alias`; if it misses, run [FIND](#find) with the same text.
3. Short phrase like `auth refactor handoff` → try `get pack --alias` first; if it misses, run [FIND](#find).
4. Question, explicit search wording, or long descriptive text → [FIND](#find) first.
5. `get/read/open/use <target>` always counts as retrieval intent.

`get pack` — by `id`, or by `alias`. Default returns outline only; pass `--full` for all blocks, or `--block-id` to load a single block.

Read the most relevant block first. Continue from there without re-reading history.

---

## FIND

**Goal:** discover the right pack when the ID is not known.

`search pack` — scan titles only (no full content). Use this for natural-language discovery requests and multi-keyword topic phrases. Search in this order:

1. `type: context`, `query: <topic>`, `filter: { visibility: ["private"] }`
2. `type: context`, `query: <topic>`, `filter: { like: "liked" }`
3. `type: context`, `query: <topic>`, `filter: { visibility: ["public"] }`

Present the title list; if the match is clear, `get pack` immediately.

For filter keys, sort options, and search recipes, see [Search & Discovery](./references/search.md).

---

## UPDATE

**Goal:** edit an existing pack directly.

### ID or alias known

Always fetch before writing.

`get pack` → inspect current blocks → `upsert pack` with `id`, updated `"blocks[]"`, and updated `"content"` if needed.

For `update <alias> based on this conversation`:

- merge new context into the right existing blocks
- rewrite blocks coherently instead of appending raw notes
- add a new block only for a distinct new topic

### ID unknown

1. `search pack` — `type: context`, `query: <inferred topic>`, `filter: { visibility: ["private"] }`, `sort: updatedAt:desc`
2. **One clear match** → fetch and update without asking.
3. **Multiple candidates** → show titles and IDs, ask the user to pick.
4. **No match** → fall back to [NEW](#new).

**"Last pack" shortcut:** sort `updatedAt:desc`, take the first result.

---

## ORGANIZE

**Goal:** improve the block structure of an existing pack.

The most common need is not managing many packs — it's making the blocks inside one pack cleaner and more useful.

### Actions

| Situation                                             | Action                                                        |
| ----------------------------------------------------- | ------------------------------------------------------------- |
| A block has grown too large or covers multiple topics | **Split** — divide into two focused blocks, rewrite each      |
| Two blocks overlap or one has become redundant        | **Merge** — combine into one coherent block, remove the other |
| A block is no longer relevant                         | **Remove** — delete the block from the pack                   |
| The whole pack is no longer needed                    | **Delete** — delete the pack (needs approval)                 |

All changes are applied via `upsert pack` with the updated `"blocks[]"` array. Pass `"id"` on each block to update in place; omit blocks you want to remove. `delete pack` requires explicit user approval.

---

## Write Safety

1. **Default private** — `pack`, `new`, and `update` always write private. Use [PUBLISH](#publish) to go public.
2. **No silent writes** — if scope is unclear, ask once before writing.
3. **Approval required** — public publication, deletion, and overwriting another owner's pack require explicit user confirmation before upsert.

For the full approval matrix, see [Visibility & Sharing — Approval Boundary](./references/visibility.md#approval-boundary).

---

## Operation Output

After every operation, return in this order:

1. **Context and block** — always return both ID and title at each level that was affected
2. **What changed** — one sentence
3. **Next action** — single most useful next step (omit if none)

```
context  <context-id>  <context-title>
block    <block-id>    <block-title>    # when a specific block was created or updated
```

Skip analysis and evidence unless the user asked for it.

## Source of Truth

Repository: `https://github.com/epismoai/skills`
