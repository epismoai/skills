# Improve

Use this guide when real work on a case reveals a repeatable playbook improvement.

## Separate evidence from proposal

Read the case before proposing a change. `case get` returns only that case's latest five records; if it has handoffs, list related records with `ancestors` or `connected` so the suggestion is based on the thread.

- Put what happened in the case as a record.
- Put what should change next time in a suggestion.
- Keep one-off exceptions out of reusable guidance unless they reveal a general rule.

Create the suggestion against the playbook and the immutable base version that produced the observation; the base must be a version of that same playbook. Include the problem, proposed change, and expected benefit, and name a target step only when that step ID exists in the base version.

A suggestion snapshots playbook access when it is created. Reading it later requires access under both that snapshot and the playbook's current access, so access removed from the playbook also closes the suggestion.

## Work with ownership

The author may edit an open suggestion's title and content. Resolution splits by role:

- an owner manager applies or declines it;
- the author archives their own withdrawn proposal;
- reopening a declined suggestion is the owner manager's call, and reopening an archived one is the author's.

To apply:

1. read the suggestion, base version, and current latest version;
2. decide whether the proposal still fits latest;
3. publish a new version through an authorized human-reviewable surface;
4. resolve the suggestion as applied, naming that version as the result.

Applying requires a result version that is newer than the base version of the same playbook, so publish first and resolve second. Never mark applied before the corresponding version exists. Folding several open suggestions into one version is reasonable — stage the merged content in a draft (see [Author](./author.md)) and publish it once, then resolve each suggestion against that same result version. A draft is not itself a suggestion and never substitutes for one: it holds the owner's own in-progress edit, not a reviewable third-party proposal.

## Close the loop

Report back to the user which suggestions were applied, declined, or left open, and which case evidence supports each one. When a suggestion is declined, note the reason with the case record it came from so the same observation is not re-filed.
