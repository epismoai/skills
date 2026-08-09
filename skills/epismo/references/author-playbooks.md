# Author Playbooks

Use this guide to create a Playbook or publish a new immutable Version.

## Search first

Search existing Playbooks before creating one. Improve a fitting Playbook when the outcome, audience, and lifecycle match. Create a new one when those boundaries differ materially.

## Write the Definition

A Definition holds exactly these fields, and unknown fields are rejected:

- **schemaVersion** — only `1` is supported.
- **title** — required; name the outcome.
- **description** — say concisely when to use it.
- **category** — optional, and one of `productivity`, `programming`, `design`, `sales`, `marketing`, `operations`, `learning`.
- **inputSchema** — JSON Schema Draft 2020-12 whose root type is an object; omit it to accept any object.
- **steps** — ordered guidance, each with a title, instructions, optional resource hints, and optional expected outputs.

Write Steps as judgment-sized guidance rather than tiny commands or a rigid execution graph. Array order is the recommended order; there is no position field, status, or assignee on a Step. Expected outputs are free-form JSON hints, not completion criteria. Keep run-specific facts in Cases and large background material in referenced documents.

## Keep Step identity stable

The server owns Step IDs: four characters of `A-Z0-9`, unique across every Version of the same Playbook.

- Omit the ID for a new Step, including when publishing from a base Version. The server assigns one and returns the published Definition.
- Send the existing ID when editing or reordering a Step that already exists in the base Version — that is what preserves its identity across Versions.
- An ID that is not in the base Version is rejected, and a deleted ID is never reused.

## Reference resources as hints

Each hint is a `kind` + `ref` + optional `selector`, unique within a Step.

- **kind** is one of `skill`, `mcp`, `cli`, `api`, `plugin`, `graph`, `document`, `agent`, `custom`.
- **ref** must use an allowed scheme (`clawhub`, `mcp`, `github`, `https`, `http`, `file`, `skill`, `plugin`, `document`) and must not contain credentials, userinfo, secret query parameters, or expiring signed URLs.
- **selector** is an opaque string such as `stable`, `main`, or `^1`. Epismo stores it without resolving or validating it.

Prefer a pinned or conservative selector for shared and audited work. Resolution, installation, trust, permission, and sandboxing belong to the runtime.

## Publish safely

Creation atomically creates the Playbook and its first Version under an owner Account you manage, with an explicit non-empty ACL.

Publishing requires the current **baseVersionId** and creates a new immutable Version; it never edits the base. Only an owner manager may publish, and an archived Playbook cannot receive new Versions.

After a conflict:

1. fetch the latest Version;
2. compare it with the intended change;
3. merge deliberately;
4. publish with a new idempotency key.

Verify the new Version ID, Definition, Step IDs, ACL, canonical digest, and latest pointer. Publishing is a reviewable change to shared guidance: show the diff before publishing on someone's behalf. Creating or publishing in an owner namespace you do not manage requires an authorized surface and role; otherwise create a Suggestion.
