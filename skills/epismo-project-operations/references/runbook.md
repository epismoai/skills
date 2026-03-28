# Project Operations Runbook

Governs day-to-day operational writes on tasks, goals, and notes.
For workflow release/update/deprecate, use [Workflow Release](./workflow-release.md).
For query/filter semantics, use [Search & Filter](./search-filter.md).
Operation labels below use the canonical surface conventions from [Project Operations](../SKILL.md#surface-conventions).
Tracks hold project execution state; assets hold reusable stock content such as workflows.

Status values must match entity-specific definitions. `blocked_by_dependency` is a derived queue state, not a status field. Scope semantics and status definitions live in [Search & Filter](./search-filter.md).

## Mode Selector

Choose one mode before writing. **Default: start with the smallest mode and escalate only if it does not fit.**

| Signal                                                                                   | Mode                 |
| ---------------------------------------------------------------------------------------- | -------------------- |
| Existing entity needs a localized field edit (status, assignee, date, priority, content) | Partial Update       |
| Exactly one new item; existing structure is valid                                        | New Item Creation    |
| Multiple new items, a new goal, or a multi-step execution plan                           | Large-Scale Planning |
| Overload, stalled queue, dependency hotspots, or backlog drift                           | Recovery             |

When in doubt between New Item Creation and Large-Scale Planning, prefer New Item Creation — you can always add more items in a follow-up.

## Planning Checks

Run before structural changes. Skip for simple partial updates and single-item creation.

1. **Diagnose** — run `search track` and `search asset` to pull current goals, tasks, notes, and reusable workflows. Separate planned items from actively-moving items.
   → Output: current state summary.
2. **Select mode** — choose one mode from the selector above. State why alternatives were rejected.
   → Output: chosen mode + one-line rationale.
3. **Design contracts** — for each planned change, define owner, expected output, and true prerequisites. Keep independent work parallel.
   → Output: ownership and dependency map.
4. **Confirm destination** — write to tracks first. Add a private workflow only when reusable structure is clearly needed.
   → Output: write destination confirmed.
5. **Set checkpoint** — define the smallest useful next review point (date, event, or deliverable).
   → Output: next review trigger.

## Minimal Upsert Payloads

Use these as the smallest safe starting shapes for CLI `--input` payloads. Add optional fields only when needed.

### Task

```json
{
  "title": "Investigate update task error",
  "projects": ["pj_123"],
  "task": {
    "status": "todo"
  }
}
```

### Goal

```json
{
  "title": "Release agent task assignment",
  "projects": ["pj_123", "pj_456"],
  "goal": {
    "status": "not_started",
    "progress": 0
  }
}
```

### Workflow Asset

```json
{
  "title": "Daily operating rhythm",
  "content": "Reusable daily workflow.",
  "category": "productivity",
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
}
```

## Mode Playbooks

### 1) Partial Update

Entry: existing entities need localized changes only.
Exit: delta reported, no structural changes introduced.

1. Read current entity values.
2. Verify requested fields differ from current state — skip if no real change (no-op check).
3. If a task status is changing to `done`, queue a downstream dependents check with `dependsOn=["{task-id}"]`.
4. Update only the affected fields in the affected scope.
5. If step 3 applies, read downstream dependents and separate `ready_now`, `blocked_by_dependency`, and `none`.
6. Report exact delta, downstream result, and confirm unchanged structure.

Typical writes: `upsert track` with entity type `task`, `goal`, or `note`, and field-level changes only.

### 2) New Item Creation

Entry: exactly one new task, note, or goal is needed; existing structure remains valid.
Exit: one item created, linked references reported.

1. Confirm active workspace, destination project, and item type (`task` / `note` / `goal`).
2. Check for duplicate intent in active queue and backlog.
3. Create one item with owner, due date, and minimal context.
4. Report created item and any linked references (goal, parent task, dependencies).

Typical writes: one `upsert track` for a `task`, `goal`, or `note`.

### 3) Large-Scale Planning

Entry: multiple new items, a new goal structure, or a multi-step execution plan.
Exit: coordinated items created with ownership contracts and dependency links.

1. Run reuse check — scan workflows (`private` → `liked` → `public`).
2. Document why existing options do not fit.
3. Propose minimal viable multi-step structure with owners and dependencies.
4. Obtain approval if required (see [Write Safety](#write-safety-and-approval-gate)).
5. Materialize only after confirmation.

Typical writes: multiple `upsert track` operations, optional `upsert asset`.

### 4) Recovery and Backlog Hygiene

Entry: delivery is slowed by overload, dependency blocking, stalled queues, or noisy backlog.
Exit: smallest high-impact set of changes applied, load rebalanced.

1. Build active-task snapshot by assignee and status.
2. Identify overload, near-term deadline risks, and stale backlog segments.
3. Detect dependency hotspots (tasks with many dependents) and backlog noise (duplicates, outdated, low-signal).
4. Propose changes: reassign, resequence, merge, archive, or close.
5. Obtain approval for multi-entity restructuring (see [Write Safety](#write-safety-and-approval-gate)).
6. Execute the smallest high-impact set.

Recovery thresholds (calibrate per project):

| Signal                                        | Threshold                | Why                        | Action               |
| --------------------------------------------- | ------------------------ | -------------------------- | -------------------- |
| `todo` rising while `done_last_7d` is flat    | Trending over 1+ weeks   | Intake outpaces throughput | Slow down new intake |
| High `in_progress` count                      | > 2x team median         | Too much concurrent work   | Limit WIP            |
| Assignee active tasks                         | >= 2x team median        | Individual overload        | Reassign or defer    |
| Stale `todo` with no dependency or value link | > 30 days without update | Dead weight in backlog     | Archive or close     |

## Write Safety and Approval Gate

Before any write:

1. Confirm active workspace and target project scope.
2. Resolve entity and assignee IDs from current Epismo results (MCP references or CLI JSON output).
3. Classify change type: `partial update` / `new item creation` / `large-scale planning` / `recovery structural` / `destructive or visibility change`.

**Require explicit approval for:**

- Large-scale multi-step goal/task structures.
- Recovery proposals that reassign, resequence, or rewire dependencies across multiple items.
- Ambiguous write destination.
- Missing critical assumptions.
- Destructive or hard-to-reverse changes.

**Do not require approval for:**

- Small, explicit, reversible partial updates.
- Single-item creation with clear scope.

If the user has not answered a clarification question, do not write. Continue read-only analysis and report the pending decision.

For workflow release/update/deprecate, apply [Workflow Release — Approval Boundary](./workflow-release.md#approval-boundary).

## Operation Output

After every operation, return this structure. Prefer names and titles in user-facing output — share raw IDs only when requested.

1. **Situation** — what is active now.
2. **Delta** — what changed (created, updated, deleted) or why no write occurred.
3. **Evidence** — which tracks, assets, or external sources informed the decision.
4. **Risks / decisions** — open questions or items needing confirmation. If a task was marked `done`, note whether downstream tasks are now ready or still blocked.
5. **Next action** — the single smallest useful next step.

## Error Handling

| Error                                    | Action                                                                                                      |
| ---------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| `Payment Required: Insufficient credits` | Stop. Navigate user to [Credit Purchase](./credit-purchase.md).                                             |
| `Permission denied`                      | Re-check accessible projects and ownership scope.                                                           |
| `Unauthorized` / `403`                   | Verify MCP token or `EPISMO_TOKEN` (CLI), active workspace, and subscription context.                       |
| `Not Found` / `404`                      | Confirm entity ID exists. It may have been deleted or moved.                                                |
| Invalid workflow graph                   | Normalize IDs and rebuild as an acyclic dependency graph. Remove self-dependencies and circular references. |
| Rate limit / `429`                       | Wait and retry with backoff. Inform user if persistent.                                                     |
| Timeout                                  | Retry once. If persistent, reduce batch size or switch to sequential writes.                                |
