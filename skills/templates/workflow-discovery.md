# Workflow Discovery

This document provides a template for finding and adapting reusable workflow patterns.
Use it when a user needs candidate options and a concrete adaptation plan for a real project.

```markdown
### Goal

- Desired outcome: <what the user wants to achieve>
- Project context: <team/project constraints>

### Evidence Sources

- Project evidence: <goals/tasks/notes/workflows checked>
- Workflow evidence (`private`/`liked`/`public`): <candidates checked>
- External evidence: <web/tools sources checked or N/A>

### Discovery Search Plan

- Filter-first scope:
    - Project entities (tasks/goals/notes): <projects/status/date filters>
    - Workflow entities: <visibility/like/category filters>
- Optional keyword query set (only when needed):
    - Outcome keywords: <2-6 words or N/A>
    - Method keywords: <2-6 words or N/A>
    - Risk/blocker keywords: <2-6 words or N/A>

### Candidate Shortlist

1. <pattern title>
   - Source: <project evidence|workflow(private/liked/public)|tools|web>
   - Why relevant: <reason>
   - Reuse cost: <low|medium|high>
   - Gaps to adapt: <list>
2. <pattern title>
   - Source: <project evidence|workflow(private/liked/public)|tools|web>
   - Why relevant: <reason>
   - Reuse cost: <low|medium|high>
   - Gaps to adapt: <list>

### Recommendation

- Selected candidate: <title + source>
- Why this one: <brief comparison>

### Adaptation Plan

- Keep as-is:
    - <steps>
- Modify:
    - <steps and changes>
- Add project-specific steps:
    - <steps>

### Materialization

- Apply as: <project tasks|private workflow (project-scoped)|both>
- Apply write safety/approval gate from [Project Operations Runbook](../references/project-operations-runbook.md) before materialization.
- If the action includes workflow release/update/deprecate, apply approval boundary from [Workflow Release](../references/workflow-release.md).
- Ownership plan:
    - AI-owned: <steps>
    - Human-owned: <approval/sensitive/external steps>

### User Report

- Candidate summary: <short>
- Then use `Standard User Report Block` from `templates/operating-templates.md`.
```
