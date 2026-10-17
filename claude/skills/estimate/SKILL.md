---
name: estimate
description: Estimate Linear issue complexity with t-shirt sizes and flag issues to split. Only /estimate, or when a command names this skill. Do not trigger proactively.
---

# Estimation

How to read a Linear issue and reason about how long it'll take.

## Mindset

Goal of an estimate is to flag surprises, not predict the future. XS/S/M should be completable in a day or less by one person. L or XL is a signal to break down, not just a bigger number.

Default to smaller. Scope almost always expands during implementation. Estimating M when unsure between S and M is usually right.

## Sizes

| Size | Meaning | Rule of thumb |
|------|---------|---------------|
| XS | Trivial | Config change, one-liner fix, copy update |
| S | Small | A couple hours — one focused area, obvious approach |
| M | Medium | About a day — a few files, some exploration needed |
| L | Large | Multi-day — consider breaking down |
| XL | Very large | Definitely break down before starting |

## Reading an issue

Work through in order:

**1. What's the scope?**
How many files, modules, systems touched? Change contained to one module is smaller than one crossing system boundaries.

**2. What's the unknowns?**
Approach obvious, or research/exploration needed? Unknowns add size. If you'd need to spend time understanding before coding, that's at least M.

**3. What's the risk?**
Touches auth, billing, data migrations, public APIs? Risky areas warrant careful implementation and testing — add a size.

**4. What tests are needed?**
Unit tests: small addition. Integration tests or test setup: meaningful addition. No tests needed: subtract a size.

**5. Is there a UI component?**
UI work usually harder to estimate due to iteration. Add a size if visual design or significant UX work.

## Signals by size

**XS:** Single file, single function, no logic change (config, copy, rename). Obvious correct answer.

**S:** One area of codebase, clear approach, basic tests. Could explain full implementation before writing code.

**M:** 2–4 files across a module, some design decisions, tests needed. Might discover one unexpected thing during implementation.

**L:** Multiple modules/systems involved, some exploration required, or significant test coverage needed. Risk of scope expansion mid-implementation.

**XL:** Unclear scope, many unknowns, touches critical systems, or is really multiple features bundled together. Should be broken into L/M issues before starting.

## When to flag for splitting

Flag L and XL issues for `/split`. Signs an issue should split:
- Description contains "and also" or "as well as" — two features bundled
- Acceptance criteria has 5+ items spanning different areas
- Can't describe full implementation without saying "it depends"
- Issue has sat unstarted a long time (probably too big to start)

## When to skip

Don't estimate:
- Issues with no description (can't reason about scope)
- Blocked issues (scope may change once unblocked)
- Issues already in progress (estimate no longer matters for planning)

Note skipped issues at end of session.

## Related Skills

- **linear-cli** — for `linear issue update --estimate <size>`
- **split** — for decomposing L/XL issues into smaller pieces
