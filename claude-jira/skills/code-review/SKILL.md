---
name: code-review
description: How to review a pull request — what to look for, how to reason about quality, and how to communicate findings. Used by /review.
---

# Code Review

How to read a PR and give useful, actionable feedback.

## Mindset

You're looking for problems that matter — bugs, missing tests, security issues, scope creep. Not style preferences or things that don't affect correctness. Be direct about problems; be brief about things that are fine.

A review that says "looks good" with no substance is not useful. A review that nitpicks formatting is noise. Aim for: everything load-bearing gets checked, everything cosmetic gets skipped.

## What to check

### 1. Correctness
- Does the code do what the PR description says?
- Are there edge cases the implementation misses?
- Are errors handled, or do they silently swallow failures?
- Are any assumptions baked in that could break under real conditions?

### 2. Tests
- Are there tests for the new behavior?
- Do the tests actually cover the interesting paths (not just the happy path)?
- If tests are missing: is it because the code is hard to test (a design smell), or just skipped?

### 3. Security
- Any unvalidated user input reaching a database, shell, or file system?
- Secrets or credentials hard-coded or logged?
- Auth/permission checks present where needed?
- SQL injection, XSS, or path traversal possibilities?

### 4. Scope
- Does the PR do more than the issue asks? Flag scope creep — not as a blocker, but worth noting.
- Does the PR do *less* than the issue asks? Check acceptance criteria.
- Are there new dependencies introduced? Worth flagging if heavy or unusual.

### 5. Maintainability (light touch)
- Is there anything so complex it needs a comment to be understood later?
- Is anything duplicated that should probably be a shared function?
- These are suggestions, not blockers, unless the code is genuinely unreadable.

## What to skip

- Formatting, whitespace, naming style (unless the project has a strong convention and this clearly breaks it)
- Refactoring opportunities unrelated to the change
- "I would have done this differently" without a concrete reason it matters

## How to communicate

**Bugs / correctness issues** — direct, specific, with line or file:
> "auth.ts:42 — `req.user` could be undefined here if the session has expired. Will throw."

**Missing tests** — state what's untested:
> "No test for the case where `userId` is null. This path exists in the code."

**Security** — be clear about the risk:
> "Line 88: `query` is interpolated directly into the SQL string. Use a parameterized query."

**Suggestions (non-blocking)** — label them:
> "Suggestion: this could be extracted into a helper, but not a blocker."

**Scope observations** — neutral, not accusatory:
> "This touches the billing module which wasn't in the original issue. Worth noting for QA."

## Approve vs. request changes

**Approve** when: no bugs, tests cover the meaningful paths, no security issues. Minor suggestions are fine alongside an approval.

**Request changes** when: there's a correctness bug, a missing test for a risky path, or a security issue. Don't block on style.

**Comment only** when: you have observations but no strong opinion either way. Useful for large PRs where you reviewed part of the diff.

## Related Skills

- **github-cli** — for the `gh pr review` commands to submit the review
- **diagnostic** — if you need to reason through a bug you found in the diff
