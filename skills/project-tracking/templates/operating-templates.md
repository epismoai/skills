# Operating Templates

Structured output templates for project tracking writes. Each template maps to an execution mode from the [Runbook](../references/runbook.md).

Replace all `{...}` placeholders before executing. Default write destinations are tracks (`task` / `goal`).
Surface conventions from [Epismo Basics](../../epismo-basics/SKILL.md).

Guardrails:
- Write safety: [Runbook — Write Safety](../references/runbook.md#write-safety-and-approval-gate)

## Pre-Write Contract

Confirm before any write. Applies to all modes.

- **Project scope**: confirmed project name or ID
- **Outcome**: what success looks like
- **Constraints**: deadline, owner, risk, or sensitivity
- **Write destination**: tracks (`task` / `goal`) or assets (`workflow` when reusable → see [Workflow Hub](../../workflow-hub/SKILL.md))

## Status Reference

| Entity | Valid statuses |
| ------ | -------------- |
| Task | `backlog`, `todo`, `in_progress`, `done` |
| Goal | `not_started`, `on_track`, `at_risk`, `postponed`, `completed` |

`blocked_by_dependency` is a derived queue state, not a status field. See [Search & Filter](../references/search.md#derived-queue-states).

---

## 1) Partial Update

Use when: existing entities need localized field edits only (status, assignee, date, priority, content).

### Assess

- [ ] Target entities identified (names and current values read)
- [ ] Change is real — not a no-op
- [ ] Project scope confirmed
- [ ] If task status -> `done`, downstream dependents check queued

### Execute

- Writes: `{entity.field → new value}` for each change
- Follow-up check when task moved to `done`: `{ready_now / blocked_by_dependency / none}`
- Structural changes: none

### Report

- **Situation**: {what is active now}
- **Delta**: {fields changed on which entities; newly unblocked tasks / still blocked tasks / no downstream dependents}
- **Evidence**: {source that informed the change}
- **Risks**: {none expected / any concern}
- **Next action**: {smallest useful step}

---

## 2) New Item Creation

Use when: exactly one new task or goal must be added; existing structure is valid.

### Assess

- [ ] Destination project and item type confirmed (`task` / `goal`)
- [ ] Duplicate check done — no matching active item found
- [ ] Project scope confirmed

### Plan

- Title: {item title}
- Owner: {assignee or unassigned}
- Due date: {date or none}
- Context: {1-2 line description}
- Links: {goalId / parentId / dependsOn or none}

### Execute

- Write: one `upsert track` for `{task|goal}`
- Projects: `{confirmed project ID}`

### Report

- **Situation**: {what is active now}
- **Delta**: {created item with key fields}
- **Evidence**: {why this item was needed}
- **Risks**: {none expected / any concern}
- **Next action**: {smallest useful step}

---

## 3) Large-Scale Planning

Use when: multiple items, a new goal structure, or a coordinated multi-step plan is needed.

### Assess

- [ ] Current goals and tasks reviewed
- [ ] Reuse check done — existing workflows scanned (`private` → `liked` → `public`)
- [ ] Why reuse is insufficient: {reason}
- [ ] Project scope confirmed

### Plan

For each item in the plan:

| Step | Title | Owner | Output | Depends on |
| ---- | ----- | ----- | ------ | ---------- |
| A | {title} | {AI/human} | {deliverable} | {step IDs or none} |
| B | {title} | {AI/human} | {deliverable} | {step IDs or none} |

- Materialization: {tracks only / tracks + workflow asset}

### Approve

- [ ] Explicit approval obtained (required for large-scale structures)
- Reason: {large-scale / ambiguous destination / critical assumption / destructive}

### Execute

- Writes: {list of `upsert track` operations}
- Optional workflow asset: see [Workflow Hub](../../workflow-hub/SKILL.md)

### Report

- **Situation**: {what is active now}
- **Delta**: {items created/updated with structure summary}
- **Evidence**: {tracks, assets, or sources that informed the plan}
- **Risks**: {open questions or dependency concerns}
- **Next action**: {smallest useful step}

---

## 4) Recovery and Backlog Hygiene

Use when: delivery is slowed by overload, stall, dependency blocking, or backlog noise.

### Assess

- [ ] Recovery trigger identified: {stall / overload / deadline risk / backlog cleanup}
- [ ] Project scope confirmed
- Active queue snapshot:
  - `backlog`: {count} | `todo`: {count} | `in_progress`: {count} | `blocked`: {count}
- Assignee load: {top overloaded owners}
- Dependency hotspots: {tasks with many dependents}
- Backlog hygiene: stale (>30d): {count} | duplicates: {count} | low-signal: {count}

### Plan

- Keep unchanged: {list}
- Throughput fixes: {reassign / resequence / dependency rewiring}
- Backlog cleanup: {merge / archive / close}
- Smallest high-impact set: {specific entities and actions}

### Approve

- [ ] Explicit approval obtained (required for multi-entity restructuring)

### Execute

- Writes: {entity → change for each affected item}
- Projects: `{confirmed project ID}`

### Report

- **Situation**: {queue state after changes}
- **Delta**: {what was reassigned, resequenced, archived, or closed}
- **Evidence**: {metrics and signals that drove the decision}
- **Risks**: {remaining bottlenecks or concerns}
- **Next action**: {smallest useful step}
