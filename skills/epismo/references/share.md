# Share

Use this guide for consequences of sharing and reference choices.

Case access is independent of its source playbook. Check both objects when
sharing a case that uses a playbook.

## Expand access carefully

Before expanding access:

- remove credentials, private identifiers, and unnecessary personal data;
- inspect instructions and resource refs for internal-only material;
- ensure referenced resources are accessible to the intended audience;
- confirm owner, workspace, and team principals;

Access setters replace the whole `access` object: `visibility` (`private` or
`public`) and `grants` (user/team UUIDs mapped to `viewer` or `editor`). Retain
unrelated grants when changing only visibility. An empty map clears explicit
sharing independently of publication; never use `public` as a grant key.
Normal reads omit private grant maps; have a manager retrieve the complete
settings before replacing them. Viewers cannot edit content or sharing;
Playbook drafts require editing. Keep every task assignee's edit access.

## Review shared-team boundaries

A team can connect multiple workspaces. Confirm both the recipient's participation
workspace and the resource's home remain connected before relying on its grant.
Invitations are email-bound; recipients choose their workspace, and acceptance
does not add every member. Disconnect removes that workspace's participants and
team access; direct user grants remain.

## Review derived content before sharing

Copying external evidence into a case record or reusable guidance creates a
separate disclosure. Review that text for its destination audience, even when
the original source has restricted access. Removing a source later does not
withdraw information already copied into another object.

## Manage references

Use an alias when repeated human-readable lookup matters. Share a qualified
reference using the current owner's handle. Before reusing an old reference,
check it again: ownership transfer, handle changes, and alias reassignment can
change where it leads without changing the text someone saved.

Verify a share URL from the intended recipient's account and workspace before
relying on it. The URL alone does not confirm what that recipient can read.
Current share URLs have no expiry or revoke operation; use a different sharing
method when either control is needed.

## Archive deliberately

Prefer narrowing access when guidance should merely stop spreading. Archive
when the playbook itself should end: pinned versions in existing cases then
stop resolving even though those cases remain readable.
