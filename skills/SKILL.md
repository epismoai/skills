---
name: epismo-project-operations
description: "Cover end-to-end day-to-day project operations in Epismo MCP. Use for intake, prioritization, planning, coordination, risk handling, handoff, and workflow release based on current project and workflow state."
---

# Project Operations

Use this skill for end-to-end Epismo MCP operations, including day-to-day coordination and reusable workflow release.

Operating principle: **check current state -> gather evidence -> apply the smallest useful change; for release, pass quality gate first and keep review trace outside published content**

## Use This Skill When

Use this skill when the request includes:

1. Ongoing coordination, triage, or re-planning.
2. Converting ideas/docs/articles into actionable tasks/workflows.
3. Reusing existing workflows (`private`/`liked`/`public`) in active project work.
4. Defining AI/human ownership with tracked progress and accountability.
5. Release-target decisions (`private`/`public`) and workflow publish/update/deprecate operations.
6. Quality-gate review and release handoff decisions.

## Routing Rule

1. Identify the user's current phase from request + current state.
2. Run only the required step(s) for that phase and adjacent blockers.
3. Do not force all six steps when one focused update is sufficient.
4. If multiple steps apply, start from the earliest blocked step.

## 6-Step Operations Flow

1. Intake
    - Confirm desired outcome, constraints, timeline, and target project/workflow scope.
2. Plan
    - Use [Project Operations Runbook](./references/project-operations-runbook.md) planning checks.
    - Choose the smallest mode that satisfies the request.
3. Discovery
    - Reuse workflows before create: scan `private` -> `liked` -> `public`.
    - Use [Search and Filter](./references/search-filter.md) for fast retrieval/filtering (queues, status/date/dependency views).
    - Use [Workflow Discovery](./templates/workflow-discovery.md) only when you must compare candidates and define keep/modify/add adaptation.
4. Coordination
    - Operate by mode (partial update / new item creation / large-scale planning / recovery) using [Project Operations Runbook](./references/project-operations-runbook.md) for day-to-day task/goal/note changes.
    - Design assignment and task contracts using [AI Delegation](./references/ai-delegation.md).
    - Use [Operating Templates](./templates/operating-templates.md) for write-ready structure.
5. Risk
    - Evaluate bottlenecks and dependency risk using [Project Operations Runbook](./references/project-operations-runbook.md).
    - Include backlog hygiene checks (stale items, duplicate intents, low-signal queue noise) during recovery decisions.
    - Apply [Workflow Quality](./references/workflow-quality.md) when reusable workflow intent exists.
6. Handoff
    - For normal operations, record lifecycle delta with [Operating Templates](./templates/operating-templates.md).
    - For release intent, run [Workflow Release](./references/workflow-release.md) and report decision traceability in the response.

## Simple Mode Names

1. Partial update
    - Update existing items only (e.g., assignee/status/due date) without structural redesign.
2. New item creation
    - Create a single new task/note/goal with minimal side effects.
3. Large-scale planning
    - Design and materialize a multi-step or bulk structure with ownership and dependencies.
4. Recovery
    - Existing plan needs major correction due to stall/overload/backlog drift.

## Write Safety and Approval

1. Task/note/goal writes must use confirmed project scope.
2. If destination is unclear, ask one focused question before writing.
3. If the question is unanswered, do not write. Continue read-only analysis and report what is pending.
4. Never guess entity IDs or assignee IDs.
5. Workflow writes:
    - `visibility="private"`: set confirmed `projects`.
    - `visibility="public"`: omit `projects`.
6. Approval boundaries:
    - General/non-release writes: [Project Operations Runbook](./references/project-operations-runbook.md)
    - Workflow release/update/deprecate: [Workflow Release](./references/workflow-release.md)
