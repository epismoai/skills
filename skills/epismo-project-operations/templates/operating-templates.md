# Operating Templates

Structured output templates for project operation writes. Each template maps to an execution mode from the [Runbook](../references/runbook.md).

Replace all `{...}` placeholders before executing. Default write destinations are tasks/goals/notes — escalate to workflow only when the structure needs reuse across projects or teams.

Guardrails:

- Write safety: [Runbook — Write Safety](../references/runbook.md#write-safety-and-approval-gate)
- Release decisions: [Workflow Release](../references/workflow-release.md)
- Quality gate: [Workflow Quality](../references/workflow-quality.md)

## Pre-Write Contract

Confirm before any write. Applies to all modes.

- **Project scope**: confirmed project name or ID
- **Outcome**: what success looks like
- **Constraints**: deadline, owner, risk, or sensitivity
- **Write destination**: tasks / goals / notes (add workflow if reusable)

## Status Reference

| Entity | Valid statuses                                    |
| ------ | ------------------------------------------------- |
| Task   | `backlog`, `todo`, `in_progress`, `done`          |
| Goal   | `not_started`, `on_track`, `at_risk`, `completed` |

`blocked_by_dependency` is a derived queue state, not a status field. See [Search & Filter](../references/search-filter.md#derived-queue-states).

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

Use when: exactly one new task, goal, or note must be added; existing structure is valid.

### Assess

- [ ] Destination project and item type confirmed (`task` / `goal` / `note`)
- [ ] Duplicate check done — no matching active item found
- [ ] Project scope confirmed

### Plan

- Title: {item title}
- Owner: {assignee or unassigned}
- Due date: {date or none}
- Context: {1-2 line description}
- Links: {goalId / parentId / dependsOn or none}

### Execute

- Write: one `upsert {task|goal|note}`
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

- [ ] Current goals, tasks, and notes reviewed
- [ ] Reuse check done — existing workflows scanned (`private` → `liked` → `public`)
- [ ] Why reuse is insufficient: {reason}
- [ ] Project scope confirmed

### Plan

For each item in the plan:

| Step | Title   | Owner      | Output        | Depends on         |
| ---- | ------- | ---------- | ------------- | ------------------ |
| A    | {title} | {AI/human} | {deliverable} | {step IDs or none} |
| B    | {title} | {AI/human} | {deliverable} | {step IDs or none} |
| C    | {title} | {AI/human} | {deliverable} | {step IDs or none} |

- Materialization: {goals/tasks/notes} + workflow if reusable: {yes → private now or after first run / no}

### Approve

- [ ] Explicit approval obtained (required for large-scale structures)
- Reason: {large-scale / ambiguous destination / critical assumption / destructive}

### Execute

- Writes: {list of upsert operations}
- Optional workflow: {upsert workflow or none}

### Report

- **Situation**: {what is active now}
- **Delta**: {items created/updated with structure summary}
- **Evidence**: {project items, workflows, or sources that informed the plan}
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

---

## 5) Pattern Capture and Workflow Release

Use when: a successful execution pattern is a candidate for reuse across projects or teams.

### Assess

- [ ] Execution evidence exists (project records with identifiable outcomes)
- [ ] Reuse boundary defined: {where this pattern fits / does not fit}
- [ ] Quality gate result: {pass / needs work} — per [Workflow Quality](../references/workflow-quality.md)

### Plan

- Generalizable structure: {steps, owners, dependencies — project-specific residue removed}
- Release target: {private / public}
- Decision: {release / update / keep private / deprecate}

### Approve

- [ ] Approval status: {approved / pending / rejected}
- Private staging writes do not require approval; public release and deprecation do.

### Execute

- `release` or `update` + `private`: set confirmed `projects`
- `release` or `update` + `public`: omit `projects`
- `deprecate`: explicit approval required

### Report

- **Situation**: {workflow state after action}
- **Delta**: {release action taken with target visibility}
- **Evidence**: {execution records and quality gate results}
- **Risks**: {duplication risk, scope limitations, or open concerns}
- **Next action**: {next review timing or follow-up}

---

## 6) Workflow Discovery

Use when: comparing candidate workflows and committing to a concrete adaptation plan before materialization.

### Assess

- [ ] Desired outcome defined: {what the user wants to achieve}
- [ ] Project context noted: {team and project constraints}
- [ ] Evidence sources checked:
  - Project items (goals/tasks/notes): {what was checked}
  - Workflows (`private` / `liked` / `public`): {what was checked}
  - External sources: {what was checked or N/A}

### Discover

Search plan:

- Project entities: {projects / status / date filters used}
- Workflow entities: {visibility / like / category filters used}
- Keyword queries (if needed): {2-6 domain keywords}

Candidate shortlist:

| #   | Pattern | Source        | Relevance      | Reuse cost     | Gaps            |
| --- | ------- | ------------- | -------------- | -------------- | --------------- |
| 1   | {title} | {source type} | {why relevant} | {low/med/high} | {what to adapt} |
| 2   | {title} | {source type} | {why relevant} | {low/med/high} | {what to adapt} |

### Recommend

- Selected: {title + source}
- Why: {brief comparison rationale}

### Adapt

- Keep as-is: {steps}
- Modify: {steps and changes}
- Add project-specific: {steps}
- Ownership: AI-owned {steps} / Human-owned {steps}

### Materialize

- Destination: {project tasks / private workflow / both}
- [ ] Write safety confirmed per [Runbook](../references/runbook.md#write-safety-and-approval-gate)
- [ ] If workflow release involved, approval boundary confirmed per [Workflow Release](../references/workflow-release.md#approval-boundary)

### Report

- **Situation**: {candidates found and recommendation}
- **Delta**: {adaptation plan or materialized items}
- **Evidence**: {discovery sources and comparison rationale}
- **Risks**: {gaps in adaptation, missing context}
- **Next action**: {smallest useful step}
