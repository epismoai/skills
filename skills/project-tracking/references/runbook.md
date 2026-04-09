# Runbook

Governs day-to-day operational writes on tasks and goals.
For workflow release/update/deprecate, use [Workflow Release](../../workflow-pack/references/release.md).
For query/filter semantics, use [Search & Filter](./search.md).
Surface conventions from [Epismo Basics](../../epismo-basics/SKILL.md).
Tracks hold project execution state; packs hold reusable stock content such as workflows — see [Workflow Pack](../../workflow-pack/SKILL.md).

Status values must match entity-specific definitions. `blocked_by_dependency` is a derived queue state, not a status field. Scope semantics and status definitions live in [Search & Filter](./search.md).

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

1. **Diagnose** — run `search track` for current goals and tasks, then scan reusable workflows through [Workflow Pack — Search & Discovery](../../workflow-pack/references/search.md). Separate planned items from actively-moving items.
   → Output: current state summary.
2. **Select mode** — choose one mode from the selector above. State why alternatives were rejected.
   → Output: chosen mode + one-line rationale.
3. **Design contracts** — for each planned change, define owner, expected output, and true prerequisites. Keep independent work parallel.
   → Output: ownership and dependency map.
4. **Confirm destination** — write to tracks first. If reusable structure is clearly needed, hand off workflow authoring to [Workflow Pack](../../workflow-pack/SKILL.md).
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

## Mode Playbooks

### 1) Partial Update

Entry: existing entities need localized changes only.
Exit: delta reported, no structural changes introduced.

Readiness checks:

- Target entities identified from current results.
- Requested change is real, not a no-op.
- Project scope confirmed.
- If task status is changing to `done`, downstream dependents check is queued.

1. Read current entity values.
2. Verify requested fields differ from current state — skip if no real change (no-op check).
3. If a task status is changing to `done`, queue a downstream dependents check with `dependsOn=["<task-id>"]`.
4. Update only the affected fields in the affected scope.
5. If step 3 applies, read downstream dependents and separate `ready_now`, `blocked_by_dependency`, and `none`.
6. Report exact delta, downstream result, and confirm unchanged structure.

Typical writes: `upsert track` with entity type `task` or `goal`, and field-level changes only.

### 2) New Item Creation

Entry: exactly one new task or goal is needed; existing structure remains valid.
Exit: one item created, linked references reported.

Readiness checks:

- Destination project and item type (`task` / `goal`) confirmed.
- Duplicate check completed against active queue and backlog.
- Project scope confirmed.

1. Confirm active workspace, destination project, and item type (`task` / `goal`).
2. Check for duplicate intent in active queue and backlog.
3. Create one item with owner, due date, and minimal context.
4. Report created item and any linked references (goal, parent task, dependencies).

Typical writes: one `upsert track` for a `task` or `goal`.

### 3) Large-Scale Planning

Entry: multiple new items, a new goal structure, or a multi-step execution plan.
Exit: coordinated items created with ownership contracts and dependency links.

Readiness checks:

- Current goals and tasks reviewed.
- Reuse check completed against workflows (`private` → `liked` → `public`).
- Why reuse is insufficient is explicit.
- Project scope confirmed.

1. Run reuse check through [Workflow Pack — Search & Discovery](../../workflow-pack/references/search.md#reuse-scan).
2. Document why existing options do not fit.
3. Propose minimal viable multi-step structure with owners and dependencies.
4. Obtain approval if required (see [Write Safety](#write-safety-and-approval-gate)).
5. Materialize only after confirmation.

Typical writes: multiple `upsert track` operations. If the result should become a reusable workflow pack, hand off to [Workflow Pack](../../workflow-pack/SKILL.md).

### 4) Recovery and Backlog Hygiene

Entry: delivery is slowed by overload, dependency blocking, stalled queues, or noisy backlog.
Exit: smallest high-impact set of changes applied, load rebalanced.

Readiness checks:

- Recovery trigger identified.
- Project scope confirmed.
- Active queue snapshot captured (`backlog`, `todo`, `in_progress`, `blocked`).
- Assignee load reviewed.
- Dependency hotspots identified.
- Backlog hygiene reviewed for stale, duplicate, and low-signal items.

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
2. Resolve entity and assignee IDs from current Epismo results.
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

For workflow release/update/deprecate, apply [Workflow Release — Approval Boundary](../../workflow-pack/references/release.md#approval-boundary).

## Operation Output

After every operation, return this structure. Prefer names and titles in user-facing output — share raw IDs only when requested.

1. **Situation** — what is active now.
2. **Delta** — what changed (created, updated, deleted) or why no write occurred.
3. **Evidence** — which tracks, packs, or external sources informed the decision.
4. **Risks / decisions** — open questions or items needing confirmation. If a task was marked `done`, note whether downstream tasks are now ready or still blocked.
5. **Next action** — the single smallest useful next step.

## Error Handling

For common errors (auth, credits, rate limits, timeouts), see [Epismo — Error Handling](../../epismo-basics/SKILL.md#error-handling).

| Error                                    | Action                                                                          |
| ---------------------------------------- | ------------------------------------------------------------------------------- |
| `Payment Required: Insufficient credits` | Stop. See [Credit Purchase](../../epismo-basics/references/credit-purchase.md). |
