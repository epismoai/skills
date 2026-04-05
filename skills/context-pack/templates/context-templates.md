# Templates

Content structure templates for context assets.
For visibility settings, see [Visibility & Sharing](./visibility.md).
For finding existing assets, see [Search & Discovery](./search.md).

## Anatomy of a Good Context Pack

A useful context asset answers these questions in order, with no wasted words:

```
# [Title — specific enough to find by search]

## Situation
What is happening right now, and why does the reader need this document?
One or two sentences. Include dates or version markers if they matter.

## Key Decisions / Rules
- Bullet each decision or rule on its own line.
- Include the rationale if it is not obvious.
- Name the decision-maker or date if it affects future choices.

## Next Steps / Open Questions
- What the reader must do next.
- What is unresolved and who owns it.

## References
- [Link title](URL or epismo://asset-id) — one-line description of what it contains.
```

Omit any section that has nothing to say. Add sections only when the template above does not cover the use case (e.g., a best-practice guide may need an "Examples" section).

## Content Templates

### 1) Thread/Session Summary

Use when moving work from one tool or session to another and needing to resume without re-reading history.

```markdown
# Session Summary — [Topic] — [Date]

## Situation
[What was being worked on and why. Include the tool, thread, or context being left behind.]

## Key Decisions
- [Decision 1 with rationale]
- [Decision 2 with rationale]

## Work Done
- [Completed action 1]
- [Completed action 2]

## Next Steps
- [ ] [Immediate next action — owner: me/team]
- [ ] [Follow-up — owner: me/team]

## Open Questions
- [Question 1 — unresolved as of [date]]

## References
- [Related asset or track title](URL)
```

**When to set visibility:** `private`. This is personal working memory; make public only if it documents a shared decision.

---

### 2) Status Report

Use when giving teammates or stakeholders a snapshot of current project state.

```markdown
# Status Report — [Project Name] — [Date]

## Summary
[One paragraph: overall health, key progress, and the single most important risk or blocker.]

## Progress
| Item | Status | Owner | Notes |
| ---- | ------ | ----- | ----- |
| [Goal/Task title] | [On track / At risk / Done] | [Name] | [Brief note] |

## Blockers
- [Blocker 1 — what it is, who can unblock it, by when]
- [Blocker 2]

## Decisions Needed
- [Decision 1 — what is needed, from whom, deadline]

## Next Review
[Date or trigger for next status update]
```

**When to set visibility:** `private` for internal updates. `public` only if explicitly shared with an external audience.

---

### 3) Project Rules/Plan

Use when recording team agreements, norms, or a structured plan that everyone needs regardless of which tool they use.

```markdown
# [Project Name] — Rules & Plan

## Purpose
[Why these rules exist and who they apply to.]

## Rules
1. [Rule 1 — specific and actionable]
2. [Rule 2]
3. [Rule 3]

## Plan
| Phase | Goal | Owner | Target date | Status |
| ----- | ---- | ----- | ----------- | ------ |
| [Phase 1] | [Goal] | [Owner] | [Date] | [Status] |

## Exceptions & Edge Cases
- [When rule X does not apply: condition]

## Change Log
- [Date] — [What changed and why]
```

**When to set visibility:** `private` for internal rules; `public` for community-facing norms or open-source project standards.

---

### 4) Task Handoff

Use when transferring a task to someone else with all context they need to continue without interruption.

```markdown
# Handoff — [Task Title]

## What This Is
[One sentence: what the task is and where it sits in the larger project.]

## Current State
- Status: [in_progress / blocked / review needed]
- Last updated: [Date]
- Epismo track ID: [id] (or link)

## What Was Done
- [Completed step 1]
- [Completed step 2]

## What Remains
- [ ] [Next action — specific enough to start immediately]
- [ ] [Following action]

## Known Issues / Blockers
- [Issue 1 — description, severity, and suggested resolution]

## Context the Recipient Needs
- [Key decision or constraint that affects how to proceed]
- [Relevant tool, credential, or resource location]

## Questions for the Recipient
- [Question 1 — what they should decide or clarify]
```

**When to set visibility:** `private` and scoped to the project. Share the asset URL or import it into the recipient's project.

---

### 5) Best-Practice Guide

Use when documenting a reusable pattern or approach to publish for the broader community.

```markdown
# [Guide Title] — [Domain]

## What This Covers
[One paragraph: the problem this guide solves and who it is for.]

## When to Use This Pattern
- [Condition 1]
- [Condition 2]

## When NOT to Use This Pattern
- [Anti-pattern or exclusion condition]

## Step-by-Step
1. [Step 1 — actionable, specific]
2. [Step 2]
3. [Step 3]

## Examples
### [Example title]
[Concrete example — show inputs, outputs, or before/after]

## Common Mistakes
- [Mistake 1 — what goes wrong and how to avoid it]

## References
- [Related guide or tool](URL)
```

**When to set visibility:** `public`. This type of asset is meant for broad discovery.
Choose `category` carefully — it affects how people find this guide in search.

---

## Minimal Upsert Payload

Use this as the smallest safe starting shape. Add optional fields only when needed.

### Private context asset

```json
{
  "title": "Session Summary — Auth Refactor — 2026-04-02",
  "content": "## Situation\nMoved from Claude to Cursor mid-session...",
  "type": "context",
  "visibility": "private",
  "projects": ["pj_123"]
}
```

### Public best-practice guide

```json
{
  "title": "How to Structure AI Task Handoffs in Epismo",
  "content": "## What This Covers\n...",
  "type": "context",
  "visibility": "public",
  "category": "productivity"
}
```

> Note: `projects[]` is valid only when `visibility="private"`. Omit it for public assets.

---

## Content Quality Checklist

Before calling `upsert asset`, verify:

- [ ] **Title is searchable** — includes the topic, project name, or date so it surfaces in a keyword search.
- [ ] **Situation is clear in the first two sentences** — a reader who opens the asset cold knows what it is about.
- [ ] **Every bullet is self-contained** — no bullets that say "see above" or "as discussed".
- [ ] **IDs and links are exact** — include Epismo track IDs, asset IDs, or URLs so the reader can act without searching.
- [ ] **Next steps have owners** — if someone must do something, name them or say "me".
- [ ] **Blocks are independently fetchable** — each block answers one distinct question on its own. Avoid mixing stable content (decisions, rules) with frequently-updated content (next steps, status) in the same block.
- [ ] **Content is under 1 500 words** — long assets lose readers. Split into multiple assets if needed.
- [ ] **Visibility matches the audience** — private for internal, public only with explicit approval.
- [ ] **Category is set for public assets** — omitting category reduces discoverability.
