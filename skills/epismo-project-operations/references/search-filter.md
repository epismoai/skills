# Search & Filter

Query patterns, filter semantics, and entity relationship traversal for Epismo MCP.
Use when building filtered queues, finding goals/tasks/notes/workflows, or reasoning about dependencies.

## Quick Reference

| What you need            | How to query                                                                        |
| ------------------------ | ----------------------------------------------------------------------------------- |
| Tasks ready to work on   | `status=["todo"]` → verify each `dependsOn[]` item is `done`                        |
| Tasks in progress        | `status=["in_progress"]`                                                            |
| Tasks due soon           | `status=["backlog","todo","in_progress"]` + `dueDateTo={date}`                      |
| Blocked tasks            | `status=["backlog","todo","in_progress"]` → check each `dependsOn[]` for non-`done` |
| Recently completed tasks | `status=["done"]` + `doneAtFrom={now-7d}`                                           |
| Reusable workflows       | Search `visibility=["private"]` first → `like="liked"` → `visibility=["public"]`    |
| Tasks under a goal       | `goalId=["{goal-id}"]`                                                              |
| Downstream dependents    | `dependsOn=["{task-id}"]` — finds what unblocks when this task completes            |

## Scope Semantics

1. Project item search (`epismo_search_project_items`) accepts top-level `projects[]` to limit scope.
2. Omitting `projects[]` searches all accessible projects.
3. Workflow search (`epismo_search_workflows`) uses `filter.visibility[]`: `private` or `public`.
4. In `epismo_upsert_workflow`, `projects[]` is valid only when `visibility="private"`.
5. If `visibility` is omitted on workflow upsert, default is `private`.
6. Keep `query` compact: 2-6 domain keywords.

## Date/Time Semantics

1. `dueDateFrom` / `dueDateTo` — date-only, normalized to UTC midnight.
2. All other datetime filters (`updatedAtFrom/To`, `doneAtFrom/To`) — keep provided ISO-8601 precision.
3. `task.doneAt` is read-only (system-managed) but searchable via `doneAtFrom/To`.

## Supported Filter Keys

### Tasks (`epismo_search_project_items`, type="task")

`status[]`, `assignee[]`, `goalId[]`, `parentId[]`, `dependsOn[]`,
`dueDateFrom`, `dueDateTo`, `updatedAtFrom`, `updatedAtTo`, `doneAtFrom`, `doneAtTo`

### Goals (`epismo_search_project_items`, type="goal")

`status[]`, `progressMin`, `progressMax`,
`dueDateFrom`, `dueDateTo`, `updatedAtFrom`, `updatedAtTo`

### Notes (`epismo_search_project_items`, type="note")

`updatedAtFrom`, `updatedAtTo`

### Workflows (`epismo_search_workflows`)

`category[]`, `visibility[]`, `like`, `ownerId[]`,
`minLikeCount`, `minDownloadCount`, `updatedAtFrom`, `updatedAtTo`

## Entity Relationships

| Relationship                  | Link field                             | Use case                                    |
| ----------------------------- | -------------------------------------- | ------------------------------------------- |
| Goal → tasks                  | `task.goalId`                          | Find all tasks contributing to a goal       |
| Parent → child tasks          | `task.parentId`                        | Navigate task hierarchy                     |
| Task → upstream prerequisites | `task.dependsOn[]`                     | Check what must finish first                |
| Task → downstream dependents  | query `filter.dependsOn=["{task-id}"]` | Find what unblocks when this task completes |

## Derived Queue States

These are computed from entity data, not stored as status field values. Use them for queue analysis and recovery decisions.

| State                   | Definition                                              | How to compute                                                              |
| ----------------------- | ------------------------------------------------------- | --------------------------------------------------------------------------- |
| `ready_now`             | All `dependsOn` are `done`; task is `backlog` or `todo` | Filter `status=["backlog","todo"]`, then verify each prerequisite is `done` |
| `active`                | Currently being worked on                               | Tasks: `status=["in_progress"]`; Goals: `status=["on_track","at_risk"]`     |
| `blocked_by_dependency` | At least one prerequisite is not `done`                 | No direct filter — fetch `dependsOn[]` and check each status                |

## Filter Recipes

### Urgent — deadline-driven

- Tasks: `status=["backlog","todo","in_progress"]` + `dueDateTo={date}`
- Goals: `status=["not_started","on_track","at_risk"]` + `dueDateTo={date}`

### Goal execution — tasks under a goal

- `status=["todo","in_progress"]` + `goalId=["{goal-id}"]`

### Downstream dependents — what unblocks next

- `dependsOn=["{task-id}"]`

### Throughput — completed in last 7 days

- `status=["done"]` + `doneAtFrom={now-7d}` + `doneAtTo={now}`

### Release evidence — completed work in a period

- Tasks: `status=["done"]` + `doneAtFrom={iso8601}` + optional `projects=["{project-id}"]`
- Goals: `status=["completed"]` + `updatedAtFrom={iso8601}` + optional `projects=["{project-id}"]`
- Notes: `updatedAtFrom={iso8601}` + optional `projects=["{project-id}"]`

### Workflow reuse scan

- Liked workflows: `like="liked"`
- Private only: `visibility=["private"]`
- By category: `category=["{category}"]`

## Pagination

All search tools use page size `20`. Iterate `page=1, 2, 3...` and merge results for large sets.
