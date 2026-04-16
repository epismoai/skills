# Track Operations

Structured report templates for project tracking writes.
Use these after mode selection, validation, and approval checks are already complete.
For pre-write checks, approval gates, and execution playbooks, use the [Runbook](../references/runbook.md).

Replace all `{...}` placeholders before returning output.

---

## 1) Partial Update

Use when: existing entities need localized field edits only.

### Write Shape

- Entity type: `{task|goal}`
- Target: `{title or id}`
- Changed fields: `{field -> value}`
- Structural changes: `none`

### Report

- **Situation**: {what is active now}
- **Delta**: {fields changed on which entities; newly unblocked tasks / still blocked tasks / no downstream dependents}
- **Evidence**: {source that informed the change}
- **Risks**: {none expected / any concern}
- **Next action**: {smallest useful step}

---

## 2) New Item Creation

Use when: exactly one new task or goal must be added and existing structure is valid.

### Write Shape

- Entity type: `{task|goal}`
- Title: `{item title}`
- Owner: `{assignee or unassigned}`
- Due date: `{date or none}`
- Context: `{1-2 line description}`
- Links: `{goalId / parentId / dependsOn or none}`

### Report

- **Situation**: {what is active now}
- **Delta**: {created item with key fields}
- **Evidence**: {why this item was needed}
- **Risks**: {none expected / any concern}
- **Next action**: {smallest useful step}

---

## 3) Large-Scale Planning

Use when: multiple items, a new goal structure, or a coordinated multi-step plan is needed.

### Write Shape

| Step | Title   | Owner      | Output        | Depends on         |
| ---- | ------- | ---------- | ------------- | ------------------ |
| A    | {title} | {AI/human} | {deliverable} | {step IDs or none} |
| B    | {title} | {AI/human} | {deliverable} | {step IDs or none} |

- Materialization: {tracks only / tracks + workflow pack}
- Write method: {`apply track` (preferred for cross-referenced batches) / sequential `create track`}

### Report

- **Situation**: {what is active now}
- **Delta**: {items created/updated with structure summary}
- **Evidence**: {tracks, packs, or sources that informed the plan}
- **Risks**: {open questions or dependency concerns}
- **Next action**: {smallest useful step}

---

## 4) Recovery and Backlog Hygiene

Use when: delivery is slowed by overload, stall, dependency blocking, or backlog noise.

### Write Shape

- Recovery trigger: `{stall / overload / deadline risk / backlog cleanup}`
- Queue snapshot: `{backlog / todo / in_progress / blocked}`
- Throughput fixes: `{reassign / resequence / dependency rewiring}`
- Backlog cleanup: `{merge / archive / close}`

### Report

- **Situation**: {queue state after changes}
- **Delta**: {what was reassigned, resequenced, archived, or closed}
- **Evidence**: {metrics and signals that drove the decision}
- **Risks**: {remaining bottlenecks or concerns}
- **Next action**: {smallest useful step}
