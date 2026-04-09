# Content Templates

Content structure templates for context packs.
For visibility settings, see [Visibility & Sharing](../references/visibility.md).
For finding existing packs, see [Search & Discovery](../references/search.md).

## What A Context Pack Should Optimize For

Context Pack exists to make context portable across tools, threads, and people.
That means the goal is not to force every pack into the same outline. The goal is to make stored context easy to scan, fetch selectively, and reuse later.

At minimum, a good pack does three things:

1. Makes it obvious what the pack is for.
2. Splits content into blocks that can be fetched independently.
3. Preserves the information someone would otherwise have to reconstruct by rereading a long thread, source, or document.

There is no universal section order. `Situation`, `Key Decisions`, and `Next Steps` are useful for handoffs, but they are not the default for every pack.

## How To Choose Blocks

Choose blocks based on the future reader's questions, not on a fixed document template.

Good reasons to split into separate blocks:

- The reader may want one part without loading the rest.
- One part changes often while another stays stable.
- The audience changes between sections.
- The pack mixes different jobs, such as summary, instructions, and references.

Useful block types:

| Block type                    | Use when the reader needs                             |
| ----------------------------- | ----------------------------------------------------- |
| `Overview` / `What This Is`   | fast orientation                                      |
| `Source`                      | metadata about a video, blog post, paper, or document |
| `Summary`                     | the core idea in compressed form                      |
| `Key Takeaways`               | scannable insights or claims                          |
| `Decisions` / `Constraints`   | durable rules that affect future work                 |
| `Next Steps` / `Action Items` | what to do next                                       |
| `Prompt` / `Instructions`     | reusable execution guidance                           |
| `References`                  | exact links, IDs, citations, and related packs        |

## Available Templates

### 1) Source Summary

Use when saving the essence of a YouTube video, blog post, paper, talk, or other source so it can be reused later without reopening the original.

```markdown
# [Source Title] — Summary

## Source

- Type: [YouTube / blog / paper / talk / doc]
- Author / channel: [Name]
- URL: [Link]
- Published: [Date if known]
- Why this matters: [Why you saved it]

## Summary

[A short paragraph with the main idea, argument, or contribution.]

## Key Takeaways

- [Insight 1]
- [Insight 2]
- [Insight 3]

## Reuse Notes

- [How this connects to your project, workflow, or decision]
- [What to borrow, test, or remember]

## References

- [Related pack, paper, repo, or note](URL)
```

**When to set visibility:** `private` for personal or team research. `public` if the summary is cleaned up for broader reuse and the source can be discussed openly.

---

### 2) Session / Thread Handoff

Use when moving work between tools, agents, or threads and needing to resume without rereading history.

```markdown
# Session Handoff — [Topic] — [Date]

## Context

[What this workstream is, where it was happening, and why it matters.]

## Decisions

- [Decision 1 with rationale]
- [Decision 2 with rationale]

## Work Done

- [Completed action 1]
- [Completed action 2]

## Next Steps

- [ ] [Immediate next action — owner: me/team]
- [ ] [Follow-up action]

## Open Questions

- [Unresolved point]

## References

- [Related pack, track, PR, or doc](URL)
```

**When to set visibility:** `private`. This is usually working memory or an internal handoff.

---

### 3) Project Memory / Working Context

Use when storing durable project knowledge that new sessions, contributors, or agents need repeatedly.

```markdown
# [Project Name] — Working Context

## Purpose

[What this project or workstream is trying to achieve.]

## Current Shape

- [Current architecture, plan, or state]
- [What stage the project is in]

## Constraints

- [Technical, product, legal, or team constraint]
- [Non-goal or boundary]

## Decisions

- [Important decision and why]
- [Important decision and why]

## References

- [Spec, repo, design, track, or related context pack](URL)
```

**When to set visibility:** `private` for team/project memory. `public` only if the content is intentionally shareable outside the workspace.

---

### 4) Task Handoff

Use when transferring an active task to another person or agent with enough context to continue immediately.

```markdown
# Handoff — [Task Title]

## What This Is

[One sentence describing the task and how it fits into the larger project.]

## Current State

- Status: [in_progress / blocked / review needed]
- Last updated: [Date]
- Track / PR / issue: [ID or link]

## What Was Done

- [Completed step 1]
- [Completed step 2]

## What Remains

- [ ] [Next action]
- [ ] [Following action]

## Important Context

- [Constraint, decision, or caveat]
- [Dependency, credential location, or environment detail]

## References

- [Relevant document, pack, or ticket](URL)
```

**When to set visibility:** `private`, usually scoped to the relevant project.

---

### 5) Prompt / Instruction Pack

Use when saving a prompt pattern, operating instructions, or reusable execution contract.

```markdown
# [Prompt or Workflow Name]

## When To Use This

- [Trigger or situation]
- [What problem it solves]

## Inputs

- [Required input 1]
- [Optional input 2]

## Instructions

[The reusable prompt, checklist, or operating procedure.]

## Output Expectations

- [Expected format]
- [Acceptance criteria]

## Variations

- [How to adapt this for a different tool, model, or audience]
```

**When to set visibility:** `private` for personal/team workflows. `public` if it is a polished pattern intended for community reuse.

---

### 6) Public Guide / Best Practice

Use when publishing reusable knowledge for a broader audience.

```markdown
# [Guide Title]

## What This Covers

[The problem this guide solves and who it is for.]

## When To Use It

- [Condition 1]
- [Condition 2]

## Approach

- [Core principle 1]
- [Core principle 2]

## Steps

1. [Step 1]
2. [Step 2]
3. [Step 3]

## Examples

- [Example, scenario, or before/after]

## References

- [Related guide, source, repo, or tool](URL)
```

**When to set visibility:** `public`. Choose `category` carefully because it affects discovery.

---

## Minimal Upsert Payload

Use this as the smallest safe starting shape. Add optional fields only when needed.

**Key rule:** Block content belongs in `"blocks[]"`, not embedded in the top-level `"content"` string. The top-level `"content"` is a short intro or summary only. If you put all content in `"content"`, blocks will appear empty when fetched.

### Private source summary

```json
{
  "title": "Attention Is All You Need - Summary",
  "content": "Summary of the Attention Is All You Need paper.",
  "type": "context",
  "visibility": "private",
  "projects": ["pj_123"],
  "blocks": [
    {
      "title": "Source",
      "content": "- Type: paper\n- URL: https://arxiv.org/..."
    },
    {
      "title": "Summary",
      "content": "..."
    },
    {
      "title": "Key Takeaways",
      "content": "- ..."
    }
  ]
}
```

### Public guide

```json
{
  "title": "How To Hand Off Work Between AI Tools",
  "content": "A guide for moving context across tools, threads, and agents.",
  "type": "context",
  "visibility": "public",
  "category": "productivity",
  "blocks": [
    {
      "title": "What This Covers",
      "content": "..."
    },
    {
      "title": "Steps",
      "content": "1. ..."
    }
  ]
}
```

### Updating existing blocks

Pass `"id"` on each block to update it in place. Blocks without `"id"` are added as new. Blocks omitted from the array are removed.

```json
{
  "id": "<pack-id>",
  "blocks": [
    { "id": "b001", "title": "Source", "content": "<updated content>" },
    { "id": "b002", "title": "Summary", "content": "<updated content>" },
    { "title": "New Block", "content": "<new block content>" }
  ]
}
```

> Note: `projects[]` is valid only when `visibility="private"`. Omit it for public packs.
