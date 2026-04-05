# Release

Release, update, and deprecate decisions for reusable workflows.
Quality gate must pass before any `release` or `update` — run [Quality Gate](./quality.md) first.
For visibility and approval rules, see [Visibility & Sharing](./visibility.md).
Surface conventions from [Epismo Basics](../../epismo-basics/SKILL.md).

## Pre-Decision Checks

Confirm all six before proceeding:

1. **Action type** is explicit: `release`, `update`, or `deprecate`.
2. **Target workflow** is confirmed (ID or title).
3. **Release target** is explicit: `private` or `public`.
4. **Quality gate status**:
   - `release` / `update`: quality gate passed per [Workflow Quality](./quality.md).
   - `deprecate`: quality gate not required; deprecation rationale and risk note are sufficient.
5. **Duplication risk** checked — no clearly superior equivalent already exists.
6. **Write scope** confirmed: `private` uses confirmed `projects[]`; `public` omits `projects[]`.

## Approval Boundary

| Action                                                  | Requires explicit approval |
| ------------------------------------------------------- | -------------------------- |
| Private staging write (review preparation)              | No                         |
| `visibility="public"` (first release or re-release)     | **Yes**                    |
| Deprecate or remove                                     | **Yes**                    |
| Major structural update to an already-released workflow | **Yes**                    |

## Decision Logic

| Decision       | When to choose                                                                            |
| -------------- | ----------------------------------------------------------------------------------------- |
| `release`      | Execution is proven, structure is reusable, and no superior equivalent exists             |
| `update`       | Improves an existing released pattern without breaking its consumers                      |
| `keep private` | Evidence is incomplete, constraints are project-specific, or quality gate has open items  |
| `deprecate`    | Workflow is obsolete, unsafe, superseded by a better alternative, or no longer maintained |

## Required Output

After every release action, report:

1. **Decision** — which action was taken and why.
2. **Risks** — what could go wrong and how it is mitigated.
3. **Approval status** — approved, pending, or rejected.
4. **Publication boundary** — confirm internal release metadata is excluded from published content.
5. **Next review** — when this workflow should be re-evaluated (date, trigger, or milestone).
