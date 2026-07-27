# Execute Work

Use this guide to plan, assign, update, review, or unblock current work.

## Model the Work

- Use a goal for an outcome whose progress matters.
- Use a task for a concrete action with a reviewable result.
- Connect tasks to a goal when they contribute to the same outcome.
- Add a dependency only when one task truly cannot proceed before another.
- Keep reusable instructions in a workflow pack; keep execution-specific facts in tracks and logs.

## Inspect Before Acting

Search current goals and tasks before creating new ones. Fetch the relevant records before deciding what should change.

When reviewing a project:

1. Identify active outcomes and work already in progress.
2. Separate actionable work from vague ideas.
3. Find blocked tasks and their real prerequisites.
4. Detect duplicates, abandoned items, missing owners, and impossible dates.
5. Preserve useful history rather than recreating equivalent tracks.

Use exact search for a known title or identifier. Use semantic search when the user describes intent in different words.

## Plan

Turn an outcome into the smallest useful structure:

1. Define the goal in observable terms.
2. Create tasks that each produce a reviewable result.
3. Set only dependencies that affect execution order.
4. Assign an owner when the responsible person or agent is known.
5. Add dates only when they represent a real constraint.

Prefer a connected bulk operation for several related tracks when the current surface supports it. Prefer a direct create or update for isolated changes.

Show a proposed structure before a broad change that would delete records, replace many relationships, or materially change ownership. Ordinary planning requested by the user does not need a second confirmation.

## Update and Communicate

Update the track when durable state changes: status, owner, dependency, goal, date, progress, title, or description.

Use a log for append-only communication:

- progress check-in;
- decision or observation;
- blocker explanation;
- review verdict.

Do not use a log instead of correcting track state. Do not rewrite a track merely to record a transient comment.

After a write, verify identifiers, relationships, status, and owner in the returned or re-fetched state.

## Unblock

For stalled work:

1. Find the earliest unresolved dependency or decision.
2. Distinguish a real blocker from missing information or an oversized task.
3. Split work only when each result can be reviewed independently.
4. Reassign only when a better owner is known.
5. Postpone or remove planned work only with explicit intent.
6. Record the reason for a non-obvious recovery decision.

Do not solve overload by moving every task or changing every date uniformly.

## Delegate to AI

Assign work to an AI agent only when its output can be reviewed without redoing the whole task.

Make these explicit:

- objective;
- expected output and location;
- acceptance criteria;
- source inputs and non-goals;
- dependencies and timing;
- human reviewer for sensitive or external-facing output.

Keep final approval, negotiation, legal judgment, hiring, budget decisions, and external communication with a human unless the user defines a safe review boundary.

Log meaningful AI check-ins and completion reviews on the task so the acceptance trail survives the chat.

## Review

Use the available review operation when completed or postponed work contains useful learning. Review related tracks together when they belong to the same outcome.

Treat review output as evidence, not as an automatic write:

- repeatable sequence or checklist → capture a workflow;
- durable decision, finding, or project knowledge → capture context;
- execution-only detail → keep it on the track or log.
