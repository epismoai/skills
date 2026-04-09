# Search & Discovery

Discovery and loading patterns for context assets.
For content templates, see [Content Templates](../templates/content.md).
For visibility and sharing, see [Visibility & Sharing](./visibility.md).

This reference shows CLI forms. For surface conventions, see [Epismo Basics — Surface Conventions](../../epismo-basics/SKILL.md#surface-conventions).

## Surface Resolution

| Operation | CLI command | Key flags |
| --------- | ----------- | --------- |
| `search asset` | `epismo asset search --type context` | `--query <keywords>` `--filter '{...}'` |
| `get asset` | `epismo asset get --id <id>` | `[--full]` `[--block-id <block-id>]` |
| `like asset` | `epismo asset like --id <id> --liked` | `--no-liked` to remove |

## Quick Reference

| What you need | How to query |
| ------------- | ------------ |
| My private context assets | `visibility=["private"]` |
| Assets I liked | `like="liked"` |
| Public guides by topic | `visibility=["public"]` + `query=<topic keywords>` |
| Assets in a specific category | `category=["productivity"]` (or other category) |
| Recent assets (last 7 days) | `updatedAtFrom=<now-7d>` |
| Assets by a specific owner | `ownerId=["<owner-id>"]` |
| Popular public assets | `visibility=["public"]` + `minLikeCount=5` |
| Assets updated in a date range | `updatedAtFrom=<iso8601>` + `updatedAtTo=<iso8601>` |

## Scope Semantics

1. `workspace` is the top-level access boundary. All searches run within the active workspace.
2. `search asset --type context` returns both private (yours) and public assets unless filtered.
3. `filter.visibility=["private"]` returns only your private assets.
4. `filter.visibility=["public"]` returns all public assets discoverable from the active workspace.
5. `filter.projects=["<project-id>"]` narrows results to private assets scoped to that project.
6. `query` runs a semantic / keyword search on title and content. Keep it to 2–6 domain keywords.
7. Omitting all filters returns the most recently updated assets in the active workspace.

## Supported Filter Keys

### Context assets (`search asset`, type `context`)

| Filter key | Type | Description |
| ---------- | ---- | ----------- |
| `visibility[]` | enum | `"private"` or `"public"` |
| `like` | enum | `"liked"` — returns only assets you have liked |
| `ownerId[]` | string array | Filter by owner user ID |
| `category[]` | enum array | One or more categories — see [Visibility & Sharing — Category Reference](./visibility.md#category-reference) |
| `minLikeCount` | integer | Minimum number of likes (useful for finding popular public guides) |
| `minDownloadCount` | integer | Minimum number of imports |
| `updatedAtFrom` | ISO-8601 datetime | Lower bound on last-updated timestamp |
| `updatedAtTo` | ISO-8601 datetime | Upper bound on last-updated timestamp |

`query` is a top-level search parameter, not a filter key. Combine it with filter keys to narrow results further.

## Loading Context from an Asset

Use the two-step scan-then-fetch pattern to avoid pulling full content from assets you don't need.

### Step 1 — Scan titles

`search asset` always returns outline format (`id`, `title`, brief metadata) — never full content blocks.

```bash
epismo asset search --type context --query "auth refactor handoff" --filter '{"visibility":["private"]}'
```

Review the returned titles to identify the right asset.

### Step 2 — Fetch full content

```bash
# Full content (all blocks)
epismo asset get --id <asset-id> --full

# Single block only
epismo asset get --id <asset-id> --block-id <block-id>
```

`get asset` without `--full` returns outline only (`id`, `title`, `view`, `contentIndex`). Use `--full` to load all block content into the session.

**MCP equivalent:**
```
epismo_asset_get  # with id parameter; pass full=true for full content
```

### Loading into a new session

When resuming work in a new tool or session:

1. Search for the context asset by topic or date: `search asset --query "<topic>"`.
2. Identify the matching asset by title.
3. Fetch its full content: `get asset --id <id>`.
4. Read the opening block that explains what the pack is, then the block most relevant to the current task, such as `Next Steps`, `Summary`, or `Key Takeaways`.
5. Proceed from that block instead of re-reading the original conversation or source.

## Search Recipes

### Find my most recent context packs

```json
{
  "type": "context",
  "filter": { "visibility": ["private"] },
  "sort": "updatedAt:desc"
}
```

### Find handoffs addressed to me (by owner or title keyword)

```bash
epismo asset search --type context --query "handoff" --filter '{"visibility":["private"]}'
```

Look for titles containing "Handoff" or your name in the results.

### Find popular public best-practice guides in a category

```json
{
  "type": "context",
  "query": "best practice",
  "filter": {
    "visibility": ["public"],
    "category": ["programming"],
    "minLikeCount": 3
  }
}
```

### Find assets updated during a specific week

```json
{
  "type": "context",
  "filter": {
    "updatedAtFrom": "2026-03-24T00:00:00Z",
    "updatedAtTo": "2026-03-31T23:59:59Z"
  }
}
```

### Find assets scoped to a project

```json
{
  "type": "context",
  "filter": {
    "visibility": ["private"],
    "projects": ["pj_123"]
  }
}
```

### Discover assets I've bookmarked

```json
{
  "type": "context",
  "filter": { "like": "liked" }
}
```

### Reuse scan before creating a new asset

See [Epismo Basics — Asset Reuse Scan Order](../../epismo-basics/SKILL.md#asset-reuse-scan-order).

## Pagination

See [Epismo Basics — Pagination](../../epismo-basics/SKILL.md#pagination).
