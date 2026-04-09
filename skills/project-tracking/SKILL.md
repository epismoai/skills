---
name: project-tracking
description: "Manage tasks and goals in Epismo projects. Run day-to-day tracking operations: create and update tracks, plan multi-step work, unblock stalled queues, and delegate tasks to AI agents. Trigger on: 'add a task', 'update status', 'plan this', 'what's blocked', 'rebalance workload', 'delegate to AI', or any intent to read or write project execution state."
---

# Project Tracking

Operate on tasks and goals in Epismo projects — from a quick status update to full goal restructuring.

**Core principle: read current state → apply the smallest useful change.**

> For connection setup, surface conventions, scope model, share URL resolution, and error handling, see [Epismo Basics](../epismo-basics/SKILL.md).
> For workflow pack discovery and release, see [Workflow Pack](../workflow-pack/SKILL.md).

> **Surface selection:** CLI and MCP connect to the same Epismo service. Use CLI if available; fall back to MCP if not. Never use both in the same session.

## Operations

| Operation      | CLI                                                      | MCP                   |
| -------------- | -------------------------------------------------------- | --------------------- |
| `search track` | `epismo track search --type task\|goal --filter '{...}'` | `epismo_track_search` |
| `get track`    | `epismo track get --id <id>`                             | `epismo_track_get`    |
| `upsert track` | `epismo track upsert --input '<json>'`                   | `epismo_track_upsert` |
| `delete track` | `epismo track delete --id <id>`                          | `epismo_track_delete` |

For filter keys, status values, entity relationships, and search recipes, see [Search & Filter](./references/search.md).

## Intent Router

| Intent                            | When to choose                                                                            | Steps                               | Key reference                                                     |
| --------------------------------- | ----------------------------------------------------------------------------------------- | ----------------------------------- | ----------------------------------------------------------------- |
| Update existing work              | Field change on something that already exists (status, assignee, date, priority, content) | 3 Coordinate — Partial Update       | [Runbook](./references/runbook.md#1-partial-update)               |
| Add one new item                  | Exactly one task or goal; existing structure is valid                                     | 3 Coordinate — New Item             | [Runbook](./references/runbook.md#2-new-item-creation)            |
| Plan multiple items or a new goal | New goal structure, multi-step plan, or batch creation                                    | 2 Plan → 3 Coordinate — Large-Scale | [Runbook](./references/runbook.md#3-large-scale-planning)         |
| Unblock or rebalance              | Stalled queue, overloaded assignee, dependency bottleneck, noisy backlog                  | 4 Risk → 3 Coordinate — Recovery    | [Runbook](./references/runbook.md#4-recovery-and-backlog-hygiene) |
| Design an AI-owned task           | Delegating work to an AI agent with clear acceptance criteria                             | 3 Coordinate + AI Delegation        | [AI Delegation](./references/ai-delegation.md)                    |
| Unclear intent                    | Cannot confidently match any row above                                                    | 1 Intake — ask once                 | —                                                                 |

## 4-Step Flow

### 1 Intake

Confirm the user's desired outcome, constraints, timeline, active workspace, and target project scope.
Ask one clarifying question if intent is ambiguous. If unanswered, default to read-only and report findings.

### 2 Plan

Run before structural changes (new goals, multi-item plans, dependency rewiring).
Follow [Runbook — Planning Checks](./references/runbook.md#planning-checks): diagnose current state → select mode → design ownership contracts → confirm write destination → set next checkpoint.
Always choose the smallest mode that satisfies the request.

### 3 Coordinate

Pick one execution mode from [Runbook — Mode Selector](./references/runbook.md#mode-selector):

- **Partial Update** — localized field edits on existing entities.
- **New Item Creation** — exactly one new task / goal.
- **Large-Scale Planning** — multiple coordinated items with ownership contracts.
- **Recovery** — address overload, stall, or noisy backlog.

When a task status changes to `done`, run a downstream dependents check: query `dependsOn=["<task-id>"]` and report what is now unblocked.

Operational state lives in tracks. Use workflow packs only when the structure should be reused beyond the current execution context — see [Workflow Pack](../workflow-pack/SKILL.md).

Design AI-owned tasks with [AI Delegation](./references/ai-delegation.md).
Use [Track Operations](./templates/operations.md) for structured write output after the [Runbook](./references/runbook.md) checks are complete.

### 4 Risk

Evaluate bottlenecks and dependency risk per [Runbook](./references/runbook.md).
Run backlog hygiene (stale, duplicate, low-signal) during recovery.

## Write Safety

1. **Scope first** — confirm the target project before any write. If unclear, ask once.
2. **No silent writes** — if a clarification question is unanswered, continue read-only.
3. **Approval for risk** — require explicit approval for large-scale plans, multi-entity recovery, and destructive changes.

## Source of Truth

Repository: `https://github.com/epismoai/skills`
