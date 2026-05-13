---
name: scope
description: How to audit a Jira project or epic for gaps, unclear issues, and missing work. Used by /scope.
---

# Scope Audit

How to look at a project's issues and find what's missing, unclear, or out of order.

## Mindset

You're not padding the backlog — you're finding things that will surprise the team mid-sprint. The test for whether a gap is worth raising: would discovering it mid-implementation block progress or change the plan?

Ask before creating anything. Present findings; let the user decide what to act on.

## What to look for

### Unclear issues
An issue is unclear if someone picking it up tomorrow couldn't start without asking questions. Signs:
- Summary is vague ("improvements", "refactor", "update X")
- No description, or description is just the summary repeated
- No acceptance criteria — how do you know it's done?
- No story point estimate

### Scope gaps
Work that's implied by the project goals but not tracked anywhere. Sources:
- `product.md` mentions a feature or requirement with no corresponding issue
- An issue's acceptance criteria mentions a dependency that has no ticket
- An epic has a logical step clearly missing (e.g., "design" and "deploy" but no "implement")
- Integration points mentioned in one issue that another team/system would need to handle

### Issues not in any sprint
Open issues that haven't been added to a sprint tend to get forgotten. Either pull them into the active sprint or confirm they're intentionally deferred to the backlog.

### Stale in-progress
Issues that have been "In Progress" for an unusually long time relative to their estimate. May be blocked, abandoned, or just forgotten. Worth a conversation.

### Oversized issues
L/XL issues (or Epics being treated as a single unit of work) that haven't been broken into child issues. These are planning liabilities — hard to schedule, easy to underestimate. Flag them with a suggestion to `/split`.

## How to assess gaps

Read `product.md` first. Without it, you're guessing at intended scope.

For each epic or sprint: trace through the work like a user journey or a data flow. What needs to happen for this epic to deliver its stated goal? Is there a child issue for each step?

Don't invent requirements. If you're unsure whether something is missing or just out of scope, say so and let the user decide.

## Presenting findings

Group by category and be specific — include issue keys, not just counts:

```
Unclear (3):
  PROJ-4: "Fix auth" — no description, no acceptance criteria
  PROJ-7: "Update dashboard" — unclear what "update" means
  PROJ-12: no description at all

Unestimated (2): PROJ-8, PROJ-11

Not in any sprint (2): PROJ-15, PROJ-16 — open but unscheduled

Potential gap:
  product.md §3 mentions email notifications for order status,
  but no issue covers this.

Oversized (1): PROJ-2 (XL) — consider splitting

Stale in-progress (1): PROJ-6 — In Progress for 14 days, estimated S
```

## Offering actions

For each finding, name the action but wait for confirmation:

- Unclear → "Want me to add a description/AC draft for PROJ-4?"
- Unestimated → "Run `/estimate` to work through these?"
- Not in sprint → "Add PROJ-15 and PROJ-16 to the active sprint?"
- Gap → "Worth creating an issue for email notifications? Tell me more and I'll draft it."
- Oversized → "Run `/split PROJ-2`?"
- Stale → "Is PROJ-6 still active? Should I check its status?"

## Related Skills

- **estimate** — for working through unestimated issues found during the audit
- **split** — for breaking down oversized issues
- **product-planning** — for adding new issues when genuine gaps are found
- **jira-cli** — for updating issues and adding them to sprints
