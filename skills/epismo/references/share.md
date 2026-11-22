# Share

Use this guide for consequences of sharing and reference choices.

Case access is independent of its source playbook. Check both objects when
sharing a case that uses a playbook.

Keep the owning Space, administrative account, and responsibility separate.
A Case assignee or Playbook maintainer needs existing editor access. The
owning Space supplies editor access. In a workspace, use membership UUIDs to
grant access for the current membership and `personal:EMAIL` or
`personal:USER_UUID` for personal guest access. A workspace membership grant
expires when that membership ends.

Use `--owner team:<name-or-id>` to create a Case or Playbook owned by a Team,
or `--owner me` for your current workspace membership. Creation and copying
require an explicit owner in a workspace, including a saved default workspace.
Outside a workspace, omitting the owner selects your personal account. Use
`case transfer` or `playbook transfer` with `--owner` to change ownership within
the same workspace while preserving additional shares. The previous owner loses
its inherited editor grant. Moving a Case clears its derived reports. Preserve
durable conclusions in Case records.

## Expand access carefully

Before expanding access:

- remove credentials, private identifiers, and unnecessary personal data;
- inspect instructions and resource refs for internal-only material;
- ensure referenced resources are accessible to the intended audience;
- confirm owner, workspace, and team principals;

Preserve unrelated collaborators when changing the audience. Match access to
the recipient's responsibility: reading guidance, contributing work, or managing
the collaboration. Ensure assigned work remains accessible to whoever must
complete it.

## Review shared-team boundaries

Use a shared team when people from separate workspaces need an ongoing
collaboration boundary. Keep unrelated clients or engagements separate. Before
adding participants, review the material already shared with that team; when a
collaboration ends, review access across its resources and participants.

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
when the playbook should leave normal use: pinned versions in existing cases
then stop resolving even though those cases remain readable. Editors can
restore it with its versions and sharing intact. An archived case likewise
retains its work until restored or permanently deleted; permanent deletion
requires ownership and cannot be undone.

Additional grants can be empty because the owning Space supplies editor access. Before removing a workspace member, review work held in their private Space and hand over open responsibilities.

Web workspace discovery uses direct user sharing and selected teams, including connected-team shares from other owning accounts. Deselecting every team leaves direct user sharing. The sidebar only narrows discovery; it does not change direct-link access, editing capabilities, or mutation permissions. CLI and MCP use normal access-based visibility and expose no sidebar team-selection filter.
