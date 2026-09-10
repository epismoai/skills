# Author

Use this guide to create a Playbook, iterate on a Draft, or publish a new immutable Version.

## Search first

Search existing Playbooks before creating one. Improve a fitting Playbook when the outcome, audience, and lifecycle match. Create a new one when those boundaries differ materially.

## Write the Definition

Write Steps as judgment-sized guidance rather than tiny commands or a rigid execution graph. Array order is the recommended order; there is no position field, status, or assignee on a Step. Expected outputs are free-form JSON hints, not completion criteria. Keep run-specific facts in Cases and large background material in referenced documents.

## Keep Step identity stable

The server owns Step IDs and never reuses one within the same Playbook.

- Omit the ID for a new Step, including when publishing from a base Version. The server assigns one and returns the published Definition.
- Send the existing ID when editing or reordering a Step that already exists in the base Version — that is what preserves its identity across Versions.
- An ID that is not in the base Version is rejected, and a deleted ID is never reused.

## Reference resources as hints

Treat each resource hint as a discovery hint, not an installation or trust decision. Never put credentials or expiring signed URLs in one. Prefer a pinned or conservative selector for shared and audited work; resolution, permission, and sandboxing belong to the runtime.

## Iterate with a Draft

A Playbook has at most one Draft: mutable, unpublished content that saves cheaply and repeatedly without minting a Version or consuming Step IDs. Use it while the Definition is still moving. A Draft applies only to an existing Playbook; create the first Version directly.

- Save against the revision you last read, or the new-Draft sentinel when none exists. If someone else saved first, re-read and merge deliberately.
- Before publishing, read and review the Draft, then publish against that returned revision. If anyone saves after the review, re-read the newer content before deciding again.
- Editors and owner managers can read, save, discard, or publish a Draft. Public readers and share links have no path to unpublished content.
- Saving validates the Definition and any retained Step IDs, but does not allocate IDs for new Steps. Allocation happens when publishing.
- Publishing the Draft mints a new immutable Version from its current content and discards the Draft in the same step. Discard it directly instead when the direction was wrong and should not become a Version.
- A Draft is not a Suggestion. It is the owner's own in-progress edit, not a third party's proposal against a fixed base Version — see [Improve](./improve.md) for that path.

## Publish safely

Creation atomically creates the Playbook and its first Version under an owner Account you manage. Use `visibility` and `editors` for access; omitting editors creates a private owner-only Playbook.

Direct publishing must be based on the current latest Version and creates a new immutable Version; it never edits the base. Draft publishing uses the reviewed Draft revision and the Version captured by its last save. It fails if either moved in the meantime. Editors and owner managers may publish; an archived Playbook cannot receive new Versions.

After a conflict:

1. fetch the latest Version;
2. compare it with the intended change;
3. merge deliberately;
4. publish with a new idempotency key.

Verify the new Version ID, Definition, Step IDs, access, canonical digest, and latest pointer. Publishing is a reviewable change to shared guidance: show the diff before publishing on someone's behalf. Creating or publishing in an owner namespace you do not manage requires an authorized surface and role; otherwise create a Suggestion.
