---
name: epismo-project-operations
description: "Run day-to-day project operations through Epismo MCP or CLI: route user intent, read current state, and apply the smallest useful change. Covers track coordination, asset discovery, workflow reuse, and release."
---

# Project Operations

Operate on projects through Epismo MCP or CLI — from a quick track update to full goal restructuring and workflow asset release.

**Core principle: read current state → gather evidence → apply the smallest useful change.**

## Connection Prerequisite

This skill assumes Epismo access is already connected.

- Prefer the surface already available in the environment: MCP for tool-based agents, CLI for shell-driven agents.
- Use the bearer token accepted by the target surface or endpoint.
- MCP calls require an OAuth access token with `scope=mcp` and the correct `resource`.
- CLI can use `epismo login` or `EPISMO_ACCESS_TOKEN` for direct command execution. `--browser` is optional if needed.
- Protected API `/v1/*` endpoints use OAuth Bearer tokens.
- If MCP or CLI access is not ready yet, follow the auth/setup steps in the `github.com/epismoai/skills` README before continuing.

## Surface Conventions

Use canonical operation labels (e.g. `search track`, `upsert asset`) in instructions, plans, and reports. Both surfaces implement the same operations — resolve at execution time with one rule:

| Surface | Pattern                                        | Example                           |
| ------- | ---------------------------------------------- | --------------------------------- |
| `cli`   | `epismo {resource} {action} [--flags]`         | `epismo track search --type task` |
| `mcp`   | `epismo_{resource}_{action}` + same parameters | `epismo_track_search`             |

MCP tool name = CLI command with spaces and hyphens replaced by underscores. Parameters are identical across both surfaces.
Workspace selection is CLI-only — in MCP, workspace scope is implicit in the OAuth token.

**Operations** (track: `task` / `goal` / `note`; asset: `workflow` type):

- track: `search track` · `get track` · `upsert track` · `delete track`
- asset: `search asset` · `get asset` · `upsert asset` · `delete asset` · `import asset` · `like asset`
- workspace: `select workspace` (CLI only)
- credits: `check credit balance` · `start credit checkout`

For flag details, filter semantics, and `api` patterns (auth/setup only), see [Search & Filter](./references/search-filter.md).

## Scope Model

- `workspace` is the top-level access boundary.
- Projects live inside a workspace.
- Tracks and assets are read and written within the active workspace.
- `projects[]` narrows search scope and controls project membership inside the active workspace.
- Omitting `projects[]` on search means "search all accessible projects in the active workspace".

When the user says "project", resolve both layers before writing:

1. active workspace
2. target project or set of `projects[]`

Status values and query/filter behavior are documented in [Search & Filter](./references/search-filter.md).
Write examples and minimal upsert payloads are documented in [Runbook](./references/runbook.md).

## Intent Router

Match the user's intent to the right steps. Run only what the situation requires. When multiple intents overlap, start from the earliest unresolved step.

