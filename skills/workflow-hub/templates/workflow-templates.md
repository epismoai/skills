# Workflow Templates

Structured output templates for workflow asset operations.
Surface conventions from [Epismo Basics](../../epismo-basics/SKILL.md).

Guardrails:
- Quality gate: [Workflow Quality](../references/quality.md)
- Release decisions: [Workflow Release](../references/release.md)

---

## 1) Pattern Capture and Workflow Release

Use when: a successful execution pattern is a candidate for reuse across projects or teams.

### Assess

- [ ] Execution evidence exists (project records with identifiable outcomes)
- [ ] Reuse boundary defined: {where this pattern fits / does not fit}
- [ ] Quality gate result: {pass / needs work} — per [Workflow Quality](../references/quality.md)

### Plan

- Generalizable structure: {steps, owners, dependencies — project-specific residue removed}
- Release target: {private / public}
- Decision: {release / update / keep private / deprecate}

### Approve

- [ ] Approval status: {approved / pending / rejected}
- Private staging writes do not require approval; public release and deprecation do.

### Execute

- `release` or `update` + `private`: set confirmed `projects`
- `release` or `update` + `public`: omit `projects`
- `deprecate`: explicit approval required

### Report

- **Situation**: {workflow asset state after action}
- **Delta**: {release action taken with target visibility}
- **Evidence**: {execution records and quality gate results}
- **Risks**: {duplication risk, scope limitations, or open concerns}
- **Next action**: {next review timing or follow-up}

---

## 2) Workflow Discovery

Use when: comparing candidate workflows and committing to a concrete adaptation plan before materialization.

### Assess

- [ ] Desired outcome defined: {what the user wants to achieve}
- [ ] Project context noted: {team and project constraints}
- [ ] Evidence sources checked:
  - Tracks (goals/tasks): {what was checked}
  - Workflows (`private` / `liked` / `public`): {what was checked}
  - External sources: {what was checked or N/A}

### Discover

Search plan:
- Tracks: {projects / status / date filters used}
- Workflows: {visibility / like / category filters used}
- Keyword queries (if needed): {2-6 domain keywords}

Candidate shortlist:

| # | Pattern | Source | Relevance | Reuse cost | Gaps |
| - | ------- | ------ | --------- | ---------- | ---- |
| 1 | {title} | {source type} | {why relevant} | {low/med/high} | {what to adapt} |
| 2 | {title} | {source type} | {why relevant} | {low/med/high} | {what to adapt} |

### Recommend

- Selected: {title + source}
- Why: {brief comparison rationale}

### Adapt

- Keep as-is: {steps}
- Modify: {steps and changes}
- Add project-specific: {steps}
- Ownership: AI-owned {steps} / Human-owned {steps}

### Materialize

- Destination: {project tracks / private workflow asset / both}
- [ ] Write safety confirmed per [Runbook](../../project-tracking/references/runbook.md#write-safety-and-approval-gate)
- [ ] If workflow release involved, approval boundary confirmed per [Workflow Release](../references/release.md#approval-boundary)

### Report

- **Situation**: {candidates found and recommendation}
- **Delta**: {adaptation plan or materialized items}
- **Evidence**: {discovery sources and comparison rationale}
- **Risks**: {gaps in adaptation, missing context}
- **Next action**: {smallest useful step}
