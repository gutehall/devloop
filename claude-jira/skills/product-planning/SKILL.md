---
name: product-planning
description: Facilitate product thinking and structure work in Jira.
---

# Product Planning

Help users think through product ideas and structure them as actionable work in Jira.

## Mindset

- **Thought partner, not ticket factory** - Understand before creating issues
- **Problem before solution** - What problem? For whom? How painful?
- **Default to smaller** - Cut scope ruthlessly, defer the non-essential
- **Incremental delivery** - Ship value early, learn, iterate

## Style

Like Jason Fried without swearing. Concise, simple wording. Short back-and-forth dialog over long answers. Casual.

## Start: Gather Context

```bash
jira board list --plain                               # Find the active board
jira sprint list --board-id <id> --plain              # Active and upcoming sprints
jira issue list --resolution Unresolved --plain       # All open work
jira epic list --plain                                # Existing epics
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

**Sizing:** XS/S/M = single Story or Task. L/XL = needs breakdown into an Epic with child issues.

```bash
# Epic (the goal)
jira epic create -s"User auth system" -b"Description"

# Stories / Tasks (the steps) as children of the epic
jira issue create -tStory -s"Design auth flow" --parent PROJ-10
jira issue create -tStory -s"Implement login" --parent PROJ-10
jira issue create -tTask -s"Add session management" --parent PROJ-10
```

Link blockers explicitly:
```bash
jira issue link PROJ-12 PROJ-11 "blocks"    # PROJ-12 is blocked by PROJ-11
```

### 4. Organize

**Sprints** group the current iteration's work:
```bash
jira sprint list --board-id <id> --plain
jira sprint add --board-id <id> PROJ-12    # Add issue to active sprint
jira sprint add --board-id <id> PROJ-13
```

**Epics** group related stories across sprints:
```bash
jira epic create -s"Phase 2: Payments"
jira issue create -tStory -s"Add Stripe integration" --parent PROJ-20
```

### 5. Prioritize

Jira priority levels: Highest → High → Medium → Low → Lowest

```bash
jira issue edit PROJ-12 --priority High
jira issue edit PROJ-13 --priority Medium
```

For ordering within a sprint, use the board drag-drop in the Jira UI, or set priorities to reflect relative importance.

## product.md

`product.md` is the persistent product context file for a project. It lives at the project root and is read at the start of every planning session to avoid re-deriving context.

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
Planned epics or sprints and their goals.

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

Keep it short. It's a reference, not a wiki. If it grows beyond 1–2 pages, trim it.

## Session Summary

Track progress throughout the session, then summarize:

1. **Created** - Issues and epics with keys
2. **Decided** - Key decisions and rationale
3. **Deferred** - What we cut (and why)
4. **Next** - `jira issue list -s"To Do" -a"$(jira me)" --plain`

## Related Skills

- **jira-cli** — full Jira CLI command reference for creating and managing issues once planning is done.
- **quick-start** — when the user wants to start immediately without creating tickets first. Use instead of product-planning for small, focused, immediate work.
