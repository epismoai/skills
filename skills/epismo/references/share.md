# Share

Use this guide for consequences of sharing and reference choices.

Case access is independent of its source playbook. Check both objects when
sharing a case that uses a playbook.

Keep ownership, responsibility, and sharing separate. The owning account is
also the billing home; a Case assignee or Playbook maintainer receives no
access just from that responsibility. Check current editing access before
assigning work. Workspace member removal clears the User's direct grants on
items owned by that Workspace, including completed and archived items. Initial
handover preserves grants; finalization catches late grants. Personal and
other-Workspace shares remain unchanged. Non-members may be explicitly shared
with again after removal. Team and Workspace management authority use live membership
and the participation Workspace's SSO checks. Required SSO applies to members;
non-member direct shares use normal Epismo authentication. Lists use explicit access and supported publication;
ownership alone does not put an item in a list.

Use assignment for responsibility and ownership transfer for the owning and
billing account. Transferring a Case clears its saved Brief, including DNA and
priority, while preserving sharing and assignment. Preserve any durable
conclusions in the Case's Records before relying on a regenerated Brief.

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
restore it with its versions and sharing intact. An archived Case likewise
retains its work until restored or permanently deleted; permanent deletion
requires ownership and cannot be undone.

Sharing updates must retain at least one explicit User or Team editor; ownership and publication do not count. Creation adds the creator as an editor. Before removing a Workspace member who is the last editor of any owned item, add another editor, including on completed and archived items.

Web Workspace discovery uses direct User sharing and selected Teams, including connected-Team shares from other owning accounts. Deselecting every Team leaves direct User sharing. The sidebar only narrows discovery; it does not change direct-link access, editing capabilities, or mutation permissions. CLI and MCP use normal access-based visibility and expose no sidebar Team-selection filter.
