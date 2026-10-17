---
name: split
description: Decompose a large Linear issue into smaller, independently trackable child issues. Only /split, or when a command names this skill. Do not trigger proactively.
---

# Issue Splitting

How to break down an L or XL issue into sub-issues that can be worked on and tracked independently.

## Mindset

A well-split issue becomes invisible after splitting — the sub-issues tell the full story, the parent just holds them together. A bad split creates artificial boundaries that don't map to how the work actually flows.

Split along natural seams, not arbitrary size targets. Goal: sub-issues you could hand to someone with no further explanation.

## When to split

Split when:
- Issue is L or XL and approach isn't fully clear
- Description contains multiple distinct deliverables ("and also", "as well as")
- Acceptance criteria spans different systems or roles
- Issue has sat unstarted — probably too big to pick up

Don't split when:
- Issue is M but just feels vague — clarify the description instead
- Sub-tasks are too small to be worth tracking (< XS each)
- Work is highly sequential with no parallelism possible and a single person will do it all — a checklist in the issue description is simpler

## How to find the seams

Work through the issue description and acceptance criteria, then ask:

1. **By layer:** UI / API / data model / infra — natural split for full-stack features
2. **By phase:** research → design → implement → test — split when phases have different skill requirements or could be blocked
3. **By component:** feature A and feature B mentioned in the same issue — split them
4. **By risk:** risky, unknown part vs. mechanical, known part — split so risky part can be explored first
5. **By dependency:** what must be done before anything else? That's a natural first sub-issue.

## Structuring the breakdown

Each sub-issue needs:
- A clear title that stands alone (not "Part 1" or "Step A" — say what it does)
- An estimate (XS–M; if L, split further)
- Dependencies encoded with `--blocked-by` (don't rely on numbering — the blocker flag keeps `--unblocked` accurate)

Parent issue keeps its original description and acceptance criteria. Add a comment naming the sub-issues so anyone reading the parent knows where the work lives.

## Example decomposition

**Before:** `FIN-12: Implement checkout flow — XL`

**After:**
```
FIN-13: Design checkout data model — S
FIN-14: Implement cart API endpoints — M  (blocked by FIN-13)
FIN-15: Build checkout UI — M            (blocked by FIN-14)
FIN-16: Add payment integration — M      (blocked by FIN-14)
FIN-17: Write E2E checkout tests — S     (blocked by FIN-15, FIN-16)
```

Notice:
- FIN-15 and FIN-16 can run in parallel once FIN-14 is done
- Each has a clear deliverable and a reasonable size
- The dependency chain is explicit — `--unblocked` surfaces the right issue at each step

## What not to do

- Don't create sub-issues that are just "do X, then test X" — testing is part of the issue, not a separate sub-issue (unless substantial integration testing)
- Don't mirror the parent's acceptance criteria one-for-one as sub-issues — group related criteria into coherent deliverables
- Don't leave the parent estimate unchanged — update it to XL to signal the scope

## Related Skills

- **estimate** — for sizing the sub-issues once created
- **linear-cli** — for `linear issue create --parent`, `--blocked-by`
- **product-planning** — if splitting reveals the issue needs rethinking, not just decomposing
