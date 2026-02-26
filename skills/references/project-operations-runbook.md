# Project Operations Runbook

This runbook governs day-to-day operational writes primarily on tasks/goals/notes.
Use it when you need to choose an execution mode, plan changes, and decide write/approval boundaries.
For workflow release/update/deprecate approval boundaries, use `references/workflow-release.md`.
For query/filter semantics, use `references/search-filter.md`.
Status handling must follow entity-specific definitions from `search-filter.md` (`task` and `goal` statuses differ; `blocked` is queue state, not a status).

## When to Use

Use this runbook when you are executing operational work, especially before writes:

1. Partial update of existing items
2. New single-item creation
3. Large-scale planning and bulk materialization
4. Workload recovery and bottleneck mitigation

## Operation Modes

1. Partial update mode
    - Trigger: existing entities need localized changes (assignee/status/due date/priority).
    - Action: update existing items only; avoid structural redesign.
2. New item creation mode
    - Trigger: user needs one new task/note/goal and existing structure remains valid.
    - Action: create the single item with minimal linked changes.
3. Large-scale planning mode
    - Trigger: user requests bulk tasks or a new multi-step execution structure.
    - Action: design ownership and dependency contract before writes.
4. Recovery mode
    - Trigger: overload, stalled queue, dependency hotspots, or backlog drift.
    - Action: rebalance ownership, clean backlog noise, and resolve top blockers first.

## Planning Checks

Use this before structural changes:

1. Situation diagnosis
    - Pull current goals/tasks/notes/workflows first.
    - Distinguish already planned vs actively moving items.
2. Path selection
    - Choose one: partial update / new item creation / large-scale planning / recovery.
    - State why alternatives were not chosen.
3. Contract design
    - For each changed step, define owner, output, and true prerequisites.
    - Keep independent work parallel.
4. Materialization decision
    - Choose destination: tasks/goals/notes first; add private workflow when reusable structure is required.
5. Checkpoint design
    - Define the smallest useful next review point.

## Mode Playbooks

### 1) Partial Update

1. Read current entity value.
2. Verify requested fields differ from current state.
3. Update only existing entities in affected scope.
4. Report exact delta and unchanged structure.

### 2) New Item Creation

1. Confirm destination project and item type (`task`/`note`/`goal`).
2. Check for obvious duplicate intent in active queue/backlog.
3. Create one item with owner, due date, and minimal context.
4. Report created item and any linked references.

### 3) Large-Scale Planning

1. Run reuse check across project assets and workflows.
2. Document why existing options do not fit.
3. Propose minimal viable multi-step structure.
4. Materialize only after required confirmations.

### 4) Recovery

1. Build active-task counts by assignee.
2. Identify overload + near-term deadlines + stale backlog segments.
3. Detect dependency hotspots (tasks with many dependents) and backlog noise (duplicates, outdated todos, low-signal items).
4. Reassign, resequence, merge, archive, or close the smallest high-impact set.

Practical thresholds (adjust per project):

1. `todo` rising while `done_last_7d` is flat -> intake risk.
2. High `in_progress` -> WIP congestion.
3. `>= 2x` team-median active tasks -> overload signal.
4. Stale `todo` older than 30 days with no dependency/value link -> backlog cleanup candidate.

## Write Safety and Approval Gate

Before writes:

1. Confirm project scope.
2. Resolve entity/assignee IDs from MCP references.
3. Classify change type (`partial update` / `new item creation` / `large-scale planning` / `recovery structural` / `destructive or visibility expansion`).

Ask explicit approval only when:

1. Creating large-scale multi-step goal/task structures.
2. Recovery proposes reassignment, resequencing, or dependency rewiring across multiple items.
3. Write destination remains ambiguous.
4. Critical assumptions are missing.
5. Destructive or hard-to-reverse change is proposed.

If destination is ambiguous and the user has not answered the clarification question, do not write.
Continue read-only analysis and report the pending decision.

Do not block on approval for small, explicit, reversible partial updates and single-item creation.
For workflow release/update/deprecate, apply approval boundary from `workflow-release.md`.

## Operation Output Fields

After major operations, return:

1. Situation: what is active now.
2. Delta: what changed (or why no write).
3. Evidence used: which project/workflow/external sources informed the decision.
4. Risks/decisions: what needs confirmation.
5. Next action: the smallest useful step.

Prefer names/titles in user-facing output. Share raw IDs only when requested.

## Error Handling

1. `Payment Required: Insufficient credits`
    - Stop expansion and request billing confirmation.
2. `Permission denied`
    - Re-check accessible projects and ownership scope.
3. `Unauthorized` or `403`
    - Verify key/subscription context.
4. Invalid workflow graph
    - Normalize IDs and rebuild as an acyclic dependency graph.
