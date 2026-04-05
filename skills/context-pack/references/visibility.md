# Visibility & Sharing

Visibility settings, sharing, and approval rules for context assets.
For the same concepts applied to workflow assets, see [Workflow Hub — Visibility & Sharing](../../workflow-hub/references/visibility.md).
For content templates, see [Content Templates](../templates/content.md).
For search and discovery, see [Search & Discovery](./search.md).

## Private vs. Public

Every context asset has a `visibility` field. Choose based on who needs to read the asset.

| Visibility | Who can read | Who can find it | Typical use |
| ---------- | ------------ | --------------- | ----------- |
| `private` | Owner and workspace members with access | Only via direct ID or workspace search | Working notes, team rules, handoffs, internal status reports |
| `public` | Anyone on Epismo | Global search and community discovery | Best-practice guides, open project norms, reusable templates |

**Default:** `private`. If `visibility` is omitted in an upsert payload, Epismo treats the asset as private.

### When to choose private

- The content contains project-specific details (track IDs, internal names, credentials references).
- The audience is only within the current workspace or team.
- The asset is a working draft not ready for external review.
- The content may change frequently.

### When to choose public

- The content is a reusable guide or pattern that benefits a broader audience.
- The project is open-source or community-facing.
- The user explicitly requests "publish" or "make this public".
- The content has been reviewed and is stable enough to represent the team externally.

**Never set public without explicit user approval.** See [Approval Boundary](#approval-boundary).

## Public Review Gate

Before publishing a context asset, confirm all four:

1. **External readability** — an outside reader can understand what the pack is for without workspace-specific background.
2. **Safe content boundary** — no credentials, internal-only URLs, private identifiers, or sensitive notes are exposed.
3. **Attribution quality** — source links, dates, versions, and references are included when they matter for trust or reuse.
4. **Discovery fit** — title, visibility, and category are deliberate and match how others would search for this pack.

If any item fails, keep the asset private and revise the content first.

## Category Reference

Set `category` on public assets to improve discoverability. On private assets it is optional but still useful for workspace-level filtering.

| Category | When to use |
| -------- | ----------- |
| `productivity` | Workflow guides, working norms, session patterns, time management |
| `programming` | Code review checklists, architecture decisions, API design rules |
| `design` | Design system norms, component guidelines, UX decision records |
| `marketing` | Campaign rules, messaging frameworks, brand guidelines |
| `operations` | Runbooks, incident post-mortems, deployment norms, team agreements |
| `learning` | Onboarding guides, tutorials, knowledge bases |
| `life` | Personal knowledge, life planning, non-work contexts |
| `` (empty) | Cross-domain or uncategorized — use sparingly |

Choose the most specific category that fits. If two categories apply equally, prefer the one the intended audience would search in.

## Project Scoping

`projects[]` attaches a private asset to one or more workspace projects. This controls which project members can discover the asset in project-scoped searches.

**Rules:**

- `projects[]` is valid **only** when `visibility="private"`.
- For public assets, omit `projects[]` — public assets are workspace-level and globally discoverable regardless of project.
- An asset can belong to multiple projects simultaneously.
- Changing `projects[]` on an existing asset is a safe partial update — it does not change content or visibility.

## Sharing a Context Asset

After publishing an asset, deliver it to the intended audience via share URL.

### Share URL

The Epismo share URL format is:

```
https://epismo.ai/share/{token}
```

Obtain the share token from the upsert response or from the Epismo UI. The token resolves to a redirect at:

```
https://epismo.ai/hub/contexts/{asset-id}
```

**Resolving a share URL to an asset ID** (use any HTTP client):

```bash
curl -s -o /dev/null -w "%{redirect_url}" "https://epismo.ai/share/${TOKEN}"
# → https://epismo.ai/hub/contexts/{id}
```

Use the resolved `id` with `get asset`.

### Who Can Use a Share URL

| Asset visibility | Link recipient has workspace access | Can view via link |
| ---------------- | ----------------------------------- | ----------------- |
| `private` | Yes | Yes |
| `private` | No | No |
| `public` | Any | Yes |

### Liking a Public Asset

Recipients can like a public asset to bookmark it and signal quality:

```bash
# CLI
epismo asset like --id <id> --liked

# To un-like
epismo asset like --id <id> --no-liked
```

Liked assets appear in search results filtered by `like="liked"` and surface higher in community discovery.

## Approval Boundary

Some visibility and lifecycle actions require **explicit user approval** before writing. Do not proceed without it.

| Action | Approval required? | Why |
| ------ | ------------------ | --- |
| First-time public publication (`visibility="public"`) | **Yes** | Irreversible — content becomes globally visible |
| Changing existing asset from `private` to `public` | **Yes** | Visibility change may expose internal information |
| Deleting an asset (`delete asset`) | **Yes** | Irreversible if not in trash/recovery |
| Overwriting another owner's asset | **Yes** | Affects content the user does not own |
| Creating or updating a private asset | No | Low risk, reversible |
| Changing `projects[]` on an existing private asset | No | Scoping change only |
| Liking or un-liking an asset | No | Reversible signal, no content change |

**How to obtain approval:** state the intended action, the asset title, and the visibility setting. Wait for an explicit "yes", "go ahead", or equivalent confirmation before writing.

## Audience Targeting Guide

Use this table to decide visibility and distribution method based on who needs the asset.

| Audience | Visibility | Distribution method |
| -------- | ---------- | ------------------- |
| Just me, this session | `private` | Read directly via `get asset` in the next session |
| My team in this workspace | `private` + `projects[]` | Share URL (team members have workspace access) |
| A specific person (any tool) | `private` then share | Share URL — recipient can view if they have access; use `public` if they don't |
| Everyone on Epismo | `public` | Share URL or community search |
| External audience (no Epismo account needed) | `public` | Share URL — no login required to view |

If the recipient does not have workspace access and the content is safe to share openly, publish as `public`. If the content must stay private but needs to reach someone outside the workspace, discuss access provisioning with the user — this skill cannot grant workspace access.
