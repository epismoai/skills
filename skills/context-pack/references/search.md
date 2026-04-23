# Search & Discovery

Discovery and loading patterns for context packs.
For content templates, see [Content Templates](../templates/content.md).
For visibility and sharing, see [Visibility & Sharing](./visibility.md).

This reference shows CLI forms. Derive MCP tool names mechanically: spaces → underscores.
Surface conventions are defined in [Context Pack](../SKILL.md#operations-context-packs).

## Surface Resolution

| Operation     | CLI command                          | Key flags                                                |
| ------------- | ------------------------------------ | -------------------------------------------------------- |
| `search pack` | `epismo pack search --type context`  | `--query <keywords>` `--filter '{...}'` `--project-ids`  |
| `get pack`    | `epismo pack get --id <id>`          | `[--full]` `[--block-id <block-id>]`                     |
| `like pack`   | `epismo pack like --id <id> --liked` | `--no-liked` to remove                                   |

## Quick Reference

| What you need                 | How to query                                        |
| ----------------------------- | --------------------------------------------------- |
| My private context packs      | `visibility=["private"]`                            |
| Packs I liked                 | `like="liked"`                                      |
| Public guides by topic        | `visibility=["public"]` + `query=<topic keywords>`  |
| Packs in a specific category  | `category=["productivity"]` (or other category)     |
| Recent packs (last 7 days)    | `updatedAtFrom=<now-7d>`                            |
| Packs by a specific owner     | `ownerId=["<owner-id>"]`                            |
| Popular public packs          | `visibility=["public"]` + `minLikeCount=5`          |
| Packs updated in a date range | `updatedAtFrom=<iso8601>` + `updatedAtTo=<iso8601>` |

## Scope Semantics

1. `workspace` is the top-level access boundary. All searches run within the active workspace.
2. `search pack --type context` returns both private (yours) and public packs unless filtered.
3. `filter.visibility=["private"]` returns only your private packs.
4. `filter.visibility=["public"]` returns all public packs discoverable from the active workspace.
5. `targets.projectIds=["<project-id>"]` narrows private search scope to that project. In CLI, pass `--project-ids <ids>`.
6. `targets.self` controls whether search includes packs that target the current user directly. In CLI, pass `--self false` to exclude them.
7. `query` runs a semantic / keyword search on title and content. Keep it to 2–6 domain keywords.
8. Omitting all filters returns the most recently updated packs in the active workspace.

## Supported Filter Keys

### Context packs (`search pack`, type `context`)

| Filter key         | Type              | Description                                                                                                  |
| ------------------ | ----------------- | ------------------------------------------------------------------------------------------------------------ |
| `visibility[]`     | enum              | `"private"` or `"public"`                                                                                    |
| `like`             | enum              | `"liked"` — returns only packs you have liked                                                                |
| `ownerId[]`        | string array      | Filter by owner user ID                                                                                      |
| `category[]`       | enum array        | One or more categories — see [Visibility & Sharing — Category Reference](./visibility.md#category-reference) |
| `minLikeCount`     | integer           | Minimum number of likes (useful for finding popular public guides)                                           |
| `minDownloadCount` | integer           | Minimum number of imports                                                                                    |
| `updatedAtFrom`    | ISO-8601 datetime | Lower bound on last-updated timestamp                                                                        |
| `updatedAtTo`      | ISO-8601 datetime | Upper bound on last-updated timestamp                                                                        |

`query` is a top-level search parameter, not a filter key. Combine it with filter keys to narrow results further.

## Loading Context from a Pack

Use the two-step scan-then-fetch pattern to avoid pulling full content from packs you don't need.

## Query vs Alias

Use `get pack --alias` first for alias-shaped targets.

- Good alias candidates: `@handle/name`, `weekly-handoff`, `frontend_notes`
- Short phrases can still be aliases: try alias first unless the input is obviously a search query
- Search-first inputs: `what context do I have for onboarding`, `find onboarding notes`, longer descriptive requests
- `get/read/open <target>` → try alias or ID first, then `search pack --query` if it misses
- For bare multi-word input, default to `search pack --query` rather than alias resolution.

### Step 1 — Scan titles

`search pack` always returns outline format (`id`, `title`, brief metadata) — never full content blocks.

```bash
epismo pack search --type context --query "auth refactor handoff" --filter '{"visibility":["private"]}'
```

Review the returned titles to identify the right pack.

### Step 2 — Fetch full content

```bash
# Full content (all blocks)
epismo pack get --id <pack-id> --full

# Single block only
epismo pack get --id <pack-id> --block-id <block-id>
```

`get pack` without `--full` returns outline only (`id`, `title`, `view`, `contentIndex`). Use `--full` to load all block content into the session.

**MCP equivalent:**

```
epismo_pack_get  # with id parameter; pass full=true for full content
```

### Loading into a new session

When resuming work in a new tool or session:

1. Search for the context pack by topic or date: `search pack --query "<topic>"`.
2. Identify the matching pack by title.
3. Fetch its full content: `get pack --id <id>`.
4. Read the opening block that explains what the pack is, then the block most relevant to the current task, such as `Next Steps`, `Summary`, or `Key Takeaways`.
5. Proceed from that block instead of re-reading the original conversation or source.

## Search Recipes

### Find my most recent context packs

```json
{
  "type": "context",
  "filter": { "visibility": ["private"] }
}
```

### Find handoffs addressed to me (by owner or title keyword)

```bash
epismo pack search --type context --query "handoff" --filter '{"visibility":["private"]}'
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

### Find packs updated during a specific week

```json
{
  "type": "context",
  "filter": {
    "updatedAtFrom": "2026-03-24T00:00:00Z",
    "updatedAtTo": "2026-03-31T23:59:59Z"
  }
}
```

### Find packs scoped to a project

```json
{
  "type": "context",
  "targets": {
    "projectIds": ["pj_123"]
  },
  "filter": {
    "visibility": ["private"]
  }
}
```

### Discover packs I've bookmarked

```json
{
  "type": "context",
  "filter": { "like": "liked" }
}
```

### Reuse scan before creating a new pack

Search private packs first, then liked, then public — to avoid duplicating something that already exists.

1. `visibility=["private"]` + `query=<topic>`
2. `like="liked"` + `query=<topic>`
3. `visibility=["public"]` + `query=<topic>`

If a close match is found, prefer `get pack` over creating a new one.

## Pagination

All `search pack` calls use page size 20. For large result sets, iterate `page=1, 2, 3…` and merge results.

```bash
epismo pack search --type context --filter '{"visibility":["public"]}' --page 2
```

Stop iterating when a page returns fewer than 20 results.
