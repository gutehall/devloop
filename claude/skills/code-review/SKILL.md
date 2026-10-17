---
name: code-review
description: Structured PR review: verify behavior, find logic errors and broken contracts, fix and re-test. Only /review, ship-gate G3, or when a command names this skill. Do not trigger proactively.
---

# Code Review

**Goal**: Determine with confidence whether code does what it's supposed to — not just whether it looks clean, but whether it works correctly in all meaningful cases.

Four-phase protocol: **Understand → Audit → Plan & Fix → Verify**.

---

## Phase 1: Understand Intent

Before looking for problems, establish what "correct" means for this code.

**Gather:**
- What is this code supposed to do? (Read docstrings, comments, function names, call sites)
- What are the inputs and their valid ranges/types?
- What are expected outputs or side effects?
- What invariants must hold? (e.g. "this list is always sorted", "this value is never null")
- Broader context? (What calls this? What does it feed into?)

If intent unclear from code itself, ask user one targeted question before proceeding. Don't guess at intent — a "correct" review of the wrong spec is worthless.

---

## Phase 2: Audit — Does It Actually Work?

Go through code systematically. For each function, module, or logical unit, check:

### Correctness
- [ ] Does logic match stated intent?
- [ ] Off-by-one errors? (loop bounds, slice indices, pagination)
- [ ] Comparisons correct? (`>` vs `>=`, `===` vs `==`, wrong type comparisons)
- [ ] Mutable state modified correctly? (mutation when copy intended, or vice versa)
- [ ] Async operations awaited where needed? Unhandled promise rejections?
- [ ] Return values used correctly by callers?

### Edge Cases
- [ ] Empty inputs (empty array, empty string, zero, null/undefined)
- [ ] Single-element inputs
- [ ] Maximum/minimum values
- [ ] Concurrent access (if relevant)
- [ ] First/last items in collections

### Error Handling
- [ ] Errors caught at the right level?
- [ ] Do errors propagate so callers can act on them?
- [ ] Silent failures? (catching and swallowing exceptions, returning null instead of throwing)
- [ ] Cleanup done correctly on error paths?

### Data Flow
- [ ] Is data transformed correctly at each step?
- [ ] Implicit type coercions that could go wrong?
- [ ] Nullable/optional values checked before use?
- [ ] Correct data passed between functions (not stale, not mutated unexpectedly)?

### Dependencies and Contracts
- [ ] Does this code rely on behavior from external functions/APIs that may not hold?
- [ ] Implicit assumptions about the environment (env vars, file paths, network state)?
- [ ] Do interfaces between components match? (function signatures, API shapes, event payloads)

### Tests (if present)
- [ ] Do existing tests actually cover main logic paths?
- [ ] Tests for failure cases and edge cases?
- [ ] Do tests assert the right things, or just that nothing throws?
- [ ] Could a test pass even if the code is wrong?

---

## Phase 3: Report and Fix Plan

### If no issues found

State clearly: "This code appears correct. Here's what I verified: [brief summary]." Note anything worth flagging even if not a bug (fragile patterns, missing tests for edge cases).

### If issues found

One line per issue, format: `file:line — problem. fix.` Include severity and reproduction inline when it isn't obvious from "problem":

```
Issue #N: [Short title]
Severity: Critical | High | Medium | Low
Location: [file:line or function name]
Problem: [what's wrong — specific and mechanical, not vague]
Why it matters: [what goes wrong at runtime]
Reproduction: [minimal input/state demonstrating the bug]
Fix: [change that resolves the root cause]
```

Severity guide:
- **Critical** — data loss, security vulnerability, crash in normal use, or silently wrong results
- **High** — incorrect behavior in a common case or edge case likely hit in production
- **Medium** — incorrect in uncommon cases, or fragile to minor surrounding changes
- **Low** — not buggy now but fragile, misleading, or will cause bugs later

After listing all issues, produce a **Fix Plan** ordered by priority:
1. Fix immediately (Critical/High)
2. Fix before merge (Medium)
3. Note/defer (Low)

Ask: "Should I proceed with the fixes?" — unless user already said fix everything, then proceed directly.

---

## Phase 4: Apply Fixes

Fix one issue at a time. For each:

1. **State what's changing and why** (one sentence)
2. **Make the minimum change** resolving root cause
3. **Don't refactor unrelated things** — stay focused
4. **After each fix**, confirm briefly: "Fixed: [what changed]"

When all fixes applied, move to verification.

---

## Phase 5: Verify

After fixing, validate fixes work and nothing broke.

### Run existing tests first
```bash
# Run whatever test command is appropriate for this project:
# npm test / pytest / go test ./... / cargo test / etc.
```

Tests pass: good. Tests fail: apply diagnostic-thinking skill before touching anything else.

### Check fix coverage
For each Critical or High issue fixed:
- [ ] Existing test that would catch a regression?
- [ ] If not, write a minimal test demonstrating the fix works

Test structure:
```
Test name: [describes the scenario, not the implementation]
Setup: [minimal state to reproduce the original bug]
Action: [call the code]
Assert: [the correct output or behavior]
```

### Final run
Run full test suite once more after adding new tests. Report:
- Tests passing: N
- Tests added: N
- Issues resolved: N

---

## Anti-Patterns to Avoid

| What it looks like | Why it's a problem |
|---|---|
| Reviewing style instead of correctness | Doesn't tell you if code works |
| "This looks fine to me" without checking edge cases | Most common bugs live in edge cases |
| Fixing symptoms instead of root causes | Bug comes back in a different form |
| Multiple fixes in one change | You won't know which fix resolved which issue |
| Skipping verification | Fix may work for the wrong reason |
| Tests that only cover the happy path | False confidence |
| Not understanding intent before auditing | Can't tell if something's wrong without knowing "right" |

---

## `/review` Command

Invoked by `/review` command (`claude/commands/review.md`), installed same way as rest of package — see "Testing changes" in repo's CLAUDE.md.

**Usage:**
```
/review                        ← reviews whatever you're currently working on
/review src/auth/token.ts      ← reviews a specific file
/review the payment flow       ← reviews a feature area
/review before I open the PR   ← full review + fix + verify cycle
```

---

## Quick Reference Checklist

Before signing off on any review:

- [ ] Understand what this code is supposed to do?
- [ ] Checked edge cases: empty, null, single-item, boundary values?
- [ ] Checked error paths, not just happy path?
- [ ] Verified data flows correctly between components?
- [ ] Checked async operations handled correctly?
- [ ] Do existing tests prove correctness, or just that nothing explodes?
- [ ] If fixed something, verified with a test run?
- [ ] Every Critical/High issue fixed or explicitly deferred with a reason?
