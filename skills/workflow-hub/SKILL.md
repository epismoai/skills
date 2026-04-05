---
name: workflow-hub
description: "Discover, reuse, and release workflow assets in Epismo. Trigger on: 'find a workflow', 'reuse this pattern', 'adapt this workflow', 'release this as a workflow', 'publish a workflow', 'update workflow', 'deprecate workflow', or any intent to search community workflows or capture a proven execution pattern for reuse."
---

# Workflow Hub

Discover, adapt, and release reusable workflow assets in Epismo — from finding a community pattern to publishing a proven execution structure.

**Core principle: reuse before creating. Search private → liked → public before building anything new.**

> For connection setup, surface conventions, scope model, share URL resolution, and error handling, see [Epismo Basics](../epismo-basics/SKILL.md).
> For task/goal tracking, see [Project Tracking](../project-tracking/SKILL.md).
> For workflow query patterns and loading, see [Search & Discovery](./references/search.md).

## Operations

| Operation | CLI | MCP |
| --------- | --- | --- |
| `search asset` | `epismo asset search --type workflow --filter '{...}'` | `epismo_asset_search` |
| `get asset` | `epismo asset get --id <id> [--full] [--step-id <ids>]`<br>`epismo asset get --alias <alias>` | `epismo_asset_get` |
| `upsert asset` | `epismo asset upsert --input '<json>'` | `epismo_asset_upsert` |
| `delete asset` | `epismo asset delete --id <id>` | `epismo_asset_delete` |
| `like asset` | `epismo asset like --id <id> --liked` | `epismo_asset_like` |
| `upsert alias` | `epismo alias upsert --type workflow --id <id> --alias <name>` | — |
| `get alias` | `epismo alias get --alias <name>` | — |
| `list aliases` | `epismo alias list --type workflow` | — |
| `delete alias` | `epismo alias delete --alias <name>` | — |

## Intent Router

| Intent | Steps | Key reference |
| ------ | ----- | ------------- |
| Find a workflow to reuse | [DISCOVER](#discover-workflow) | — |
| Adapt and materialize a workflow | [DISCOVER](#discover-workflow) → [ADAPT](#adapt-and-materialize) | [Workflow Patterns — Discovery](./templates/patterns.md#2-workflow-discovery) |
| Capture a proven pattern | [CAPTURE](#capture-and-release) | [Workflow Patterns — Pattern Capture](./templates/patterns.md#1-pattern-capture-and-workflow-release) |
| Release / update / deprecate | [CAPTURE](#capture-and-release) → quality gate → release | [Quality Gate](./references/quality.md) → [Release](./references/release.md) |
| Unclear | Ask once | — |

For visibility rules, categories, project scoping, sharing, and approval boundaries, see [Visibility & Sharing](./references/visibility.md).

---

## DISCOVER Workflow

**Goal:** find relevant workflows without fetching more than needed.

### Step 0 — Resolve if ID or alias given

If the user supplies a UUID, alias (`@handle/name`), or short label, try direct fetch before scanning:

| Input looks like | Try first | Fallback |
| ---------------- | --------- | -------- |
| UUID | `asset get --id` | — |
| `@handle/name` | `asset get --alias` | tell user if not found |
| short word / phrase | `asset get --alias` | if not found → Step 1 with it as keyword |

Do not ask the user to clarify before trying both paths.

### Step 1 — Scan: titles only

`search asset` returns `id` + `title` only — no step content. Review titles before fetching.

Search in this order:

```bash
# 1. Your own private workflows
epismo asset search --type workflow --query "<topic>" \
  --filter '{"visibility":["private"]}'

# 2. Workflows you bookmarked
epismo asset search --type workflow \
  --filter '{"like":"liked"}'

# 3. Community public workflows
epismo asset search --type workflow --query "<topic>" \
  --filter '{"visibility":["public"]}'
```

For filter keys (`category`, `minLikeCount`, `updatedAtFrom`, etc.), see [Search & Discovery](./references/search.md#supported-filter-keys).

### Step 2 — Select: identify candidates

From the title list, shortlist relevant workflows. Skip clearly unrelated ones. If ambiguous, show titles and ask the user.

### Step 3 — Fetch: get full content

```bash
# Full content (all steps)
epismo asset get --id <asset-id> --full

# Fetch only specific steps
epismo asset get --id <asset-id> --step-id <step-id-1>,<step-id-2>

# Fetch by alias (full content)
epismo asset get --type workflow --alias <alias> --full
```

---

## ADAPT and Materialize

After finding a candidate:

1. Identify steps to keep, modify, or replace.
2. Remove project-specific names, IDs, and one-off constraints.
3. Assign owners — `human` for decisions/reviews, AI agent name for generation/research.
4. Choose destination: project tracks (immediate execution) or private workflow asset (reuse).

Before materializing anything, confirm the desired outcome, project context, evidence sources checked, and destination. If the destination includes released workflow content, also apply the approval boundary from [Release](./references/release.md#approval-boundary).

Use [Workflow Discovery template](./templates/patterns.md#2-workflow-discovery) for structured recommendations and adaptation reports.

---

## CAPTURE and Release

**Goal:** extract a reusable pattern from a successful execution and publish it.

### Step 1 — Quality Gate

Run [Quality Gate](./references/quality.md) before any release decision. All 8 criteria must pass for a public release. A single fail means: fix or keep private.

### Step 2 — Release Decision

| Decision | When |
| -------- | ---- |
| `release` | Execution proven, structure generalizable, no superior equivalent exists |
| `update` | Improves an existing released pattern without breaking consumers |
| `keep private` | Evidence incomplete or quality gate has open items |
| `deprecate` | Obsolete, unsafe, or superseded |

Run the pre-decision checks in [Release](./references/release.md) before choosing one of these paths. That includes confirming action type, target workflow, release target, duplication risk, and write scope.

See [Release](./references/release.md) for the approval boundary and required output.

### Step 3 — Upsert

**Private workflow:**

```bash
epismo asset upsert --input '{
  "title": "Daily Operating Rhythm",
  "content": "Reusable daily workflow.",
  "type": "workflow",
  "visibility": "private",
  "projects": ["pj_123"],
  "workflow": [
    {
      "id": "t001",
      "title": "Morning briefing",
      "content": "Review priorities and produce a short plan.",
      "dueDate": "",
      "dependsOn": [],
      "parentId": "",
      "assignee": "human"
    }
  ]
}'
```

**Public workflow (requires explicit user approval — see [Visibility & Sharing](./references/visibility.md#approval-boundary)):**

```bash
epismo asset upsert --input '{
  "title": "AI-Assisted Code Review Process",
  "content": "...",
  "type": "workflow",
  "visibility": "public",
  "category": "programming",
  "workflow": [...]
}'
```

---

## Write Safety

1. **Scope first** — confirm workspace and project scope before any write. If unclear, ask once.
2. **No silent writes** — if a clarification question is unanswered, continue read-only.
3. **Approval for risk** — public publication, deprecation, and overwriting another owner's asset require explicit approval. See [Visibility & Sharing — Approval Boundary](./references/visibility.md#approval-boundary).

Use [Workflow Patterns](./templates/patterns.md) for report structure only. Do not treat those templates as a substitute for [Quality Gate](./references/quality.md) or [Release](./references/release.md).

## Source of Truth

Repository: `https://github.com/epismoai/skills`
