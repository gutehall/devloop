---
name: split
description: Decompose a large Jira issue into smaller, independently trackable child issues. Only /split, or when a command names this skill. Do not trigger proactively.
---

# Issue Splitting

How to break down an L or XL issue into child issues that can be worked on and tracked independently.

## Mindset

A well-split issue becomes invisible after splitting — child issues tell the full story, parent just holds them together. A bad split creates artificial boundaries that don't map to how the work actually flows.

Split along natural seams, not arbitrary size targets. Goal: child issues you could hand to someone with no further explanation.

## When to split

Split when:
- Issue is L or XL and approach isn't fully clear
- Description contains multiple distinct deliverables ("and also", "as well as")
- Acceptance criteria spans different systems or roles
- Issue has been sitting unstarted — probably too big to pick up

Don't split when:
- Issue is M but just feels vague — clarify the description instead
- Child tasks are too small to be worth tracking (< XS each)
- Work is highly sequential with no parallelism possible and one person will do it all — a checklist in the issue description is simpler

## How to find the seams

Work through issue description and acceptance criteria, then ask:

1. **By layer:** UI / API / data model / infra — natural split for full-stack features
2. **By phase:** research → design → implement → test — split when phases have different skill requirements or could be blocked
3. **By component:** feature A and feature B mentioned in the same issue — split them
4. **By risk:** risky, unknown part vs. mechanical, known part — split so risky part can be explored first
5. **By dependency:** what must be done before anything else? That's a natural first child issue.

## Structuring the breakdown

Each child issue needs:
- A clear summary that stands alone (not "Part 1" or "Step A" — say what it does)
- An estimate (XS–M; if it's L, split it further)
- Dependencies encoded with `jira issue link <child> <dependency> "blocks"` — don't rely on numbering or description text alone

Parent issue keeps original description and acceptance criteria. Add a comment naming child issues so anyone reading the parent knows where the work lives.

## Example decomposition

**Before:** `PROJ-12: Implement checkout flow — XL`

**After:**
```
PROJ-13: Design checkout data model — S
PROJ-14: Implement cart API endpoints — M  (blocked by PROJ-13)
PROJ-15: Build checkout UI — M            (blocked by PROJ-14)
PROJ-16: Add payment integration — M      (blocked by PROJ-14)
PROJ-17: Write E2E checkout tests — S     (blocked by PROJ-15, PROJ-16)
```

Notice:
- PROJ-15 and PROJ-16 can run in parallel once PROJ-14 is done
- Each has a clear deliverable and a reasonable size
- Dependency chain is explicit via issue links

## Encoding dependencies in Jira

```bash
# PROJ-14 is blocked by PROJ-13:
jira issue link PROJ-13 PROJ-14 "blocks"

# To see blocking relationships:
jira issue view PROJ-14   # Will show linked issues
```

Note: Jira's `issue link "blocks"` direction is `<blocker> blocks <blocked>`.

## What not to do

- Don't create child issues that are just "do X, then test X" — testing is part of the issue, not a separate child (unless it's substantial integration testing)
- Don't mirror the parent's acceptance criteria one-for-one — group related criteria into coherent deliverables
- Don't leave the parent estimate unchanged — update it to reflect the full scope

## Related Skills

- **estimate** — for sizing the child issues once created
- **jira-cli** — for `jira issue create --parent`, `jira issue link`
- **product-planning** — if splitting reveals the issue needs rethinking, not just decomposing
