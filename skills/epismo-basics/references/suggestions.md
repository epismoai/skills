# Suggestions

A suggestion is a text-first improvement proposal sent to a pack's **owner**. Anyone who can read a workflow or context pack can suggest a change to it; the owner reviews and resolves it. Suggestions never edit the pack directly — proposed edits go in the suggestion `content`, and the owner applies them through a normal `update pack`.

`suggestion` is a top-level resource — managed via `epismo suggestion ...` (CLI) or `epismo_suggestion_*` (MCP), separate from pack CRUD.

## Lifecycle

Every suggestion has a `status`:

| Status     | Meaning                                                   | Set by          |
| ---------- | --------------------------------------------------------- | --------------- |
| `open`     | Submitted, awaiting the owner's decision (initial state). | author (create) |
| `applied`  | Owner accepted it; the change has been (or will be) made. | owner (resolve) |
| `declined` | Owner decided not to act on it.                           | owner (resolve) |
| `archived` | Set aside without a decision; keeps the inbox clean.      | owner (resolve) |

`open` → `applied` / `declined` / `archived`. The owner can move a suggestion back to `open` with `resolve --status open`.

## Snapshot

When a suggestion is created, the pack's content at that moment is captured as an immutable **snapshot**. The suggestion stays anchored to what the author actually saw even if the pack changes later. Read it with `--include-snapshot` (get) or `--include-snapshots` (list). It is omitted by default to keep responses small.

## Who can do what

| Action                 | Who                                                    |
| ---------------------- | ------------------------------------------------------ |
| Create                 | Anyone who can read the pack (private or public).      |
| Update (title/content) | The **author** only.                                   |
| Resolve (status)       | The pack **owner** only.                               |
| Get / List             | Anyone whose ACL grants read access to the suggestion. |

Creating a suggestion costs **5 credits**, `list` costs **2 credits**, and `get`, `update`, and `resolve` cost **1 credit** each. Creation is additionally capped at **20 suggestions per day** per account; beyond that, create fails with a daily-limit error.

## Reference resolution

`create` and `list --reference` accept the same pack reference forms as `pack get`: a pack ID, an `@alias`, a share URL, or a hub URL. See [Pack Alias](./pack-alias.md) for alias forms and [Pack References](../SKILL.md#pack-references-resolving-share-urls) for share/hub links.

## Listing modes

`list` answers three different questions depending on the flags:

| Goal                                      | Flags                                       |
| ----------------------------------------- | ------------------------------------------- |
| Suggestions **I submitted** (default)     | _(no flags)_                                |
| My **inbox** — suggestions on packs I own | `--owner`                                   |
| Suggestions **for one pack**              | `--reference @pack` (optionally `--author`) |

Add `--status open` (or a comma-separated list) to filter by lifecycle state, and `--page` to paginate (page size 20).

`list` returns each suggestion's `content` as a truncated preview — the same selective-fetch pattern packs use. Use `get` to read the full content.

---

## CLI

```bash
# Send an improvement suggestion to a pack owner
epismo suggestion create @market-research --title "Add review step" \
  --content "Add a final human review before publishing to reduce hallucinated findings."
epismo suggestion create <pack-id> --input @suggestion.json

# Read one suggestion, optionally with the captured pack snapshot
epismo suggestion get <suggestion-id>
epismo suggestion get <suggestion-id> --include-snapshot

# List: submitted (default), inbox, or per-pack
epismo suggestion list                              # suggestions you submitted
epismo suggestion list --owner --status open        # your inbox, open only
epismo suggestion list --reference @market-research # for one pack
epismo suggestion list --status open,declined

# Edit your own suggestion (author only)
epismo suggestion update <suggestion-id> --title "Add review step" \
  --content "Please add a reviewer approval step before publish."

# Resolve a suggestion you received (owner only)
epismo suggestion resolve <suggestion-id> --status applied
```

`--status` accepts `open`, `applied`, `declined`, or `archived`.

On MCP, derive the tool name mechanically from the full CLI command name (`epismo suggestion list` → `epismo_suggestion_list`); flags become camelCase parameters and `--status open,applied` becomes `statuses: ["open", "applied"]`. See [surface conventions](../SKILL.md#surface-conventions).
