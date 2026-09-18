# Share

Use this guide for access, aliases, share tokens, public visibility, and archival. A playbook and a case have independent access; pick the mechanism that matches the object.

## Choose the mechanism

- **Playbook access:** `visibility` plus durable editor user account and team IDs. `public` permits published reads only; owners, and every member of a workspace-owned playbook, are implicit.
- **Case access:** never inherits from its playbook. It may include `public`, which is read-only and exposes the current title, input, records, and readable handoff neighborhood; it never exposes tasks, assignment, or collaborator identities. Tasks and records have no access list of their own and follow the current parent case. Anyone with case work access may append records to an open case, update tasks, assign, retitle, close, reopen, and connect handoffs; only the creator may update or redact their own non-system record. Access changes and archiving stay with the current assignee. Editors may also request an Epismo AI review and change whether OUTPUT records enqueue one.
- **Alias:** one account namespace's human-readable reference to a playbook; it grants no access.
- **Share URL:** an opaque token URL for a playbook or a case. The current web flow still applies the target's live access check; the token itself does not widen access.
- **Star:** personal saving and a discovery signal, not access.
- **Draft:** unpublished, mutable playbook content restricted to callers with playbook edit access; neither public readers nor a share link grants access.

## Expand access carefully

Require explicit intent before public access or a wider audience. Before expanding access:

- remove credentials, private identifiers, and unnecessary personal data;
- inspect instructions and resource refs for internal-only material;
- ensure referenced resources are accessible to the intended audience;
- confirm owner, workspace, and team principals;
- preserve unrelated editors only when the operation and user's intent allow it.

`playbook access set` and `case access set` replace the complete editor list; they are not an incremental add. Read current access before replacing it, and retain editor IDs that the caller cannot resolve. Only an owner manager can change playbook access. A case access change requires the current case assignee, the current lock version, and work access that still covers every task assignee. The current case assignee has implicit access and is omitted from the editor list; the service rejects an update that would strand a task assignee rather than silently unassigning them.

## Manage references

Use an alias when repeated human-readable lookup matters. Each personal or managed workspace namespace can assign at most one alias to any readable playbook. Setting a different alias for the same playbook renames that namespace's reference; it does not move an alias already occupied by another playbook. The playbook owner's alias is official and may be indexed. A third-party alias is not indexed or shown as official; do not surface it outside its namespace even though a raw alias listing may return it. Resolution enforces live playbook access, so an alias never preserves access after removal.

Treat a share URL as a credential even though the current web flow does not bypass access. Any authenticated reader can create or retrieve the object's single stable token; there is currently no expiry, rotation, or revoke operation. Do not create one speculatively, and keep it out of public text, playbook content, cases, records, and logs.

Before relying on a share URL, verify it as the intended recipient in the intended workspace or anonymous context. The web route resolves the token to `/playbooks/{id}` or `/cases/{id}` and performs the normal access check. Use `playbook share` or `epismo_playbook_share` for a playbook link, and `case share` or `epismo_case_share` for a case link. A private target still requires access; a public case exposes its current title, input, records, and readable handoffs. Archiving blocks the target read but does not delete the token mapping. Share URLs do not expose drafts, tasks, or restricted case fields.

## Archive deliberately

Archive a playbook only with explicit intent. It disappears from search and direct reads, and it accepts no further versions, cases, or share tokens.

Archival reaches further than discovery: its versions stop resolving too. Existing cases keep their pinned version ID and stay readable as cases, but the definition behind them can no longer be fetched. Prefer narrowing access when guidance should merely stop spreading, and archive when the playbook itself should end.