| Intent                                  | When to choose                                                                              | Steps                                            | Key reference                                                                                               |
| --------------------------------------- | ------------------------------------------------------------------------------------------- | ------------------------------------------------ | ----------------------------------------------------------------------------------------------------------- |
| Update existing work                    | A field change on something that already exists (status, assignee, date, priority, content) | 4 Coordinate — Partial Update                    | [Runbook](./references/runbook.md#1-partial-update)                                                         |
| Add one new item                        | Exactly one task, goal, or note; existing structure is valid                                | 4 Coordinate — New Item                          | [Runbook](./references/runbook.md#2-new-item-creation)                                                      |
| Plan multiple items or a new goal       | New goal structure, multi-step plan, or batch creation                                      | 2 Plan → 3 Discover → 4 Coordinate — Large-Scale | [Runbook](./references/runbook.md#3-large-scale-planning)                                                   |
| Unblock or rebalance                    | Stalled queue, overloaded assignee, dependency bottleneck, noisy backlog                    | 5 Risk → 4 Coordinate — Recovery                 | [Runbook](./references/runbook.md#4-recovery-and-backlog-hygiene)                                           |
| Find or reuse a workflow                | Need to discover, compare, or adapt an existing workflow                                    | 3 Discover                                       | [Search & Filter](./references/search-filter.md)                                                            |
| Design an AI-owned task                 | Delegating work to an AI agent with clear acceptance criteria                               | 4 Coordinate + AI Delegation                     | [AI Delegation](./references/ai-delegation.md)                                                              |
| Release / update / deprecate a workflow | Workflow is ready for publication, needs structural update, or should be retired            | 5 Risk (Quality Gate) → 6 Handoff                | [Workflow Quality](./references/workflow-quality.md) → [Workflow Release](./references/workflow-release.md) |
| Not enough credits                      | Epismo returns `Payment Required` or the user asks about credit balance / shortfall         | Stop — check balance or purchase credits         | [Credit Purchase](./references/credit-purchase.md)                                                          |
| Unclear intent                          | Cannot confidently match to any row above                                                   | 1 Intake — ask once                              | —                                                                                                           |

## 6-Step Flow

### 1 Intake

Confirm the user's desired outcome, constraints, timeline, active workspace, and target project scope.
Ask one clarifying question if intent is ambiguous. If the user does not answer, default to read-only discovery and report findings.

### 2 Plan

Run before structural changes (new goals, multi-item plans, dependency rewiring).
Follow [Runbook — Planning Checks](./references/runbook.md#planning-checks): diagnose current state → select mode → design ownership contracts → confirm write destination → set next checkpoint.
Always choose the smallest mode that satisfies the request.

### 3 Discover

Reuse before creating. Search workflow assets in this order: `private` → `liked` → `public`.
Use [Search & Filter](./references/search-filter.md) for filtered queues, status views, date ranges, and dependency traversal.
Use [Workflow Discovery template](./templates/operating-templates.md#6-workflow-discovery) only when comparing candidates and committing to a concrete adaptation plan.

### 4 Coordinate

Pick one execution mode from the [Runbook — Mode Selector](./references/runbook.md#mode-selector) and follow its playbook:

- **Partial Update** — localized field edits on existing entities.
- **New Item Creation** — exactly one new task / goal / note.
- **Large-Scale Planning** — multiple coordinated items with ownership contracts.
- **Recovery** — address overload, stall, or noisy backlog.

When a task status changes to `done`, run a downstream dependents check in the same Partial Update: query `dependsOn=["{task-id}"]` and report what is now unblocked.

Default destination rule: operational state lives in tracks; reusable patterns live in assets. Use workflow assets only when the structure should be stocked for reuse beyond the current execution context.

Design AI-owned tasks with [AI Delegation](./references/ai-delegation.md).
Use [Operating Templates](./templates/operating-templates.md) for structured write output.

### 5 Risk

Evaluate bottlenecks and dependency risk per [Runbook](./references/runbook.md).
Run backlog hygiene (stale, duplicate, low-signal) during recovery.
Apply [Workflow Quality](./references/workflow-quality.md) gate before any workflow release or update.

### 6 Handoff

- **Normal ops**: report lifecycle delta using [Operating Templates](./templates/operating-templates.md).
- **Workflow release**: execute [Workflow Release](./references/workflow-release.md) and report decision traceability.

## Write Safety

Three rules govern every write:

1. **Scope first** — confirm the target project before any write. If unclear, ask once.
2. **No silent writes** — if the user has not answered a clarification question, continue read-only analysis and report the pending decision.
3. **Approval for risk** — require explicit approval for large-scale plans, multi-entity recovery, ambiguous destinations, and destructive changes. Small, explicit, reversible updates do not need approval.

In practice, "scope first" means:

1. confirm the active workspace
2. confirm the target `projects[]`

Full rules: [Runbook — Write Safety](./references/runbook.md#write-safety-and-approval-gate) | [Workflow Release — Approval Boundary](./references/workflow-release.md#approval-boundary)
