# Search & Filter

Query patterns, filter semantics, and entity relationship traversal for tracks and workflow assets.

For surface conventions, selective fetch pattern, pagination, and asset reuse scan order, see [Epismo Basics](../../epismo-basics/SKILL.md).

## Operations (tracks and workflow assets)

| Operation | CLI command | Key flags |
| --------- | ----------- | --------- |
| `search track` | `epismo track search` | `--type task\|goal` `--filter '{...}'` |
| `get track` | `epismo track get` | `--id <id>` |
| `upsert track` | `epismo track upsert` | `--input @item.json` |
| `delete track` | `epismo track delete` | `--id <id>` |
| `search asset` | `epismo asset search` | `--type workflow` `--filter '{...}'` |
| `get asset` | `epismo asset get` | `--id <id>` `[--full]` `[--block-id <id>]` `[--step-id <ids>]` |
| `upsert asset` | `epismo asset upsert` | `--type workflow` `--input @asset.json` |
| `delete asset` | `epismo asset delete` | `--id <id>` |
| `import asset` | `epismo asset import` | `--id <id>` `--project-id <project-id>` `[--item-ids <item-ids>]` |
| `like asset` | `epismo asset like` | `--id <id>` `--liked` / `--no-liked` |

## Quick Reference

| What you need                 | How to query                                                                            |
| ----------------------------- | --------------------------------------------------------------------------------------- |
| Tasks ready to work on        | `status=["todo"]` → verify each `dependsOn[]` item is `done`                            |
| Tasks in progress             | `status=["in_progress"]`                                                                |
| Tasks due soon                | `status=["backlog","todo","in_progress"]` + `dueDateTo=<date>`                          |
| Blocked tasks                 | `status=["backlog","todo","in_progress"]` → check each `dependsOn[]` for non-`done`     |
| Recently completed tasks      | `status=["done"]` + `doneAtFrom=<now-7d>`                                               |
| Reusable workflows      | Search `visibility=["private"]` first → `like="liked"` → `visibility=["public"]`        |
| Tasks under a goal            | `goalId=["<goal-id>"]`                                                                  |
| Downstream dependents         | `dependsOn=["<task-id>"]` — finds what unblocks when this task completes                |
| Post-completion unblock check | Set task to `done` → query `dependsOn=["<task-id>"]` → re-check remaining prerequisites |

## Scope Semantics

1. `workspace` is the top-level access boundary for CLI and MCP calls.
2. Each workspace can contain multiple projects.
3. `search track` accepts top-level `projects[]` to limit scope inside the active workspace.
4. Omitting `projects[]` searches all accessible projects in the active workspace.
5. `search track` requires `type`: `task` or `goal`.
6. `search asset` uses `--type workflow`; filter with `filter.visibility[]`: `private` or `public`.
7. In `upsert asset`, `projects[]` is valid only when `visibility="private"`.
8. If `visibility` is omitted on asset upsert, default is `private`.
9. Keep `query` compact: 2-6 domain keywords.

## Entity Reference

### Task and Goal statuses

| Entity | Valid statuses                                                 |
| ------ | -------------------------------------------------------------- |
| Task   | `backlog`, `todo`, `in_progress`, `done`                       |
| Goal   | `not_started`, `on_track`, `at_risk`, `postponed`, `completed` |

`blocked_by_dependency` is a derived queue state, not a status field.

## Date/Time Semantics

1. `dueDateFrom` / `dueDateTo` — date-only, normalized to UTC midnight.
2. All other datetime filters (`updatedAtFrom/To`, `doneAtFrom/To`) — keep provided ISO-8601 precision.
3. `task.doneAt` is read-only (system-managed) but searchable via `doneAtFrom/To`.

## Supported Filter Keys

### Tasks (`search track`, type `task`)

`status[]`, `assignee[]`, `goalId[]`, `parentId[]`, `dependsOn[]`,
`dueDateFrom`, `dueDateTo`, `updatedAtFrom`, `updatedAtTo`, `doneAtFrom`, `doneAtTo`

### Goals (`search track`, type `goal`)

`status[]`, `progressMin`, `progressMax`,
`dueDateFrom`, `dueDateTo`, `updatedAtFrom`, `updatedAtTo`

### Workflows (`search asset`, type `workflow`)

`category[]`, `visibility[]`, `like`, `ownerId[]`,
`minLikeCount`, `minDownloadCount`, `updatedAtFrom`, `updatedAtTo`

## Entity Relationships

| Relationship                  | Link field                             | Use case                                    |
| ----------------------------- | -------------------------------------- | ------------------------------------------- |
| Goal → tasks                  | `task.goalId`                          | Find all tasks contributing to a goal       |
| Parent → child tasks          | `task.parentId`                        | Navigate task hierarchy                     |
| Task → upstream prerequisites | `task.dependsOn[]`                     | Check what must finish first                |
| Task → downstream dependents  | query `filter.dependsOn=["<task-id>"]` | Find what unblocks when this task completes |

## Derived Queue States

These are computed from entity data, not stored as status field values. Use them for queue analysis and recovery decisions.

| State                   | Definition                                              | How to compute                                                              |
| ----------------------- | ------------------------------------------------------- | --------------------------------------------------------------------------- |
| `ready_now`             | All `dependsOn` are `done`; task is `backlog` or `todo` | Filter `status=["backlog","todo"]`, then verify each prerequisite is `done` |
| `active`                | Currently being worked on                               | Tasks: `status=["in_progress"]`; Goals: `status=["on_track","at_risk"]`     |
| `blocked_by_dependency` | At least one prerequisite is not `done`                 | No direct filter — fetch `dependsOn[]` and check each status                |

## Filter Recipes

### Urgent — deadline-driven

- Tasks: `status=["backlog","todo","in_progress"]` + `dueDateTo=<date>`
- Goals: `status=["not_started","on_track","at_risk"]` + `dueDateTo=<date>`

### Goal execution — tasks under a goal

- `status=["todo","in_progress"]` + `goalId=["<goal-id>"]`

### Downstream dependents — what unblocks next

- `dependsOn=["<task-id>"]`

### Post-completion unblock check

- `dependsOn=["<task-id>"]`
- Re-check every prerequisite in each returned task's `dependsOn[]`
- Separate `ready_now` from `blocked_by_dependency`
- Report both the newly unblocked tasks and the tasks that remain blocked

### Throughput — completed in last 7 days

- `status=["done"]` + `doneAtFrom=<now-7d>` + `doneAtTo=<now>`

### Release evidence — completed work in a period

- Tasks: `status=["done"]` + `doneAtFrom=<iso8601>` + optional `projects=["<project-id>"]`
- Goals: `status=["completed"]` + `updatedAtFrom=<iso8601>` + optional `projects=["<project-id>"]`

### Workflow asset reuse scan

Search in this order — see [Epismo — Asset Reuse Scan Order](../../epismo-basics/SKILL.md#asset-reuse-scan-order) for the full pattern.

- Liked workflows: `like="liked"`
- Liked + specific category: `like="liked"` + `category=["<category>"]`
- Private only: `visibility=["private"]`
- By category: `category=["<category>"]`
