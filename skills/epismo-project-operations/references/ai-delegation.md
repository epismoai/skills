# AI Delegation

This document defines how to design and assign high-quality AI-owned tasks.
Use it when you create or update delegated work and need clear outputs, acceptance criteria, and review gates.
For overload and rebalance analysis, use [Project Operations Runbook](./project-operations-runbook.md) (`Mode Playbooks > Recovery`).

## Delegation Fit Check

Delegate when work is:

1. Repetitive or template-driven.
2. Analysis/synthesis heavy.
3. Draft-oriented and reviewable.

Keep human owners for final approval, external communication, and sensitive decisions.

## Delegation Quality Checks

Before assigning AI-owned work, check:

1. Objective clarity
   - Is the task outcome specific and testable?
2. Output specificity
   - Is output shape/path/format clear enough to verify completion?
3. Acceptance criteria
   - Is there an objective done condition beyond "looks good"?
4. Input and boundary definition
   - Are source-of-truth inputs and non-goals explicit?
5. Dependency and timing clarity
   - Are required prerequisites and review timing clear?

## Delegation Workflow

1. Resolve assignee from `references.assignees`.
2. Create or update task with explicit quality checks satisfied.
3. If no AI assignee exists, assign human fallback and state reason.
4. Report assignment with title, assignee, expected output, and review timing.

## Typical Failure Modes

1. Objective unclear -> output drifts and rework increases.
2. Output shape missing -> reviewers cannot quickly validate completion.
3. Acceptance criteria missing -> premature "done" status.
4. Source inputs/non-goals missing -> hallucinated assumptions or scope creep.
5. Dependency timing unclear -> blocked tasks or idle waiting.
6. Reviewer missing -> sensitive decisions bypass human gate.
