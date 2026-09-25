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

Access setters replace the collaborator list. Read the current list first and
retain unrelated collaborators unless the user intended to remove them.

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
