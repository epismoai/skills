# Pack Alias

An alias is a short, human-readable name that resolves to a pack ID. Pass it as the positional `<reference>` to `pack get` (same slot as the ID) to avoid copying and pasting full UUIDs.

`alias` is a top-level resource — managed via `epismo alias ...` (CLI) or `/v1/aliases` (API), separate from pack CRUD.

## Alias Format

| Format                    | Where accepted            | Resolves to          |
| ------------------------- | ------------------------- | -------------------- |
| `<alias>`                 | upsert, get, list, delete | Your own alias       |
| `@<alias>`                | upsert, get, delete       | Your own alias       |
| `@<handle>/<alias>`       | get only                  | Another user's alias |

- **upsert / delete** — accept `<alias>` or `@<alias>`; both are stored without `@`. The namespace is always your own account.
- **get** — prefer `@<alias>` for your own alias and `@<handle>/<alias>` for another user's alias. Bare aliases are accepted for compatibility.

## Access Options

- **CLI** — preferred when operating interactively. Credentials from `epismo login` are used automatically.
- **API (curl / scripted)** — use when building an integration or when the CLI is not available. Requires an OAuth `access_token`; to obtain one, see [Auth Setup](../SKILL.md#auth-setup-cli) in Epismo Basics.

---

### CLI

#### Upsert (create or update)

```bash
epismo alias upsert @myproject --type workflow --id <id>
epismo alias upsert @mycontext --type context  --id <id>
```

You can only alias your own packs. The command verifies ownership before writing.

#### Get (resolve a single alias)

```bash
epismo alias get @myproject
epismo alias get @handle/myproject    # another user's alias
epismo alias get @myproject --type workflow
```

Returns `id`, `type`, `accountId`, and `alias`.

#### List (all your aliases)

`--type` is optional. Omit to return all aliases; pass it to narrow results to a specific pack type.

```bash
epismo alias list
epismo alias list --type workflow
epismo alias list --type context
```

#### Delete

```bash
epismo alias delete @myproject
```

#### Use an alias to fetch a pack

`pack get` accepts an alias in the same positional `<reference>` slot as an ID:

```bash
epismo pack get @myproject
epismo pack get @handle/myproject
epismo pack get @mycontext --block-id <block-id>
```

---

### API (curl / scripted)

`accountId` is never passed as a request parameter — the API resolves it from the auth token internally.

#### Upsert

```bash
curl -sX PUT https://api.epismo.ai/v1/aliases \
  -H "Authorization: Bearer <ACCESS_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{"type":"workflow","id":"<pack-id>","alias":"@myproject"}'
```

#### List

```bash
curl -sX GET https://api.epismo.ai/v1/aliases \
  -H "Authorization: Bearer <ACCESS_TOKEN>"
```

#### Get (resolve a single alias)

```bash
# Own alias
curl -sX GET "https://api.epismo.ai/v1/aliases?alias=%40myproject" \
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
```

#### Use an alias to fetch a pack

```bash
curl -sX GET "https://api.epismo.ai/v1/packs?alias=%40myproject" \
  -H "Authorization: Bearer <ACCESS_TOKEN>"
```

---

## Access Control

- **Upsert / Delete** — your aliases only. Ownership of the target pack is verified at write time.
- **Get (by alias)** — resolves the alias, then runs the normal pack ACL check. If you lack read access to the pack, the fetch fails even if the alias exists.
- **List** — your aliases only.
