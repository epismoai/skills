# Pack Alias

An alias is a short, human-readable name that resolves to a pack ID. Use it anywhere `--id` is accepted on `pack get` to avoid copying and pasting full UUIDs.

`alias` is a top-level resource — managed via `epismo alias ...` (CLI) or `/v1/aliases` (API), separate from pack CRUD.

## Alias Format

| Format              | Where accepted            | Resolves to          |
| ------------------- | ------------------------- | -------------------- |
| `myproject`         | upsert, get, list, delete | Your own alias       |
| `@handle/myproject` | get only                  | Another user's alias |

- **upsert / delete** — bare name only (`myproject`). The namespace is always your own account.
- **get** — accepts both forms to support referencing others' aliases.

## Access Options

- **CLI** — preferred when operating interactively. Credentials from `epismo login` are used automatically.
- **API (curl / scripted)** — use when building an integration or when the CLI is not available. Requires an OAuth `access_token`; to obtain one, see [Auth Setup](../SKILL.md#auth-setup-cli) in Epismo Basics.

---

### CLI

#### Upsert (create or update)

```bash
epismo alias upsert --type workflow --id <id> --alias myproject
epismo alias upsert --type context  --id <id> --alias mycontext
```

You can only alias your own packs. The command verifies ownership before writing.

#### Get (resolve a single alias)

```bash
epismo alias get --alias myproject
epismo alias get --alias @handle/myproject    # another user's alias
epismo alias get --alias myproject --type workflow
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
epismo alias delete --alias myproject
```

#### Use an alias to fetch a pack

`pack get` accepts `--alias` as an alternative to `--id`:

```bash
epismo pack get --alias myproject
epismo pack get --alias @handle/myproject
epismo pack get --alias mycontext --block-id <block-id>
```

`--id` and `--alias` are mutually exclusive; one is required.

---

### API (curl / scripted)

`accountId` is never passed as a request parameter — the API resolves it from the auth token internally.

#### Upsert

```bash
curl -sX PUT https://api.epismo.ai/v1/aliases \
  -H "Authorization: Bearer <ACCESS_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{"type":"workflow","id":"<pack-id>","alias":"myproject"}'
```

#### List

```bash
curl -sX GET https://api.epismo.ai/v1/aliases \
  -H "Authorization: Bearer <ACCESS_TOKEN>"
```

#### Get (resolve a single alias)

```bash
# Own alias
curl -sX GET "https://api.epismo.ai/v1/aliases?alias=myproject" \
  -H "Authorization: Bearer <ACCESS_TOKEN>"

# Another user's alias
curl -sX GET "https://api.epismo.ai/v1/aliases?alias=%40epismo%2Fmyproject" \
  -H "Authorization: Bearer <ACCESS_TOKEN>"

# With type filter
curl -sX GET "https://api.epismo.ai/v1/aliases?alias=myproject&type=workflow" \
  -H "Authorization: Bearer <ACCESS_TOKEN>"
```

#### Delete

```bash
curl -sX DELETE "https://api.epismo.ai/v1/aliases/myproject" \
  -H "Authorization: Bearer <ACCESS_TOKEN>"
```

#### Use an alias to fetch a pack

```bash
curl -sX GET "https://api.epismo.ai/v1/packs?alias=myproject" \
  -H "Authorization: Bearer <ACCESS_TOKEN>"
```

---

## Access Control

- **Upsert / Delete** — your aliases only. Ownership of the target pack is verified at write time.
- **Get (by alias)** — resolves the alias, then runs the normal pack ACL check. If you lack read access to the pack, the fetch fails even if the alias exists.
- **List** — your aliases only.
