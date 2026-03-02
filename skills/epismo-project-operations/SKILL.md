---
name: epismo-project-operations
description: "Run day-to-day project operations through Epismo MCP: route user intent, read current state, and apply the smallest useful change. Covers intake, planning, coordination, risk, workflow discovery, and release."
---

# Project Operations

Operate on projects through Epismo MCP — from a quick status update to full goal restructuring and workflow release.

**Core principle: read current state → gather evidence → apply the smallest useful change.**

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
| Not enough credits                      | MCP returns `Payment Required`                                                              | Stop — purchase credits                          | [Credit Purchase](./references/credit-purchase.md)                                                          |
| Unclear intent                          | Cannot confidently match to any row above                                                   | 1 Intake — ask once                              | —                                                                                                           |

## 6-Step Flow

### 1 Intake

Confirm the user's desired outcome, constraints, timeline, and target project scope.
Ask one clarifying question if intent is ambiguous. If the user does not answer, default to read-only discovery and report findings.

### 2 Plan

Run before structural changes (new goals, multi-item plans, dependency rewiring).
Follow [Runbook — Planning Checks](./references/runbook.md#planning-checks): diagnose current state → select mode → design ownership contracts → confirm write destination → set next checkpoint.
Always choose the smallest mode that satisfies the request.

### 3 Discover

Reuse before creating. Search in this order: `private` → `liked` → `public`.
Use [Search & Filter](./references/search-filter.md) for filtered queues, status views, date ranges, and dependency traversal.
Use [Workflow Discovery template](./templates/operating-templates.md#6-workflow-discovery) only when comparing candidates and committing to a concrete adaptation plan.

### 4 Coordinate

Pick one execution mode from the [Runbook — Mode Selector](./references/runbook.md#mode-selector) and follow its playbook:

- **Partial Update** — localized field edits on existing entities.
- **New Item Creation** — exactly one new task / goal / note.
- **Large-Scale Planning** — multiple coordinated items with ownership contracts.
- **Recovery** — address overload, stall, or noisy backlog.

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

Full rules: [Runbook — Write Safety](./references/runbook.md#write-safety-and-approval-gate) | [Workflow Release — Approval Boundary](./references/workflow-release.md#approval-boundary)
