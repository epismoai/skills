# Workflow Quality

This document is the quality gate for reusable workflows.
Use it whenever reusable workflow intent exists, before deciding whether to release, update, or keep private.
Use [Workflow Release](./workflow-release.md) for final action choice and approval handling.

## Required Release Criteria (Go/No-Go)

All criteria must pass.

1. Proven execution
   - Prefer at least one real execution path with observable outcome.
   - Accept either:
     - Project-record evidence (`task`/`goal`/`note`) with concrete identifiers or links, or
     - Seed-source evidence (blog/docs/chat design rationale), optionally strengthened by focused web/tool references, with explicit assumptions and limits.
2. Reusability
   - Steps avoid project-specific names, IDs, and one-off constraints.
3. Structural integrity
   - Step IDs are unique.
   - `dependsOn` and `parentId` references are valid.
   - No self-dependency.
   - Independent steps are not unnecessarily serialized; parallelizable work is expressed with minimal valid `dependsOn`.
4. Actionability
   - Each step has a clear verb and deliverable.
   - Sequence can be followed by another team without hidden assumptions.
5. Safe defaults
   - Human assignment is possible when agent assignment is unavailable.
6. Reproducibility evidence
   - Workflow overview and each step include enough notes for replay by another team.
   - Tooling readiness and external references are included when relevant.
   - Prompt examples are added where they materially improve repeatability.
   - Key replay hints are grounded in observed project history, not only author assumptions.
   - Add supporting links when available (source code, official docs, design docs, blog posts).
7. Publication boundary
   - Published workflow content excludes internal release metadata (release target/intent, gate decision, approvals).
   - Release metadata is captured outside published workflow fields.
8. Review traceability
   - Release decision rationale is explicit and reviewable.
   - Ownership/dependency changes are explicitly justified.

## Release Decision

- If all criteria pass, hand off to [Workflow Release](./workflow-release.md).
- If evidence is limited or seed-only, prefer private scope with explicit risk notes.

## Workflow Content Review Guide (Recommended)

Use this as a flexible review guide when preparing workflow content for publication.
Do not force a fixed outline. Keep the author's style and structure when it is clear and reusable.
This guide improves quality but does not override Go/No-Go criteria above.

Scope:

- Applies to published workflow content fields only: `title`, `content`, step `title`, step `content`.
- Do not include internal release metadata such as release target, release intent, gate result, approval state, or decision trace.

### Recommended Content Review Checklist

Use `pass`/`needs work` per item.

1. Workflow overview clarity
   - A reader can understand what the workflow does and when to use it.
2. Preconditions and constraints
   - Required context/tools/inputs and important safety constraints are present.
3. Step reproducibility
   - Steps include enough memo/context to execute without hidden assumptions.
4. Adaptability
   - Key tuning points are discoverable (where to customize by team/project).
5. Example guidance
   - Prompt examples are included where they improve reliability (optional, but recommended).
6. Supporting references
   - Relevant links are included to strengthen reproducibility (source code, docs, blog posts).

### Field Constraints (Short)

1. `visibility`: `public` or `private`.
2. `category`: `""`, `productivity`, `learning`, `programming`, `design`, `marketing`, `operations`, `life`.
3. `content` / `workflow[].content`: Markdown text.
4. `workflow[].dueDate`: day-offset numeric string (digits only) or empty string.
