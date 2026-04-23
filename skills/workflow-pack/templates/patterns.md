# Workflow Patterns

Structured report templates for workflow pack operations.
Use these after discovery, quality, approval, and release checks are already complete.
For internal validation, use [Quality Gate](../references/quality.md) and [Workflow Release](../references/release.md).

Surface conventions from [Epismo Basics](../../epismo-basics/SKILL.md).

---

## 1) Pattern Capture and Workflow Release

Use when: a successful execution pattern is a candidate for reuse across projects or teams.

### Workflow Shape

- Pattern: {title or summary}
- Generalizable structure: {steps, owners, dependencies}
- Release target: {private / public}
- Decision: {release / update / keep private / deprecate}

### Report

- **Situation**: {workflow pack state after action}
- **Decision**: {release action taken and why}
- **Evidence**: {execution records and quality gate results}
- **Risks**: {duplication risk, scope limitations, or open concerns}
- **Approval status**: {approved / pending / rejected / not required}
- **Publication boundary**: {confirm internal release metadata is excluded from published content}
- **Next action**: {next review timing or follow-up}

---

## 2) Workflow Discovery

Use when: comparing candidate workflows and committing to a concrete adaptation plan before materialization.

### Discovery Record

- Desired outcome: {what the user wants to achieve}
- Project context: {team and project constraints}
- Search plan:
  - Tracks: {targets.projectIds / status / date filters used}
  - Workflows: {visibility / like / category filters used}
  - Keyword queries: {2-6 domain keywords or none}

Candidate shortlist:

| #   | Pattern | Source        | Relevance      | Reuse cost     | Gaps            |
| --- | ------- | ------------- | -------------- | -------------- | --------------- |
| 1   | {title} | {source type} | {why relevant} | {low/med/high} | {what to adapt} |
| 2   | {title} | {source type} | {why relevant} | {low/med/high} | {what to adapt} |

### Recommendation

- Selected: {title + source}
- Why: {brief comparison rationale}

### Adaptation Plan

- Keep as-is: {steps}
- Modify: {steps and changes}
- Add project-specific: {steps}
- Ownership: AI-owned {steps} / Human-owned {steps}
- Destination: {project tracks / private workflow pack / both}
  - If tracks: use `apply track` with step IDs as client labels — `dependsOn` cross-references resolve in one request

### Report

- **Situation**: {candidates found and recommendation}
- **Delta**: {adaptation plan or materialized items}
- **Evidence**: {discovery sources and comparison rationale}
- **Risks**: {gaps in adaptation, missing context}
- **Next action**: {smallest useful step}
