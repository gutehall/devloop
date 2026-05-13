---
name: quick-start
description: Plan and start work immediately without Jira ceremony. For tasks where thinking comes first and tickets come last (or never). Used by /work.
---

# Quick Start

Help users think through immediate work and start coding fast. No Jira required.

## Mindset

You're not planning a project — you're thinking before coding. The goal is: understand the problem, pick the approach, start. Five minutes of thinking, not a planning session.

Default to starting. The risk of over-thinking is higher than the risk of under-planning on small work.

## Flow

### 1. Frame the work (1–2 turns max)

Ask only what you need to start. If the input already answers these, skip straight to step 2.

- What should this do? (one sentence)
- Where in the codebase does it live? (or: explore briefly)
- Any constraints? (performance, backwards compat, API surface)

Read `product.md` if it exists. Grep for relevant symbols. Understand the affected area before proposing an approach.

### 2. Propose an approach

State it concisely — 3–5 sentences:
- What you'll build
- Which files/modules change
- Any tradeoffs or risks worth naming
- What you won't do (scope boundary)

Ask: "Good to go, or adjust?"

One round of feedback is fine. Don't iterate endlessly — if the scope is genuinely unclear, suggest `/plan` instead.

### 3. Start immediately

Once confirmed:

1. Create a branch with a short, descriptive name (no issue key needed):
   ```bash
   git checkout -b <short-descriptor>   # e.g. add-rate-limiting, fix-auth-redirect
   ```

2. Start implementing — minimal, focused, existing patterns. No speculative abstractions.

### 4. After implementation

When the work is done, offer two paths:

**Track it (if it grew or matters):**
> "Want to create a Jira issue for this and open a PR? Describe what you built and I'll create the ticket, then run `/done`."

**Ship it directly (if it's small):**
> "Or just push: `git push -u origin HEAD`, then open a PR with `gh pr create`."

Skipping Jira entirely is the right call for small, low-risk changes.

## Scope check

If implementation reveals the work is bigger than expected (L/XL scope):

Stop and say: "This is bigger than expected — [explain what you found]. Worth creating a Jira issue to track it properly? Or define a smaller slice we can finish now?"

Don't silently expand scope. Don't abandon the work either. Surface the decision.

## Rules

- No Jira calls unless the user asks
- Don't over-plan — start, and adjust as you learn
- If you can't find the right place in the code, grep first, ask second
- Keep branches short-lived — this work should ship quickly or be promoted to a proper issue

## Related Skills

- **product-planning** — when the work needs tickets and scheduling, not an immediate start
- **diagnostic** — when you're debugging rather than building something new
- **jira-cli** — if you decide to track the work in Jira after all
