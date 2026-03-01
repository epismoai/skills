# Search Filter

This document defines query patterns, filter semantics, and entity relationship traversal.
Use it when you need to quickly find the right goals/tasks/notes/workflows and build filtered queues.

## When to Use

Use this guide when you need to:

1. Find the right entities quickly (search patterns).
2. Apply correct filter keys and date semantics.
3. Reason about goal/task/dependency relationships.

## Scope Semantics

1. Project item (task/note/goal) search accepts top-level `projects[]`.
2. Omitting `projects[]` searches all accessible projects.
3. `epismo_search_workflows.filter.visibility[]` supports `private` and `public`.
4. In `epismo_upsert_workflow`, `projects[]` is valid only when `visibility="private"`.
5. If `visibility` is omitted on workflow upsert, default is `private`.
6. If you use `query`, keep it compact with domain keywords (for example, 2-6 words).

## Date/Time Semantics

1. `dueDateFrom/To` is date-only and normalized to UTC midnight.
2. All other datetime filters (`updatedAtFrom/To`) keep provided ISO-8601 precision.
3. `task.doneAt` is read-only (system-managed) but searchable via `doneAtFrom/To`.

## Supported Filter Keys

1. `epismo_search_project_items` (type="task") `.filter`
   - `status[]`, `assignee[]`, `goalId[]`, `parentId[]`, `dependsOn[]`
   - `dueDateFrom`, `dueDateTo`, `updatedAtFrom`, `updatedAtTo`, `doneAtFrom`, `doneAtTo`
2. `epismo_search_project_items` (type="goal") `.filter`
   - `status[]`, `progressMin`, `progressMax`
   - `dueDateFrom`, `dueDateTo`, `updatedAtFrom`, `updatedAtTo`
3. `epismo_search_project_items` (type="note") `.filter`
   - `updatedAtFrom`, `updatedAtTo`
4. `epismo_search_workflows.filter`
   - `category[]`, `visibility[]`, `like`, `ownerId[]`
   - `minLikeCount`, `minDownloadCount`, `updatedAtFrom`, `updatedAtTo`

## Relationship Map

1. Goal -> tasks
   - Link: `task.goalId`
2. Parent task -> children
   - Link: `task.parentId`
3. Task -> upstream prerequisites
   - Link: `task.dependsOn[]`
4. Task -> downstream dependents
   - Query dependents with `filter.dependsOn=[task-id]`

Operational queues:

1. `ready_now`: dependencies complete
2. `in_progress`: active and not blocked
3. `blocked_by_dependency`: at least one prerequisite incomplete

## Filter Recipes

1. Active queue
   - tasks: `status=["todo","in_progress"]`
   - goals: `status=["on_track","at_risk"]`
2. Urgent queue
   - tasks: `status=["backlog","todo","in_progress"]` + `dueDateTo=<date>`
   - goals: `status=["not_started","on_track","at_risk"]` + `dueDateTo=<date>`
3. Goal execution queue
   - tasks: `status=["todo","in_progress"]` + `goalId=["<goal-id>"]`
4. Dependency-risk queue
   - tasks: `dependsOn=["<task-id>"]`
5. Liked workflow reuse
   - workflows: `like="liked"`
6. Private-only workflow scan
   - workflows: `visibility=["private"]`
7. Release evidence quick scan
   - tasks: `status=["done"]` + `doneAtFrom=<iso8601>` + optional `projects=["<project-id>"]`
   - goals: `status=["completed"]` + `updatedAtFrom=<iso8601>` + optional `projects=["<project-id>"]`
   - notes: `updatedAtFrom=<iso8601>` + optional `projects=["<project-id>"]`
8. Throughput (`done_last_7d`) quick scan
   - tasks: `status=["done"]` + `doneAtFrom=<now-7d>` + `doneAtTo=<now>`

## Pagination

All search tools use page size `20`.
Iterate `page=1,2,3...` and merge results when sets are large.
