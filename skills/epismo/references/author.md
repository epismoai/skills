# Author

Use this guide to create a playbook, iterate on a draft, or publish a new immutable version. Do not author a playbook merely to store one case's facts; those belong on the case as records.

## Search first

Search existing playbooks before creating one. Improve a fitting playbook when the outcome, audience, and lifecycle match. Create a new one when those boundaries differ materially. If the request is "turn this case into guidance," extract the repeatable procedure and file it through [Improve](./improve.md) when a playbook already exists.

## Write the definition

Write steps as judgment-sized guidance rather than tiny commands or a rigid execution graph. Array order is the recommended order; there is no position field, status, or assignee on a step. Expected outputs are free-form JSON hints, not completion criteria. Keep run-specific facts in cases and large background material in referenced documents.

## Keep step identity stable

The server owns step IDs and never reuses one within the same playbook.

- Omit the ID for a new step, including when publishing from a base version. The server assigns one and returns the published definition.
- Send the existing ID when editing or reordering a step that already exists in the base version — that is what preserves its identity across versions.
- An ID that is not in the base version is rejected, and a deleted ID is never reused.

## Reference resources as hints

Treat each resource hint as a discovery hint, not an installation or trust decision. Never put credentials or expiring signed URLs in one. Prefer a pinned or conservative selector for shared and audited work; resolution, permission, and sandboxing belong to the runtime.

## Iterate with a draft

A playbook has at most one draft: mutable, unpublished content that saves cheaply and repeatedly without minting a version or consuming step IDs. Use it while the definition is still moving. A draft applies only to an existing playbook; create the first version directly.

- Save against the revision you last read, or the new-draft sentinel when none exists. If someone else saved first, re-read and merge deliberately.
- Before publishing, read and review the draft, then publish against that returned revision. If anyone saves after the review, re-read the newer content before deciding again.
- Anyone with playbook edit access can read, save, discard, or publish a draft. That includes editors, owner managers, and members of a workspace-owned playbook. Public readers and share links have no path to unpublished content.
- Saving validates the definition and any retained step IDs, but does not allocate IDs for new steps. Allocation happens when publishing.
- Publishing the draft mints a new immutable version from its current content and discards the draft in the same step. Discard it directly instead when the direction was wrong and should not become a version.
- A draft is not a suggestion. It is an in-progress edit by someone with playbook edit access, not a third party's proposal against a fixed base version — see [Improve](./improve.md) for that path.

## Publish safely

Creation atomically creates the playbook and its first version. `ownerId` may be your personal account, or a workspace you belong to. Any member may create a playbook owned by that workspace; `visibility` and `editors` apply only when you manage the owner namespace, otherwise the playbook is private with you as an editor. Access changes still require an owner manager.

Direct publishing must be based on the current latest version and creates a new immutable version; it never edits the base. Publishing a draft uses the reviewed draft revision and the version captured by its last save. It fails if either moved in the meantime. Anyone with playbook edit access may publish; an archived playbook cannot receive new versions.

After a conflict:

1. fetch the latest version;
2. compare it with the intended change;
3. merge deliberately;
4. publish with a new idempotency key.

Verify the new version ID, definition, step IDs, access, canonical digest, and latest pointer. Publishing is a reviewable change to shared guidance: show the diff before publishing on someone's behalf. Creating under another person's account, in a workspace you do not belong to, or publishing a playbook you cannot edit requires an authorized surface and role; otherwise create a suggestion.
