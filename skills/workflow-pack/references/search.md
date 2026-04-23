# Search & Discovery

Discovery and loading patterns for workflow packs.
For release and update decisions, see [Release](./release.md).
For visibility and sharing, see [Visibility & Sharing](./visibility.md).

This reference shows CLI forms. For surface conventions, see [Epismo Basics — Surface Conventions](../../epismo-basics/SKILL.md#surface-conventions).

## Operations

| Operation      | CLI command                          | Key flags                                        |
| -------------- | ------------------------------------ | ------------------------------------------------ |
| `search pack`  | `epismo pack search --type workflow` | `--query <keywords>` `--filter '{...}'` `--project-ids` |
| `get pack`     | `epismo pack get --id <id>`          | `[--full]` `[--step-id <step-id-1>,<step-id-2>]` |
| `create pack`  | `epismo pack create`                 | `--input @pack.json` or `--project-ids <ids>`    |
| `update pack`  | `epismo pack update --id <id>`       | `--input @changes.json` or `--project-ids <ids>` |
| `delete pack`  | `epismo pack delete --id <id>`       | —                                                |
| `like pack`    | `epismo pack like --id <id>`         | `--liked` / `--no-liked`                         |

## Quick Reference

| What you need              | How to query                                       |
| -------------------------- | -------------------------------------------------- |
| My private workflows       | `visibility=["private"]`                           |
| Workflows I liked          | `like="liked"`                                     |
| Public workflows by topic  | `visibility=["public"]` + `query=<topic keywords>` |
| Workflows in a category    | `category=["programming"]` (or other category)     |
| Popular public workflows   | `visibility=["public"]` + `minLikeCount=5`         |
| Recently updated workflows | `updatedAtFrom=<iso8601>`                          |

## Scope Semantics

1. `workspace` is the top-level access boundary for CLI and MCP calls.
2. `search pack --type workflow` returns both private and public workflows unless filtered.
3. `filter.visibility=["private"]` returns only private workflows in the active workspace.
4. `filter.visibility=["public"]` returns globally discoverable public workflows.
5. In `create pack` and `update pack`, `targets.projectIds[]` is valid only when `visibility="private"`. In CLI, pass `--project-ids <ids>`.
6. `targets.self` controls whether search includes workflows that target the current user directly. In CLI, pass `--self false` to exclude them.
7. If `visibility` is omitted on pack create/update, default is `private`.
8. Keep `query` compact: 2-6 domain keywords.

## Supported Filter Keys

### Workflows (`search pack`, type `workflow`)

`category[]`, `visibility[]`, `like`, `ownerId[]`,
`minLikeCount`, `minDownloadCount`, `updatedAtFrom`, `updatedAtTo`

## Loading a Workflow

Use the two-step scan-then-fetch pattern to avoid loading full workflow content before you know the candidate is relevant.

## Query vs Alias

Use `get pack --alias` first for alias-shaped targets.

- Good alias candidates: `@handle/name`, `prd-review`, `incident_postmortem`
- Short phrases can still be aliases: try alias first unless the input is obviously a search query
- Search-first inputs: `what workflows do I have for release`, `find release workflow`, longer descriptive requests
- `get/read/open <target>` → try alias or ID first, then `search pack --query` if it misses
- For bare multi-word input, default to `search pack --query` rather than alias resolution.

### Step 1 — Scan titles

```bash
epismo pack search --type workflow --query "code review" --filter '{"visibility":["private"]}'
```

Review returned titles first.

### Step 2 — Fetch full content or selected steps

```bash
# Full workflow
epismo pack get --id <pack-id> --full

# Selected steps only
epismo pack get --id <pack-id> --step-id <step-id-1>,<step-id-2>
```

## Reuse Scan

Search in this order to avoid rebuilding something that already exists — see [Epismo Basics — Pack Reuse Scan Order](../../epismo-basics/SKILL.md#pack-reuse-scan-order).

## Search Recipes

### Find my reusable private workflows

```json
{
  "type": "workflow",
  "filter": { "visibility": ["private"] }
}
```

### Find public workflows in a category

```json
{
  "type": "workflow",
  "query": "deployment",
  "filter": {
    "visibility": ["public"],
    "category": ["operations"]
  }
}
```

### Find workflows I bookmarked

```json
{
  "type": "workflow",
  "filter": { "like": "liked" }
}
```
