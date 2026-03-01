# Operating Templates

This document provides execution-first templates for project operations.
Default day-to-day writes target tasks/goals/notes. Use workflow materialization when reusable structure is required.

Guardrails:

- Approval/scope rules follow [Project Operations Runbook](../references/project-operations-runbook.md) (`Write Safety and Approval Gate`).
- Release decisions follow [Workflow Release](../references/workflow-release.md).
- Release quality checks follow [Workflow Quality](../references/workflow-quality.md).

## Quick Router (Pick One Mode First)

1. Partial Update
   - Existing item(s) need localized field changes only.
2. New Item Creation
   - One new task/goal/note should be added without structural redesign.
3. Large-Scale Planning
   - Multiple tasks/goals must be designed, sequenced, and assigned.
4. Recovery and Backlog Hygiene
   - Throughput is stalled or backlog quality is degraded.

Escalate to workflow:

- When the planned structure is expected to be reused across projects/teams.

## Minimal Input Contract (Ask/Confirm Before Writes)

Use this short block for all modes:

```markdown
### Required Inputs

- Project scope: <project name or confirmed project IDs>
- Outcome: <what success looks like>
- Constraint: <deadline/owner/risk/sensitivity>
- Write destination: <tasks/goals/notes; add workflow if reusable pattern is intended>
```

## Standard User Report Block

Use this in every mode output:

```markdown
### User Report

- Situation: <what is active now>
- Delta: <what changed or why no write>
- Evidence used: <project goals/tasks/notes/workflow/external sources>
- Risks/decisions: <what needs confirmation>
- Next action: <smallest useful step>
```

## Status Reminder (Use Entity-Specific Statuses)

- Task statuses: `backlog`, `todo`, `in_progress`, `done`
- Goal statuses: `not_started`, `on_track`, `at_risk`, `completed`
- For exact filter usage, follow [Search Filter](../references/search-filter.md).

## 1) Partial Update

Use when existing work is valid and only localized edits are needed.

```markdown
### Request

- Target project: <project>
- Target entities: <task|goal|note titles>
- Requested fields: <status|assignee|due date|priority|content>
- If status update: <use valid status set for that entity type>

### Validation

- Current values read: <value list>
- Change is real (no-op check): <yes/no>
- Scope confirmed: <yes/no>

### Update Plan

- Localized changes only: <entity.field -> new value>
- Structural changes included: <none>

### Execution

- Write actions: <upsert task|upsert goal|upsert note>
- `projects` set to: <confirmed project ID(s)>

### User Report

Use Standard User Report Block.
```

## 2) New Item Creation

Use when exactly one new item should be added.

```markdown
### Request

- Target project: <project>
- Item type: <task|goal|note>
- Intent: <what this item enables now>

### Validation

- Duplicate check in active queue/backlog: <none|matches>
- If matches found: <reuse existing|create anyway + reason>

### Creation Plan

- New item: <title>
- Owner: <assignee or none>
- Due date: <date or none>
- Minimal context: <1-2 lines>
- Linked refs: <goal/task/note IDs or none>

### Execution

- Write action: <upsert task|upsert goal|upsert note>
- `projects` set to: <confirmed project ID(s)>

### User Report

Use Standard User Report Block.
```

## 3) Large-Scale Planning

Use when multiple items must be created/updated as one coordinated plan.

```markdown
### Request

- Outcome: <desired result>
- Constraints: <time/quality/ownership>
- Target scope: <project>

### Reuse Check (Required)

- Existing goals/tasks reviewed: <list>
- Existing workflows reviewed (`private`/`liked`/`public`): <list>
- Why reuse is insufficient: <reason>

### Plan Design

- Step A: <title> | owner: <AI/human> | output: <deliverable> | dependsOn: <ids/none>
- Step B: <title> | owner: <AI/human> | output: <deliverable> | dependsOn: <ids/none>
- Step C: <title> | owner: <AI/human> | output: <deliverable> | dependsOn: <ids/none>

### Materialization Decision

- Day-to-day destination (default): <goals/tasks/notes>
- Workflow needed for reuse: <yes/no>
- If yes: <private workflow now|after first successful run>

### Approval Check

- Explicit approval required: <yes/no>
- Reason: <large-scale structure|ambiguous destination|critical assumption|destructive risk>

### Execution

- Planned writes: <upsert goal/task/note list>
- Optional workflow action: <upsert workflow|none>

### User Report

Use Standard User Report Block.
```

## 4) Recovery and Backlog Hygiene

Use when delivery is slowed by overload, dependency blocking, or noisy backlog.

```markdown
### Trigger

- Recovery reason: <stall|overload|deadline risk|backlog cleanup request>
- Target scope: <project>

### Snapshot

- Active queue:
  - `backlog`: <count>
  - `todo`: <count>
  - `in_progress`: <count>
  - `blocked_by_dependency`: <count>
- Assignee load: <top overloaded owners>
- Dependency hotspots: <tasks with many dependents>
- Backlog hygiene:
  - stale todo (>30d): <count>
  - possible duplicates: <count>
  - low-signal items: <count>

### Recovery Decision

- Keep unchanged: <list>
- Throughput fixes: <reassign|resequence|dependency rewiring>
- Backlog cleanup: <merge|archive|close stale items>
- Smallest high-impact set: <specific entities>

### Approval Check

- Explicit approval required: <yes/no>
- Reason: <multi-entity restructuring|destructive/hard-to-reverse changes>

### Execution

- Updates applied: <task/goal/note -> change>
- `projects` set to: <confirmed project ID(s)>

### User Report

Use Standard User Report Block.
```

## 5) Pattern Capture and Workflow Release

Use after successful execution when converting proven work into reusable workflow assets.

```markdown
### Success Evidence

- What worked: <outcome + why>
- Reuse boundary: <where this pattern fits / does not fit>

### Pattern Extraction

- Generalizable structure: <steps/owners/dependencies>
- Remove project-specific residue: <names/IDs/URLs/context>

### Release Readiness

- Quality gate result: <pass|needs work>
- Release target: <private|public>
- Decision: <release|update|keep private|deprecate>
- Approval status: <approved|pending|rejected>

### Release Execution

- `release/update` + `private`: set confirmed `projects`
- `release/update` + `public`: omit `projects`
- `deprecate`: explicit approval required

### User Report

Use Standard User Report Block.
```
