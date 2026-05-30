# Pack Alias

An alias is a short, human-readable name that resolves to a pack ID. Pass it as the positional `<reference>` to `pack get` (same slot as the ID) to avoid copying and pasting full UUIDs.

`alias` is a top-level resource — managed via `epismo alias ...` (CLI) or `/v1/aliases` (API), separate from pack CRUD.

## Namespaces

Aliases live in one of two namespaces:

- **personal** — your own account (the default).
- **workspace** — the active workspace's shared account. A workspace alias is a single shared record: any workspace member resolves it, and any member can repoint it (last write wins). Lets a teammate's pack be called almost like your own.

On **upsert / delete**, pick the namespace with `--namespace personal|workspace` (default `personal`). The alias name itself is bare — `<alias>` or `@<alias>` (a leading `@` is stripped); it must not contain `/`.

## Reference Format (get / `pack get`)

| Format                | Resolves in                                                        |
| --------------------- | ------------------------------------------------------------------ |
| `<alias>` / `@<alias>`| personal namespace first, then the active workspace (fallback)     |
| `@<handle>/<alias>`   | another account's alias, by public handle                          |

- The bare form keeps daily use short while letting a personal alias shadow a shared team one.
- The namespace is **not** encoded in the reference string (any `@me/`/`@team/`-style prefix would collide with real account handles, which are plain `[a-z0-9-]`). To pin resolution to one side when a personal alias shadows a workspace one, pass `--namespace personal|workspace` (CLI) / `namespace` (API/MCP) alongside the bare alias.

## Who can create an alias

Only the pack's **owner** may create or repoint an alias to it (in either the personal or the workspace namespace). Everyone else simply *uses* the alias — resolution only checks that you can read the target. To bookmark someone else's pack for yourself, use `like`, not an alias.

## Access Options

- **CLI** — preferred when operating interactively. Credentials from `epismo login` are used automatically.
- **API (curl / scripted)** — use when building an integration or when the CLI is not available. Requires an OAuth `access_token`; to obtain one, see [Auth Setup](../SKILL.md#auth-setup-cli) in Epismo Basics.

---

### CLI

#### Upsert (create or update)

```bash
epismo alias upsert @myproject --type workflow --id <id>                        # personal
epismo alias upsert @deploy --type workflow --id <id> --namespace workspace     # shared with the workspace
```

You can alias only packs you own; the `--namespace workspace` form shares the name with the active workspace.

#### Get (resolve a single alias)

```bash
epismo alias get @myproject                       # personal, then workspace fallback
epismo alias get @deploy --namespace workspace    # pin to the workspace's alias
epismo alias get @handle/myproject                # another account's alias
```

Returns `id`, `type`, `accountId`, `alias`, `namespace` (`personal` | `workspace`, when applicable), and `reference` — the bare `@alias` (resolves personal-first, then the workspace). For a **public** pack, clients that know the owner's handle render the public `@handle/alias` form instead.

#### List (your personal + workspace aliases)

Lists your personal aliases plus, when a workspace is active, the workspace's shared aliases. Each entry is tagged with its `namespace`. `--type` is optional.

```bash
epismo alias list
epismo alias list --type workflow
```

#### Delete

```bash
epismo alias delete @myproject                     # personal
epismo alias delete @deploy --namespace workspace  # the workspace's shared alias
```

#### Use an alias to fetch a pack

`pack get` accepts an alias reference in the same positional `<reference>` slot as an ID:

```bash
epismo pack get @myproject
epismo pack get @deploy --namespace workspace
epismo pack get @handle/myproject
epismo pack get @mycontext --block-id <block-id>
```

---

### API (curl / scripted)

`accountId` is never passed as a request parameter — the API resolves it from the auth token internally.

#### Upsert

`namespace` is optional (`personal` default, or `workspace`). The workspace is taken from the auth token / `workspaceId` query param.

```bash
curl -sX PUT https://api.epismo.ai/v1/aliases \
  -H "Authorization: Bearer <ACCESS_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{"type":"workflow","id":"<pack-id>","alias":"@myproject","namespace":"personal"}'
```

#### List

```bash
curl -sX GET https://api.epismo.ai/v1/aliases \
  -H "Authorization: Bearer <ACCESS_TOKEN>"
```

#### Get (resolve a single alias)

```bash
# Own alias (personal, then workspace fallback)
curl -sX GET "https://api.epismo.ai/v1/aliases?alias=%40myproject" \
  -H "Authorization: Bearer <ACCESS_TOKEN>"

# Pin to the workspace namespace
curl -sX GET "https://api.epismo.ai/v1/aliases?alias=%40deploy&namespace=workspace" \
  -H "Authorization: Bearer <ACCESS_TOKEN>"

# Another user's alias
curl -sX GET "https://api.epismo.ai/v1/aliases?alias=%40epismo%2Fmyproject" \
  -H "Authorization: Bearer <ACCESS_TOKEN>"

# With type filter
curl -sX GET "https://api.epismo.ai/v1/aliases?alias=%40myproject&type=workflow" \
  -H "Authorization: Bearer <ACCESS_TOKEN>"
```

#### Delete

```bash
curl -sX DELETE "https://api.epismo.ai/v1/aliases/%40myproject" \
  -H "Authorization: Bearer <ACCESS_TOKEN>"

# workspace-scoped alias
curl -sX DELETE "https://api.epismo.ai/v1/aliases/%40deploy?namespace=workspace" \
  -H "Authorization: Bearer <ACCESS_TOKEN>"
```

#### Use an alias to fetch a pack

```bash
curl -sX GET "https://api.epismo.ai/v1/packs?alias=%40myproject" \
  -H "Authorization: Bearer <ACCESS_TOKEN>"
```

---

## Access Control

- **Upsert** — writes to your personal namespace or the active workspace (`namespace`). Only the pack's **owner** may alias it.
- **Delete** — removes an alias from the chosen namespace (`namespace`).
- **Get (by alias)** — resolves the alias, then runs the normal pack ACL check. If you lack read access to the pack, the fetch fails even if the alias exists.
- **List** — your personal aliases plus the active workspace's shared aliases, each tagged with `namespace`.
