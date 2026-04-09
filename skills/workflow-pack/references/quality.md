# Quality Gate

Quality gate for reusable workflows.
Run whenever release, update, or deprecate intent exists — before deciding which action to take.
For final action choice and approval handling, use [Release](./release.md).
For visibility and approval rules, see [Visibility & Sharing](./visibility.md).

## Go/No-Go Criteria

All criteria must pass. A single fail means: fix the issue or keep the workflow private.

1. **Proven execution** — at least one real execution path with observable outcome (tracks or linked records with identifiers), OR seed-source evidence (blog/docs/design rationale) with explicit assumptions and limits.
   Fail: outcome claimed but no records, links, or rationale provided.

2. **Reusability** — steps avoid project-specific names, IDs, and one-off constraints.
   Fail: steps reference specific team members, internal URLs, or non-generalizable configurations.

3. **Structural integrity** — step IDs are unique; `dependsOn` and `parentId` references resolve correctly; no self-dependency or circular references; parallelizable work uses minimal valid `dependsOn`.
   Fail: circular references, broken IDs, or unnecessary serialization.

4. **Actionability** — each step has a clear verb and deliverable; another team can follow the sequence without hidden assumptions.
   Fail: vague steps ("research X", "improve Y") with no concrete deliverable.

5. **Safe defaults** — human assignment is possible when agent assignment is unavailable.
   Fail: requires a specific AI agent with no documented human fallback.

6. **Reproducibility evidence** — overview and steps include enough context for another team to replay; prompt examples added where they improve repeatability; supporting links included when relevant.
   Fail: missing tool context, replay hints, or prompt examples where they would prevent rework.

7. **Publication boundary** — published content excludes internal release metadata (release target/intent, gate decision, approvals).
   Fail: internal notes, gate results, or approval state embedded in published fields.

8. **Review traceability** — release decision rationale is explicit and reviewable; ownership and dependency changes are justified.
   Fail: decision trail absent or vague.

## Step Authoring Rules

Apply these rules when designing workflow steps. They balance reproducibility with cognitive load.

1. Write detailed step content
   - Each step's `content` must include enough procedural detail for someone unfamiliar with the project to execute it.
   - Include: concrete actions, expected inputs/outputs, tool or command examples (commands, prompts, or configuration snippets), decision criteria, and links to references when relevant.
   - Avoid vague instructions like "handle X" or "set up Y" without explaining how.
2. Assign an owner to every step
   - Every step must have a concrete `assignee`. Leaving `assignee` empty is not acceptable.
   - Resolve assignee from `references.assignees`. Match by role or tool:
     - Human-driven steps (decisions, reviews, commits, coordination): use `human`.
     - AI-driven steps (generation, research, summarization, transformation): use the most appropriate AI agent by name (e.g., Claude Code for coding tasks, Perplexity for research, Gemini for analysis).
   - If the exact tool is not listed, assign the closest equivalent and note the assumption in the step content.
   - When a step is shared between human and AI, assign the party who owns the deliverable.
3. Right-size step granularity
   - Avoid splitting work into too many fine-grained steps; excessive steps increase cognitive load and tracking overhead.
   - Use assignee boundaries as the primary split heuristic: create a new step when the responsible person changes.
   - A workflow with 3-8 top-level steps is typical; exceed this only when assignee changes or hard dependencies require it.

## Release Decision

- All criteria pass → hand off to [Workflow Release](./release.md).
- Evidence limited or seed-only → prefer private scope with explicit risk notes.

## Content Review Guide

Use as a flexible review guide when preparing workflow content for publication. Do not force a fixed outline — keep the author's style when it is clear and reusable. This guide improves quality but does not override the Go/No-Go criteria above.

**Scope**: published content fields only — `title`, `content`, step `title`, step `content`. Exclude internal release metadata.

| Item                  | What to check                                                              |
| --------------------- | -------------------------------------------------------------------------- |
| Workflow overview     | A reader can understand what the workflow does and when to use it          |
| Preconditions         | Required context, tools, inputs, and safety constraints are present        |
| Step reproducibility  | Steps include enough context to execute without hidden assumptions         |
| Adaptability          | Key tuning points are discoverable (where to customize by team or project) |
| Example guidance      | Prompt examples or sample outputs included where they improve reliability  |
| Supporting references | Relevant links included to strengthen reproducibility                      |

## Field Constraints

1. `visibility`: `public` or `private`.
2. `category`: `""` (empty), `productivity`, `learning`, `programming`, `design`, `marketing`, `operations`, `life`. For descriptions, see [Visibility & Sharing — Category Reference](./visibility.md#category-reference).
3. `content` / `steps[].content`: Markdown text.
4. `steps[].dueDate`: day-offset numeric string (digits only, e.g. `"3"` means 3 days after start) or empty string.
