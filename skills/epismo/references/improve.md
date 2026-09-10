# Improve

Use this guide when real Case work reveals a repeatable Playbook improvement.

## Separate evidence from proposal

Read the Case before proposing a change. `case get` returns only that Case's latest five Records; if it has handoffs, list related Records with `ancestors` or `connected` so the Suggestion is based on the thread.

- Put what happened in the Case as a Record.
- Put what should change next time in a Suggestion.
- Keep one-off exceptions out of reusable guidance unless they reveal a general rule.

Create the Suggestion against the Playbook and the immutable base Version that produced the observation; the base must be a Version of that same Playbook. Include the problem, proposed change, and expected benefit, and name a target Step only when that Step ID exists in the base Version.

A Suggestion snapshots Playbook access when it is created. Reading it later requires access under both that snapshot and the Playbook's current access, so access removed from the Playbook also closes the Suggestion.

## Work with ownership

The author may edit an open Suggestion's title and content. Resolution splits by role:

- an owner manager applies or declines it;
- the author archives their own withdrawn proposal;
- reopening a declined Suggestion is the owner manager's call, and reopening an archived one is the author's.

To apply:

1. read the Suggestion, base Version, and current latest Version;
2. decide whether the proposal still fits latest;
3. publish a new Version through an authorized human-reviewable surface;
4. resolve the Suggestion as applied, naming that Version as the result.

Applying requires a result Version that is newer than the base Version of the same Playbook, so publish first and resolve second. Never mark applied before the corresponding Version exists. Folding several open Suggestions into one Version is reasonable — stage the merged content in a Draft (see [Author](./author.md)) and publish it once, then resolve each Suggestion against that same result Version. A Draft is not itself a Suggestion and never substitutes for one: it holds the owner's own in-progress edit, not a reviewable third-party proposal.

## Close the loop

Report back to the user which Suggestions were applied, declined, or left open, and which Case evidence supports each one. When a Suggestion is declined, note the reason with the Case Record it came from so the same observation is not re-filed.
