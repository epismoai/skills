# AI Delegation

Design and assignment standards for AI-owned tasks.
Use when creating or updating delegated work that requires clear outputs, acceptance criteria, and review gates.
For overload and rebalance analysis, use [Runbook — Recovery](./runbook.md#4-recovery-and-backlog-hygiene).
Surface conventions from [Epismo Basics](../../epismo-basics/SKILL.md).

## When to Delegate

**Delegate when work is:**

- Repetitive or template-driven (data extraction, formatting, reporting)
- Analysis, synthesis, or research-heavy (comparison tables, summaries, audits)
- Draft-oriented and reviewable by a human afterward (first drafts, proposals, outlines)

**Keep human owners for:**

- Final approval and sign-off
- External communication (client-facing, public-facing)
- Sensitive or high-stakes decisions (budget, hiring, legal)
- Work requiring real-time judgment or negotiation

**When not to delegate:**
If the task cannot be verified by inspecting its output alone, it should not be AI-owned. A human reviewer who cannot tell whether the output is correct without redoing the work means the task is a poor delegation candidate.

## Pre-Assignment Checklist

All six must pass before assigning AI-owned work. If any item fails, fix the gap or assign to a human.

- [ ] **Objective** — outcome is specific and testable
- [ ] **Output shape** — format, path, or structure is clear enough to verify completion
- [ ] **Done condition** — acceptance criteria are objective, not "looks good"
- [ ] **Inputs and boundaries** — source-of-truth inputs and non-goals are explicit
- [ ] **Dependencies and timing** — prerequisites and review timing are unambiguous
- [ ] **Reviewer** — a human reviewer is assigned for sensitive or external-facing output

## Good vs. Bad Delegation

| Dimension      | Good                                                                                    | Bad                      |
| -------------- | --------------------------------------------------------------------------------------- | ------------------------ |
| Objective      | "Generate competitor pricing analysis for products A, B, C using their public websites" | "Research competitors"   |
| Output shape   | "Markdown table saved to notes/competitor-pricing.md"                                   | "A summary"              |
| Done condition | "Table covers all 3 products: price, tier name, and key features"                       | "When it looks complete" |
| Non-goals      | "Exclude integrations and enterprise plans"                                             | (not stated)             |
| Review timing  | "Review before sharing with sales team on Friday"                                       | (not stated)             |
| Inputs         | "Use public pricing pages listed in notes/competitor-urls.md"                           | "Find the information"   |

## Delegation Workflow

1. **Resolve assignee** — look up AI assignee from `references.assignees`. If no AI assignee exists, assign a human fallback and state the reason.
2. **Validate checklist** — confirm all six items pass. Document any gaps filled during validation.
3. **Create or update task** — write the task with all six checklist items satisfied. Include the expected output description in the task content.
4. **Report** — confirm: title, assignee, expected output, done condition, and review timing.

## Failure Modes

| Failure                           | Effect                                  | Prevention                                         |
| --------------------------------- | --------------------------------------- | -------------------------------------------------- |
| Unclear objective                 | Output drifts; rework increases         | Write a testable outcome statement                 |
| Output shape missing              | Reviewers cannot validate completion    | Specify format, path, or structure                 |
| Acceptance criteria missing       | Premature "done" status                 | Define objective done conditions                   |
| Source inputs / non-goals missing | Hallucinated assumptions or scope creep | List inputs explicitly; state what is out of scope |
| Dependency timing unclear         | Blocked tasks or idle waiting           | Set prerequisites and review dates                 |
| Reviewer missing                  | Sensitive decisions bypass human gate   | Always name a reviewer for external-facing work    |
