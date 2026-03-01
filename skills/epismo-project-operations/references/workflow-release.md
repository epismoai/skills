# Workflow Release

This document defines release/update/deprecate decisions and approval boundaries for reusable workflows.
Use it when deciding or executing release actions, including release, update, keep private, and deprecate.

## Pre-Decision Checks

1. Action type is explicit (`release`, `update`, `deprecate`).
2. Target workflow is explicit (ID/title confirmed).
3. Release target is explicit (`private` or `public`).
4. Quality gate handling by action:
   - `release` / `update`: quality gate passed in [Workflow Quality](./workflow-quality.md).
   - `deprecate`: quality gate pass is not required; include deprecate rationale and risk note.
5. Duplication risk checked (no clearly superior equivalent exists).
6. Write scope is explicit (`private` uses confirmed `projects`; `public` omits `projects`).

## Approval Boundary

Private staging is optional. Use it only when it improves review safety/clarity.

1. Private staging writes for review preparation do not require explicit approval.
2. Explicit approval is required before `visibility="public"`.
3. Explicit approval is required for deprecate/removal.
4. Explicit approval is required for major structural updates to already released workflows.

## Decision Logic

1. `release`: proven, reusable, and non-duplicative.
2. `update`: improves an existing released pattern.
3. `keep private`: evidence is incomplete or constraints are project-specific.
4. `deprecate`: obsolete, unsafe, or superseded.

## Required Output Fields

1. Decision and rationale.
2. Risks and mitigations.
3. User approval status.
4. Next review timing.
5. Publication boundary confirmation (internal release metadata is not embedded in published workflow content).
