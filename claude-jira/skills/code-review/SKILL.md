---
name: code-review
description: Structured PR review: verify behavior, find logic errors and broken contracts, fix and re-test. Only /review, ship-gate G3, or when a command names this skill. Do not trigger proactively.
---

# Code Review

How to read a PR and give useful, actionable feedback.

## Mindset

Look for problems that matter — bugs, missing tests, security issues, scope creep. Not style preferences or things that don't affect correctness. Be direct about problems; brief about things that are fine.

A review saying "looks good" with no substance isn't useful. A review nitpicking formatting is noise. Aim for: everything load-bearing gets checked, everything cosmetic gets skipped.

## What to check

### 1. Correctness
- Does code do what PR description says?
- Edge cases the implementation misses?
- Errors handled, or silently swallowed?
- Assumptions baked in that could break under real conditions?

### 2. Tests
- Tests exist for new behavior?
- Do tests cover interesting paths (not just happy path)?
- If tests missing: hard to test (design smell), or just skipped?

### 3. Security
- Unvalidated user input reaching database, shell, or file system?
- Secrets or credentials hard-coded or logged?
- Auth/permission checks present where needed?
- SQL injection, XSS, or path traversal possibilities?

### 4. Scope
- PR does more than issue asks? Flag scope creep — not a blocker, but worth noting.
- PR does *less* than issue asks? Check acceptance criteria.
- New dependencies introduced? Flag if heavy or unusual.

### 5. Maintainability (light touch)
- Anything so complex it needs a comment to be understood later?
- Anything duplicated that should probably be a shared function?
- Suggestions, not blockers, unless code is genuinely unreadable.

## What to skip

- Formatting, whitespace, naming style (unless project has strong convention this clearly breaks)
- Refactoring opportunities unrelated to the change
- "I would have done this differently" without a concrete reason it matters

## Output Format

One line per finding:

`file:line: [severity] problem — fix.`

Group by category (Bugs / Tests / Security / Scope / Maintainability) only if the finding count warrants it; otherwise one flat list. No praise lines, no restating what's fine.

## Approve vs. request changes

**Approve** when: no bugs, tests cover meaningful paths, no security issues. Minor suggestions fine alongside approval.

**Request changes** when: correctness bug, missing test for risky path, or security issue. Don't block on style.

**Comment only** when: observations but no strong opinion either way. Useful for large PRs where you reviewed part of the diff.

## Related Skills

- **github-cli** — for `gh pr review` commands to submit the review
- **diagnostic** — if you need to reason through a bug found in the diff
