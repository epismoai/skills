# Visibility & Sharing

Visibility settings, sharing, and approval rules for workflow assets.
For the same concepts applied to context assets, see [Context Pack — Visibility & Sharing](../../context-pack/references/visibility.md).
For workflow search and loading patterns, see [Search & Discovery](./search.md).

## Private vs. Public

Every workflow asset has a `visibility` field. Choose based on who the workflow is intended for.

| Visibility | Who can read | Who can find it | Typical use |
| ---------- | ------------ | --------------- | ----------- |
| `private` | Owner and workspace members with access | Only via direct ID or workspace search | Team-specific workflows, drafts, work in progress |
| `public` | Anyone on Epismo | Global search and community discovery | Proven, generalizable patterns ready for community reuse |

**Default:** `private`. If `visibility` is omitted in an upsert payload, Epismo treats the asset as private.

### When to choose private

- The workflow references project-specific names, IDs, or internal constraints.
- The audience is only within the current workspace or team.
- The pattern has not been validated with real execution evidence.
- The quality gate has open items — see [Quality Gate](./quality.md).

### When to choose public

- The pattern is proven and generalizable (no project-specific residue).
- The structure is stable enough for community reuse.
- The user explicitly requests "publish" or "make this public".
- All 8 quality criteria pass — see [Quality Gate](./quality.md).

**Never set public without explicit user approval.** See [Approval Boundary](#approval-boundary).

## Category Reference

Set `category` on public assets to improve discoverability. On private assets it is optional but still useful for workspace-level filtering.

| Category | When to use |
| -------- | ----------- |
| `productivity` | Daily operating rhythms, meeting flows, time management |
| `programming` | Code review, deployment, testing, development workflows |
| `design` | Design system processes, component reviews, UX research flows |
| `marketing` | Campaign planning, content creation, launch workflows |
| `operations` | Incident response, onboarding, deployment pipelines, team agreements |
| `learning` | Onboarding guides, skill-building plans, training workflows |
| `life` | Personal productivity, non-work routines |
| `` (empty) | Cross-domain or uncategorized — use sparingly |

Choose the most specific category that fits. If two apply equally, prefer the one the intended audience would search in.

## Project Scoping

`projects[]` attaches a private workflow asset to one or more workspace projects. This controls which project members can discover the asset in project-scoped searches.

**Rules:**

- `projects[]` is valid **only** when `visibility="private"`.
- For public assets, omit `projects[]` — public assets are workspace-level and globally discoverable.
- An asset can belong to multiple projects simultaneously.
- Changing `projects[]` on an existing asset is a safe partial update — it does not change content or visibility.

## Sharing a Workflow Asset

After publishing, deliver to the intended audience via share URL.

### Share URL

Obtain the share token from the upsert response or the Epismo UI. Resolve it to an asset ID:

```bash
curl -s -o /dev/null -w "%{redirect_url}" "https://epismo.ai/share/${TOKEN}"
# → https://epismo.ai/hub/workflows/{id}
```

See [Epismo Basics — Resolving Share URLs](../../epismo-basics/SKILL.md#resolving-share-urls) for the full algorithm.

### Who Can Use a Share URL

| Asset visibility | Recipient has workspace access | Can view via link |
| ---------------- | ------------------------------ | ----------------- |
| `private` | Yes | Yes |
| `private` | No | No |
| `public` | Any | Yes |

### Liking a Public Workflow

```bash
epismo asset like --id <id> --liked      # bookmark and signal quality
epismo asset like --id <id> --no-liked   # remove like
```

Liked assets surface higher in community discovery and appear in `like="liked"` filtered searches.

## Approval Boundary

Some actions require **explicit user approval** before writing. Do not proceed without it.

| Action | Approval required? | Why |
| ------ | ------------------ | --- |
| First-time public release (`visibility="public"`) | **Yes** | Content becomes globally visible |
| Changing existing workflow from `private` to `public` | **Yes** | May expose internal patterns or team information |
| Deprecating a workflow | **Yes** | Affects existing users of the pattern |
| Overwriting another owner's workflow | **Yes** | Affects content the current user does not own |
| Creating or updating a private workflow | No | Low risk, reversible |
| Changing `projects[]` on a private workflow | No | Scoping change only |
| Liking or un-liking | No | Reversible signal, no content change |

**How to obtain approval:** state the intended action, the workflow title, and the visibility setting. Wait for an explicit "yes" or equivalent before writing.
