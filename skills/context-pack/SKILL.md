---
name: context-pack
description: "Pack, share, and load context using Epismo context assets. Trigger on: 'pack this', 'new pack', 'use <id>', 'load my context', 'what context do I have', 'restore session', 'save this context', 'share with my team', 'pack this up', 'hand this off', 'publish this guide', 'organize my packs', or any intent to persist or retrieve knowledge across tools or sessions."
---

# Context Pack

Store and retrieve context through Epismo context assets — so knowledge survives tool switches, session ends, and team handoffs.

The unit of organization is a **block** inside a pack. The goal is one well-structured pack per topic, grown over time by adding and refining blocks — not many small packs.

> For Epismo connection setup, CLI/MCP surface conventions, scope model, share URL resolution, and error handling, see [Epismo Basics](../epismo-basics/SKILL.md).

## Commands

| Command | Natural language triggers | → |
| ------- | ------------------------- | - |
| `pack` | pack this, save this, summarize session | [PACK](#pack) |
| `new` | new pack, start fresh, hand off a task, share project status | [NEW](#new) |
| `publish [<id>]` | publish this, make this public, publish a guide, share with community | [PUBLISH](#publish) |
| `use <id\|alias>` | load this ID, restore from ID, load alias | [USE](#use) |
| `find <query>` | what context do I have, load my context, search | [FIND](#find) |
| `update [<id\|alias>]` | edit this pack, update my last pack | [UPDATE](#update) |
| `organize` | reorganize blocks, split this, clean up | [ORGANIZE](#organize) |
| — | unclear intent | Ask once |

## Operations (context assets)

| Operation | CLI | MCP |
| --------- | --- | --- |
| `search asset` | `epismo asset search --type context [--query <keywords>] [--filter '{...}']` | `epismo_asset_search` |
| `get asset` | `epismo asset get --id <id> [--full] [--block-id <id>]`<br>`epismo asset get --type context --alias <alias>` | `epismo_asset_get` |
| `upsert asset` | `epismo asset upsert --input '<json>'` | `epismo_asset_upsert` |
| `delete asset` | `epismo asset delete --id <id>` | `epismo_asset_delete` |
| `like asset` | `epismo asset like --id <id> --liked` | `epismo_asset_like` |
| `upsert alias` | `epismo alias upsert --type context --id <id> --alias <name>` | — |
| `get alias` | `epismo alias get --alias <name>` | — |
| `list aliases` | `epismo alias list --type context` | — |
| `delete alias` | `epismo alias delete --alias <name>` | — |

---

## PACK

**Goal:** save current context as a block inside the right pack. Find an existing pack and add or refine a block; create a new pack only when nothing relevant exists.

### Step 1 — Find the right pack

Infer 2–4 topic keywords from the current session. Search private packs (titles only — one fast call).

`search asset` — `type: context`, `query: <inferred topic>`, `filter: { visibility: ["private"] }`, `sort: updatedAt:desc`

| Result | Action |
| ------ | ------ |
| One clear match | Fetch it → add or update a block → upsert |
| Multiple candidates | Show titles and IDs → ask user to pick |
| No match | Proceed as [NEW](#new) |

### Step 2 — Identify the right block

`get asset` — fetch the pack and read its existing blocks. Decide:

- **Update an existing block** — if the new content belongs to a block that already exists (e.g., decisions, next steps), integrate it there. Rewrite the block so it reads as a coherent whole, not as an append.
- **Add a new block** — if the new content is a distinct topic not covered by any existing block.

The pack should read well from scratch at any point. Do not leave stale or contradicted information in place.

### Step 3 — Upsert and return

`upsert asset`:

```json
{
  "id": "<existing-id>",
  "content": "<updated content with refined blocks>"
}
```

```
context  <context-id>  <context-title>
block    <block-id>    <block-title>
```

---

## NEW

**Goal:** create a fresh pack with an initial block structure. Use when no relevant pack exists, or when the content is clearly a separate topic.

### Step 1 — Design the block structure

Before writing, decide how the content divides into blocks. Each block should be focused enough to update independently and broad enough to stay relevant over time.

**Design for selective fetch.** Because blocks can be loaded individually with `--block-id`, a well-split pack lets future sessions load only what they need without pulling the whole asset. Aim for blocks where any one block answers a distinct question on its own.

| Use case | Starting blocks |
| -------- | --------------- |
| Session / tool handoff | Context, Decisions, Next Steps |
| Source summary (YouTube, blog, paper) | Source, Summary, Key Takeaways, Reuse Notes |
| Project memory / working context | Purpose, Constraints, Decisions, References |
| Prompt or instruction pack | When To Use This, Inputs, Instructions, Output Expectations |
| Project status | Summary, Progress, Blockers |
| Team rules or plan | Purpose, Rules, Plan |
| Task handoff | Current State, What Remains, Context for Recipient |

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

`upsert asset`:

```json
{
  "title": "<title>",
  "content": "<content>",
  "type": "context",
  "visibility": "private"
}
```

| Who needs it | `visibility` | `projects[]` |
| ------------ | ------------ | ------------ |
| Just me | `"private"` | omit |
| My team | `"private"` | `["pj_xxx"]` |

```
context  <context-id>  <context-title>
block    <block-id>    <block-title>
```

---

## PUBLISH

**Goal:** make a pack publicly discoverable. Always requires explicit user confirmation before writing.

Two paths:

| Situation | Path |
| --------- | ---- |
| Existing private pack worth sharing | Promote — change visibility to `public` |
| New content written for a broad audience | Create new public pack |

### Promote: private → public

1. `get asset` — fetch the pack. Run the [Public Review Gate](./references/visibility.md#public-review-gate). Flag issues before proceeding.
2. Confirm with user: **"Publish '{title}' (`{id}`) as public under category `{category}`?"**
3. `upsert asset`:

```json
{
  "id": "<id>",
  "visibility": "public",
  "category": "<category>"
}
```

### Create new public

Write for an external reader using the public guide template in [Content Templates](./templates/content.md). Content must be self-contained and pass the [Public Review Gate](./references/visibility.md#public-review-gate).

Choose category carefully — it determines how people find this pack:

| Category | When to use |
| -------- | ----------- |
| `productivity` | Workflow guides, session patterns, working norms |
| `programming` | Code review checklists, architecture decisions, API design |
| `design` | Design system norms, component guidelines, UX decisions |
| `marketing` | Campaign rules, messaging frameworks, brand guidelines |
| `operations` | Runbooks, incident post-mortems, deployment norms |
| `learning` | Onboarding guides, tutorials, knowledge bases |
| `life` | Personal knowledge, life planning, non-work contexts |

Confirm with user, then `upsert asset`:

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

## USE

**Goal:** load a pack when the user gives an ID, an alias, or a short label that might be either.

### Resolve the input

| Input looks like | Try first | Fallback |
| ---------------- | --------- | -------- |
| UUID | `get asset` by `id` | — |
| `@handle/name` | `get asset` by `alias` | tell user if not found |
| short word / phrase | `get asset` by `alias` | if not found → [FIND](#find) with it as keyword |

When a short word fails alias resolution, treat it as a search keyword rather than an error. Do not ask the user to clarify before trying both paths.

`get asset` — by `id`, or by `alias` with `type: context`. Default returns outline only; pass `--full` for all blocks, or `--block-id` to load a single block.

Read the most relevant block first. Continue from there without re-reading history.

---

## FIND

**Goal:** discover the right pack when the ID is not known.

`search asset` — scan titles only (no full content). Search in this order:

1. `type: context`, `query: <topic>`, `filter: { visibility: ["private"] }`
2. `type: context`, `query: <topic>`, `filter: { like: "liked" }`
3. `type: context`, `query: <topic>`, `filter: { visibility: ["public"] }`

Present the title list; if the match is clear, `get asset` immediately.

For filter keys, sort options, and search recipes, see [Search & Discovery](./references/search.md).

---

## UPDATE

**Goal:** edit an existing pack directly.

### ID or alias known

`get asset` → `upsert asset` with `id` and updated `content`.

### ID unknown

1. `search asset` — `type: context`, `query: <inferred topic>`, `filter: { visibility: ["private"] }`, `sort: updatedAt:desc`
2. **One clear match** → fetch and update without asking.
3. **Multiple candidates** → show titles and IDs, ask the user to pick.
4. **No match** → fall back to [NEW](#new).

**"Last pack" shortcut:** sort `updatedAt:desc`, take the first result.

---

## ORGANIZE

**Goal:** improve the block structure of an existing pack.

The most common need is not managing many packs — it's making the blocks inside one pack cleaner and more useful.

### Actions

| Situation | Action |
| --------- | ------ |
| A block has grown too large or covers multiple topics | **Split** — divide into two focused blocks, rewrite each |
| Two blocks overlap or one has become redundant | **Merge** — combine into one coherent block, remove the other |
| A block is no longer relevant | **Remove** — delete the block from the pack |
| The whole pack is no longer needed | **Delete** — delete the pack (needs approval) |

All changes are applied via `upsert asset` (rewrite the full content with the new block structure). `delete asset` requires explicit user approval.

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
