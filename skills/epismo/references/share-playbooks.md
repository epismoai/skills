# Share Playbooks

Use this guide for ACLs, aliases, share tokens, public access, and archival.

## Choose the mechanism

- **ACL:** durable access for User Account IDs and Project IDs; Playbooks may also include `public`.
- **Alias:** stable human-readable reference to a logical Playbook; it grants no access.
- **Share token:** bearer read access to one Playbook without changing its ACL.
- **Star:** personal saving and a discovery signal, not access.

Cases have independent ACLs and cannot be public. Tasks and Records carry no ACL of their own and are authorized through the current parent Case ACL. A cross-Case Record list checks those live Case ACLs before applying optional filters; an ACL filter only narrows results and never grants access.

## Expand access carefully

Require explicit intent before public access or a wider audience. Before expanding access:

- remove credentials, private identifiers, and unnecessary personal data;
- inspect instructions and resource refs for internal-only material;
- ensure referenced resources are accessible to the intended audience;
- confirm owner, workspace, and project principals;
- preserve unrelated ACL entries only when the operation and user's intent allow it.

An ACL update replaces the whole list; it is not an incremental add. Read the current access before replacing it. A Playbook ACL change requires rights over the owner Account. A Case ACL change requires the Case starter or assignee, the current lock version, and an ACL that still covers every current Case and Task assignee — the service rejects an update that would strand one rather than silently unassigning them.

## Manage references

Use aliases when repeated human-readable lookup matters. Write aliases only in an owner namespace you manage. Resolve the alias, then enforce the live Playbook ACL. Before deleting or repointing an alias, distinguish changing the name, the target, and the underlying Playbook.

Treat share tokens as credentials: anyone holding one can read that Playbook, and the token holder gets read access only — not the Playbook's Cases, and not the ability to start one. Only an owner manager can create a token, and the MVP surface has no revoke operation, so set an expiry when creating it rather than assuming access can be withdrawn later. Return a token only to the intended recipient and keep it out of public text, Records, and Playbook content. Archiving the Playbook is what stops existing tokens from resolving.

## Archive deliberately

Archive a Playbook only with explicit intent. It disappears from search, direct reads, and starred lists, and it accepts no further Versions, Cases, or share tokens.

Archival reaches further than discovery: its Versions stop resolving too. Existing Cases keep their pinned Version ID and digest and stay readable as Cases, but the Definition behind them can no longer be fetched. Prefer narrowing the ACL when guidance should merely stop spreading, and archive when the Playbook itself should end.
