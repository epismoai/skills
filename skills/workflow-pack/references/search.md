# Search & Discovery

Discovery and loading patterns for workflow packs.
For release and update decisions, see [Release](./release.md).
For visibility and sharing, see [Visibility & Sharing](./visibility.md).

This reference shows terminal CLI forms (`epismo ...`). For MCP naming and parameters, see [Epismo Basics — Surface Conventions](../../epismo-basics/SKILL.md#surface-conventions).

## Operations

| Operation     | CLI command                          | Key flags                                               |
| ------------- | ------------------------------------ | ------------------------------------------------------- |
| `search pack` | `epismo pack search --type workflow` | `--query <keywords>` `--filter '{...}'` `--projects` `--search-mode` |
| `get pack`    | `epismo pack get <id>`               | `[--full]` `[--step-id <step-id-1>,<step-id-2>]`        |
| `create pack` | `epismo pack create`                 | `--input @pack.json` or `--projects <ids>`              |
| `update pack` | `epismo pack update <id>`            | `--input @changes.json` or `--projects <ids>`           |
| `delete pack` | `epismo pack delete <id>`            | —                                                       |
| `like pack`   | `epismo pack like <id>`              | `--liked` / `--no-liked`                                |

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
5. In `create pack` and `update pack`, `scope` and `sharedWith` apply only when `visibility="private"`. Use `scope: { type: "personal" }` or `scope: { type: "projects", ids: [...] }`, plus optional `sharedWith: { userIds, emails }`. In CLI, pass `--personal`, `--projects <id...>`, or `--share-with <userIdOrEmail...>`.
6. In search, pass `scopes:[{type:"personal"}, {type:"projects", ids:[...]}]` or use CLI `--personal --projects <ids>`.
7. If `visibility` is omitted on pack create/update, default is `private`.
8. Keep `query` compact: 2-6 domain keywords.
9. `searchMode` defaults to `keyword`; pass `semantic` (CLI `--search-mode semantic`) to add vector similarity for intent-described queries. See [Epismo Basics — Search Ranking Mode](../../epismo-basics/SKILL.md#search-ranking-mode).

## Supported Filter Keys

### Workflows (`search pack`, type `workflow`)

`category[]`, `visibility[]`, `like`, `ownerId[]`,
`minLikeCount`, `minSuccessCount`, `updatedAtFrom`, `updatedAtTo`

## Loading a Workflow

Use the two-step scan-then-fetch pattern to avoid loading full workflow content before you know the candidate is relevant.

## Query vs Alias

Use `get pack @<alias>` first for alias-shaped targets.

- Good alias candidates: `@<alias>`, `@<handle>/<alias>`, `prd-review`, `incident_postmortem`
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
epismo pack get <pack-id> --full

# Selected steps only
epismo pack get <pack-id> --step-id <step-id-1>,<step-id-2>
```

## Reuse Scan

Search private and public scopes for the same topic, then compare candidates before fetching full content — see [Epismo Basics — Pack Reuse Scan](../../epismo-basics/SKILL.md#pack-reuse-scan).

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
