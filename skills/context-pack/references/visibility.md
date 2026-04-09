# Visibility & Sharing

Visibility settings, sharing, and approval rules for context packs.
For the same concepts applied to workflow packs, see [Workflow Pack — Visibility & Sharing](../../workflow-pack/references/visibility.md).
For content templates, see [Content Templates](../templates/content.md).
For search and discovery, see [Search & Discovery](./search.md).

## Private vs. Public

Every context pack has a `visibility` field. Choose based on who needs to read the pack.

| Visibility | Who can read                            | Who can find it                        | Typical use                                                  |
| ---------- | --------------------------------------- | -------------------------------------- | ------------------------------------------------------------ |
| `private`  | Owner and workspace members with access | Only via direct ID or workspace search | Working notes, team rules, handoffs, internal status reports |
| `public`   | Anyone on Epismo                        | Global search and community discovery  | Best-practice guides, open project norms, reusable templates |

**Default:** `private`. If `visibility` is omitted in an upsert payload, Epismo treats the pack as private.

### When to choose private

- The content contains project-specific details (track IDs, internal names, credentials references).
- The audience is only within the current workspace or team.
- The pack is a working draft not ready for external review.
- The content may change frequently.

### When to choose public

- The content is a reusable guide or pattern that benefits a broader audience.
- The project is open-source or community-facing.
- The user explicitly requests "publish" or "make this public".
- The content has been reviewed and is stable enough to represent the team externally.

**Never set public without explicit user approval.** See [Approval Boundary](#approval-boundary).

## Public Review Gate

Before publishing a context pack, confirm all four:

1. **External readability** — an outside reader can understand what the pack is for without workspace-specific background.
2. **Safe content boundary** — no credentials, internal-only URLs, private identifiers, or sensitive notes are exposed.
3. **Attribution quality** — source links, dates, versions, and references are included when they matter for trust or reuse.
4. **Discovery fit** — title, visibility, and category are deliberate and match how others would search for this pack.

If any item fails, keep the pack private and revise the content first.

## Category Reference

Set `category` on public packs to improve discoverability. On private packs it is optional but still useful for workspace-level filtering.

| Category       | When to use                                                        |
| -------------- | ------------------------------------------------------------------ |
| `productivity` | Workflow guides, working norms, session patterns, time management  |
| `programming`  | Code review checklists, architecture decisions, API design rules   |
| `design`       | Design system norms, component guidelines, UX decision records     |
| `marketing`    | Campaign rules, messaging frameworks, brand guidelines             |
| `operations`   | Runbooks, incident post-mortems, deployment norms, team agreements |
| `learning`     | Onboarding guides, tutorials, knowledge bases                      |
| `life`         | Personal knowledge, life planning, non-work contexts               |
| `` (empty)     | Cross-domain or uncategorized — use sparingly                      |

Choose the most specific category that fits. If two categories apply equally, prefer the one the intended audience would search in.

## Project Scoping

`projects[]` attaches a private pack to one or more workspace projects. This controls which project members can discover the pack in project-scoped searches.

**Rules:**

- `projects[]` is valid **only** when `visibility="private"`.
- For public packs, omit `projects[]` — public packs are workspace-level and globally discoverable regardless of project.
- A pack can belong to multiple projects simultaneously.
- Changing `projects[]` on an existing pack is a safe partial update — it does not change content or visibility.

## Sharing a Context Pack

After publishing a pack, deliver it to the intended audience via share URL.

### Share URL

The Epismo share URL format is:

```
https://epismo.ai/share/{token}
```

Obtain the share token from the upsert response or from the Epismo UI. The token resolves to a redirect at:

```
https://epismo.ai/hub/contexts/{pack-id}
```

**Resolving a share URL to an pack ID:** see [Epismo Basics — Resolving Share URLs](../../epismo-basics/SKILL.md#resolving-share-urls).

### Who Can Use a Share URL

| Pack visibility | Link recipient has workspace access | Can view via link |
| --------------- | ----------------------------------- | ----------------- |
| `private`       | Yes                                 | Yes               |
| `private`       | No                                  | No                |
| `public`        | Any                                 | Yes               |

### Liking a Public Pack

Recipients can like a public pack to bookmark it and signal quality:

```bash
# CLI
epismo pack like --id <id> --liked

# To un-like
epismo pack like --id <id> --no-liked
```

Liked packs appear in search results filtered by `like="liked"` and surface higher in community discovery.

## Approval Boundary

Some visibility and lifecycle actions require **explicit user approval** before writing. Do not proceed without it.

| Action                                                | Approval required? | Why                                               |
| ----------------------------------------------------- | ------------------ | ------------------------------------------------- |
| First-time public publication (`visibility="public"`) | **Yes**            | Irreversible — content becomes globally visible   |
| Changing existing pack from `private` to `public`     | **Yes**            | Visibility change may expose internal information |
| Deleting a pack (`delete pack`)                       | **Yes**            | Irreversible if not in trash/recovery             |
| Overwriting another owner's pack                      | **Yes**            | Affects content the user does not own             |
| Creating or updating a private pack                   | No                 | Low risk, reversible                              |
| Changing `projects[]` on an existing private pack     | No                 | Scoping change only                               |
| Liking or un-liking a pack                            | No                 | Reversible signal, no content change              |

**How to obtain approval:** state the intended action, the pack title, and the visibility setting. Wait for an explicit "yes", "go ahead", or equivalent confirmation before writing.

## Audience Targeting Guide

Use this table to decide visibility and distribution method based on who needs the pack.

| Audience                                     | Visibility               | Distribution method                                                            |
| -------------------------------------------- | ------------------------ | ------------------------------------------------------------------------------ |
| Just me, this session                        | `private`                | Read directly via `get pack` in the next session                               |
| My team in this workspace                    | `private` + `projects[]` | Share URL (team members have workspace access)                                 |
| A specific person (any tool)                 | `private` then share     | Share URL — recipient can view if they have access; use `public` if they don't |
| Everyone on Epismo                           | `public`                 | Share URL or community search                                                  |
| External audience (no Epismo account needed) | `public`                 | Share URL — no login required to view                                          |

If the recipient does not have workspace access and the content is safe to share openly, publish as `public`. If the content must stay private but needs to reach someone outside the workspace, discuss access provisioning with the user — this skill cannot grant workspace access.
