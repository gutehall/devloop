---
name: estimate
description: How to estimate issue complexity using t-shirt sizes — what signals map to what sizes, and when to flag for splitting. Used by /estimate.
---

# Estimation

How to read a Jira issue and reason about how long it will take.

## Mindset

The goal of an estimate is to flag surprises, not predict the future. An XS/S/M issue should be completable in a day or less by one person. Anything L or XL is a signal to break it down, not just a bigger number.

Default to smaller. Scope almost always expands during implementation. Estimating M when you're unsure between S and M is usually right.

## Sizes

| Size | Story Points | Meaning | Rule of thumb |
|------|-------------|---------|---------------|
| XS | 1 | Trivial | Config change, one-liner fix, copy update |
| S | 2 | Small | A couple hours — one focused area, obvious approach |
| M | 3 | Medium | About a day — a few files, some exploration needed |
| L | 5 | Large | Multi-day — consider breaking down |
| XL | 8 | Very large | Definitely break down before starting |

## Reading an issue

Work through these in order:

**1. What's the scope?**
How many files, modules, or systems are touched? A change contained to one module is smaller than one that crosses system boundaries.

**2. What's the unknowns?**
Is the approach obvious, or does it require research/exploration? Unknowns add size. If you'd need to spend time understanding before coding, that's at least an M.

**3. What's the risk?**
Does it touch auth, billing, data migrations, public APIs? Risky areas warrant more careful implementation and testing — add a size.

**4. What tests are needed?**
Unit tests: small addition. Integration tests or test setup: meaningful addition. No tests needed: subtract a size.

**5. Is there a UI component?**
UI work is usually harder to estimate because of iteration. Add a size if there's visual design or significant UX work.

## Signals by size

**XS:** Single file, single function, no logic change (config, copy, rename). Obvious correct answer.

**S:** One area of the codebase, clear approach, basic tests. Could explain the full implementation before writing any code.

**M:** 2–4 files across a module, some design decisions to make, tests needed. Might discover one unexpected thing during implementation.

**L:** Multiple modules or systems involved, some exploration required, or significant test coverage needed. Risk of scope expansion mid-implementation.

**XL:** Unclear scope, many unknowns, touches critical systems, or is really multiple features bundled together. Should be broken into child issues before starting.

## When to flag for splitting

Flag L and XL issues for `/split`. Signs an issue should be split:
- The description contains "and also" or "as well as" — two features bundled
- The acceptance criteria has 5+ items spanning different areas
- You can't describe the full implementation without saying "it depends"
- The issue has been sitting unstarted for a long time (probably too big to start)

## When to skip

Don't estimate:
- Issues with no description (can't reason about scope)
- Issues that are blocked (scope may change once unblocked)
- Issues that are already In Progress (the estimate no longer matters for planning)

Note skipped issues at the end of the session.

## Related Skills

- **jira-cli** — for applying story point estimates via `jira issue edit`
- **split** — for decomposing L/XL issues into smaller child issues
