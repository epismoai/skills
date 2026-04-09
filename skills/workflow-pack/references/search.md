# Search & Discovery

Discovery and loading patterns for workflow packs.
For release and update decisions, see [Release](./release.md).
For visibility and sharing, see [Visibility & Sharing](./visibility.md).

This reference shows CLI forms. For surface conventions, see [Epismo Basics — Surface Conventions](../../epismo-basics/SKILL.md#surface-conventions).

## Operations

| Operation     | CLI command                          | Key flags                                        |
| ------------- | ------------------------------------ | ------------------------------------------------ |
| `search pack` | `epismo pack search --type workflow` | `--query <keywords>` `--filter '{...}'`          |
| `get pack`    | `epismo pack get --id <id>`          | `[--full]` `[--step-id <step-id-1>,<step-id-2>]` |
| `upsert pack` | `epismo pack upsert`                 | `--input @pack.json`                             |
| `delete pack` | `epismo pack delete --id <id>`       | —                                                |
| `like pack`   | `epismo pack like --id <id>`         | `--liked` / `--no-liked`                         |

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
5. In `upsert pack`, `projects[]` is valid only when `visibility="private"`.
6. If `visibility` is omitted on pack upsert, default is `private`.
7. Keep `query` compact: 2-6 domain keywords.

## Supported Filter Keys

### Workflows (`search pack`, type `workflow`)

`category[]`, `visibility[]`, `like`, `ownerId[]`,
`minLikeCount`, `minDownloadCount`, `updatedAtFrom`, `updatedAtTo`

## Loading a Workflow

Use the two-step scan-then-fetch pattern to avoid loading full workflow content before you know the candidate is relevant.

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
  "filter": { "visibility": ["private"] },
  "sort": "updatedAt:desc"
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
