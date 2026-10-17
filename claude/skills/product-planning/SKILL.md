---
name: product-planning
description: Product thinking and structuring work into Linear projects and issues. Only /plan, /next, or when a command names this skill. Do not trigger proactively.
---

# Product Planning

Help users think through product ideas and structure them as actionable work in Linear.

## Mindset

- **Thought partner, not ticket factory** - Understand before creating issues
- **Problem before solution** - What problem? For whom? How painful?
- **Default to smaller** - Cut scope ruthlessly, defer the non-essential
- **Incremental delivery** - Ship value early, learn, iterate

## Style

Like Jason Fried without swearing. Concise, simple wording. Short back-and-forth dialog over long answers. Casual.

## Start: Gather Context

```bash
linear roadmap        # Projects, milestones, progress
linear issues --open  # Active work
```

Read `product.md` if it exists — contains product vision, brand, tech decisions, prior planning context.

## Process

### 1. Explore the Problem

Before solutions:
- What problem? For whom? How painful?
- What job is the user hiring this product/feature to do?
- What happens if we don't solve it?
- What constraints? (time, tech, dependencies)
- What's uncertain or risky?

Proactively search the web when exploring unfamiliar territory (competitors, market, technical approaches).

### 2. Design the Solution

- What approaches exist? Tradeoffs?
- What's the riskiest assumption to test first?
- What's the **simplest version** that delivers value?
- What can wait for later?
- How will users discover and use this? What's prominent vs. hidden?

Cut scope aggressively — except for core/differentiating features where polish and detail set you apart.

For technical work, explore the existing codebase first to understand patterns and architecture.

### 3. Structure the Work

**Sizing:** XS/S/M = single issue, < 1 day. L/XL = needs breakdown.

Every issue must have: title, description, priority, and at least one label.

```bash
# Parent issue (the goal)
linear issue create --title "User auth system" \
  --description "Implement full auth flow so users can log in securely.\n\n**Acceptance criteria:**\n- Users can sign up, log in, and log out\n- Sessions expire after 30 days" \
  --priority medium --label feature --estimate L --project "Phase 2"

# Sub-issues (the steps)
linear issue create --title "Design auth flow" \
  --description "Diagram and document the sign-up/login/logout flow." \
  --priority medium --label feature --parent ISSUE-10 --estimate S
linear issue create --title "Implement login endpoint" \
  --description "POST /auth/login — validate credentials, issue session token." \
  --priority high --label feature --parent ISSUE-10 --estimate M
linear issue create --title "Add session management" \
  --description "Store and expire session tokens. Use Redis." \
  --priority medium --label feature --parent ISSUE-10 --blocked-by ISSUE-11 --estimate M
```

### 4. Organize

**Milestones** for phases within a project:
```bash
linear milestone create "Beta" --project "Phase 2" --target-date 2024-03-01
linear issue create --title "Core feature" --milestone "Beta" --estimate M
```

**Dependencies** make `--unblocked` useful:
```bash
linear issue create --title "Need API credentials" --blocks ISSUE-5
```

### 5. Order by Implementation Sequence

After creating issues, sort them in the order they should be implemented — not by priority alone, but by actual execution order (dependencies first, blocked work last).

```bash
linear issue move ISSUE-11 --before ISSUE-12   # design before build
linear issue move ISSUE-12 --before ISSUE-13   # build before deploy
```

Issues should appear in Linear in the sequence a developer would pick them up. Two issues independent → put the higher-value one first.

### 6. Prioritize Projects and Milestones

```bash
linear projects reorder "Phase 1" "Phase 2" "Phase 3"
linear milestones reorder "Alpha" "Beta" --project "Phase 2"
```

## product.md

`product.md` is the persistent product context file for a project. Lives at project root, read at the start of every planning session to avoid re-deriving context.

### When to create it

Create `product.md` if it doesn't exist after any substantive planning session that produces decisions worth remembering. Also create it when planning work in an unfamiliar existing codebase.

### Template

```markdown
# Product

## Vision
One sentence: what is this and who is it for?

## Problem
What problem does it solve? Who has this problem?

## Users
Who are the primary users? What do they care about?

## Architecture
Key technical decisions and why.

## Roadmap
Planned phases or milestones and their goals.

## Decisions
| Decision | Rationale | Date |
|----------|-----------|------|

## Deferred
Things explicitly cut from scope, and why.
```

### What to update after each session

- New decisions about scope, architecture, or product direction
- Items explicitly deferred (and why)
- Refined understanding of the user or problem

Keep it short. It's a reference, not a wiki. Grows beyond 1–2 pages → trim it.

## Session Summary

Track progress throughout the session, then summarize:

1. **Created** - Issues/milestones with IDs
2. **Decided** - Key decisions and rationale
3. **Deferred** - What we cut (and why)
4. **Next** - `linear issues --unblocked`

## Related Skills

- **linear-cli** — full Linear CLI command reference for creating and managing issues once planning is done.
