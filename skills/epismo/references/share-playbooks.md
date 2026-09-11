# Share Playbooks

Use this guide for ACLs, aliases, share tokens, public access, and archival.

## Choose the mechanism

- **Playbook access:** `visibility` plus durable editor User Account and Team IDs. `public` permits published reads only; owners, and every member of a workspace-owned Playbook, are implicit.
- **Alias:** one Account namespace's human-readable reference to a Playbook; it grants no access.
- **Share URL:** an opaque token URL intended for recipient access; the current web flow still applies Playbook access.
- **Star:** personal saving and a discovery signal, not access.
- **Draft:** unpublished, mutable Playbook content restricted to callers with Playbook edit access; neither public readers nor a share link grants access.

Cases have independent ACLs and may include `public`. Public Case access is read-only and exposes the current title, Records, and readable handoff neighborhood; it never exposes Tasks, assignment, input, or collaborator identities. Tasks and Records carry no ACL of their own and are authorized through the current parent Case ACL. Anyone covered by that live ACL may append Records to an open Case; managing the Case remains limited to its current assignee, while ACL editors can create Tasks. Records can be listed across readable Cases, and optional filters only narrow results; they never grant access.

## Expand access carefully

Require explicit intent before public access or a wider audience. Before expanding access:

- remove credentials, private identifiers, and unnecessary personal data;
- inspect instructions and resource refs for internal-only material;
- ensure referenced resources are accessible to the intended audience;
- confirm owner, workspace, and team principals;
- preserve unrelated ACL entries only when the operation and user's intent allow it.

`playbook access set` replaces the complete editor list; it is not an incremental add. Read current access before replacing it, and retain editor IDs that the caller cannot resolve. Only an owner manager can change Playbook access. A Case ACL or access change requires the current Case assignee, the current lock version, and work access that still covers every Task assignee. The current Case assignee has implicit access and is omitted from the editor list; the service rejects an update that would strand a Task assignee rather than silently unassigning them.

## Manage references

Use an alias when repeated human-readable lookup matters. Each personal or managed Workspace namespace can assign at most one alias to any readable Playbook. Setting a different alias for the same Playbook renames that namespace's reference; it does not move an alias already occupied by another Playbook. The Playbook owner's alias is official and may be indexed. A third-party alias is not indexed or shown as official; do not surface it outside its namespace even though a raw alias listing may return it. Resolution enforces live Playbook access, so an alias never preserves access after removal.

Treat a share URL as a credential even though the current web flow does not bypass Playbook access. Any authenticated reader can create or retrieve the Playbook's single stable token; there is currently no expiry, rotation, or revoke operation. Do not create one speculatively, and keep it out of public text, Playbook content, Cases, Records, and logs.

Before relying on a share URL, verify it as the intended recipient in the intended workspace or anonymous context. The web route resolves the token to `/playbooks/{id}` or `/cases/{id}` and performs the normal access check. Use `case share` or `epismo_case_share` for a Case link. A private target still requires access; a public Case exposes its current title, Records, and readable handoffs. Archiving blocks the target read but does not delete the token mapping. Share URLs do not widen access or expose Drafts, Tasks, or restricted Case fields.

## Archive deliberately

Archive a Playbook only with explicit intent. It disappears from search and direct reads, and it accepts no further Versions, Cases, or share tokens.

Archival reaches further than discovery: its Versions stop resolving too. Existing Cases keep their pinned Version ID and stay readable as Cases, but the Definition behind them can no longer be fetched. Prefer narrowing access when guidance should merely stop spreading, and archive when the Playbook itself should end.
