---
name: bugs
description: Scan the codebase for bugs and create prioritized Jira issues. Only /bugs, or when a command names this skill. Do not trigger proactively.
---

# Bug Scanner

Exhaustively scan codebase for bugs, create one Jira issue per finding, ordered by severity.

## Setup

Before scanning:

1. `jira project list --plain` — confirm active project key
2. Note project key (e.g. `PROJ`) for issue creation

Skip this if project key already known from earlier in the session.

## Scanning

### `--changed` mode

A full scan re-reads the entire codebase every run. `--changed` narrows the scan to what moved since last time:

1. Find the latest local tag matching `bugs-last-scan-*` (`git tag -l 'bugs-last-scan-*'`).
2. `git diff --name-only <that-tag>..HEAD` (if no tag, treat as a full scan).
3. After filing issues, tag the current commit so the next `--changed` run has a fresh marker: `git tag "bugs-last-scan-$(date +%Y-%m-%d)"`. Don't push the tag unless the user asks — it's a local bookkeeping marker.

If `--changed` yields zero files, say so and stop — nothing to scan.

### Full scan

Explore full directory tree first (LS, Glob) — or, in `--changed` mode, just the file set above. For a large tree, spawn several cheap locator sweeps in parallel (split by directory or category; prefer `cavecrew-investigator` if that type exists, else Task/Explore on Haiku). Then read every non-trivial source file yourself, including the sites any subagent cites. Don't skip files that look clean — bugs hide in boring code.

### What to look for

**Security** → Highest or High
- Hardcoded secrets, API keys, passwords in source
- SQL or command injection via string concatenation or template literals
- Missing input validation at API or CLI boundaries
- Insecure deserialization or eval of external input
- Internal error details exposed to end users

**Error handling** → High or Medium
- Unhandled promise rejections (missing `.catch` or `await` without try/catch)
- Empty catch blocks that swallow exceptions silently
- Missing error checks after I/O operations
- Errors logged but not propagated when they should be

**Logic errors** → High or Medium
- Off-by-one errors in loops, slices, or index calculations
- Wrong comparison operators or inverted conditions
- Incorrect boolean logic (check De Morgan's law violations)
- State mutations affecting other code paths unexpectedly
- Missing edge cases (empty arrays, zero, negative numbers, null inputs)

**Null / undefined access** → Medium
- Property access without null checks where value may be absent
- Missing optional chaining in chains that can be undefined
- Variables used before assignment

**Type safety** → Medium
- Implicit type coercion producing wrong results
- `parseInt` / `parseFloat` without radix
- Comparisons between values of incompatible types

**Resource management** → Medium or Low
- Connections or file handles not closed in finally blocks
- Timers or intervals created but never cleared
- Streams not destroyed on error

**Concurrency** → High or Medium
- Shared mutable state accessed from concurrent async paths
- Promises not awaited where ordering matters
- Race conditions in async event handlers

## Priority mapping

| Priority | Criteria |
|----------|----------|
| Highest | Security vulnerabilities, authentication bypass, data corruption, data loss |
| High    | Crashes, incorrect behavior in critical paths, unhandled exceptions surfacing to users |
| Medium  | Logic errors in non-critical paths, missing null checks, incorrect error handling |
| Low     | Edge cases, fragile patterns, minor incorrect behavior unlikely to be hit |

## Creating Jira issues

Rank findings first, create in order: Highest → High → Medium → Low.
Priority order keeps backlog correctly sorted. Ask if issues go under existing epic or unassigned.

**Summary format:** `[Category] Short description`
Example: `[Security] SQL injection in user search endpoint`

**Description template:**
```
## Bug
<what the bug is and why it is wrong>

## Location
`path/to/file.ext:line`

## Impact
<what can go wrong if this is hit>

## Suggested fix
<concrete fix — be specific>
```

```bash
jira issue create -tBug \
  -s"[Category] Short description" \
  -b"## Bug\n...\n\n## Location\n...\n\n## Impact\n...\n\n## Suggested fix\n..." \
  --priority High \
  --label "bug"
```

## Output Format

One line per finding when listing to the user:

`file:line: [severity] problem — fix.`

Then the summary block once all issues are created:

```
Found N bugs — X highest, Y high, Z medium, W low.
Created N Jira issues.
```

List each created issue key and summary.

## Rules

- Read every source file — no sampling or skipping
- One issue per bug, not one per file or category
- Never create issue for pattern not actually wrong in this codebase
- Keep descriptions concrete: file path, line reference, exact problem, specific fix

## Related skills

- **jira-cli** — full CLI reference for issue management
- **product-planning** — for planning follow-up work after bug audit
