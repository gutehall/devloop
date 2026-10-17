---
name: debt
description: Scan the codebase for tech debt and create prioritized Jira issues tagged tech-debt. Only /debt, or when a command names this skill. Do not trigger proactively.
---

# Tech Debt Scanner

Exhaustively scan codebase for tech debt, create one Jira issue per finding, ordered by severity.

## Setup

Before scanning:

1. `jira project list --plain` — confirm active project key
2. Note project key (e.g. `PROJ`) for issue creation

Skip this if project key already known from earlier in the session.

## Scanning

### `--changed` mode

A full scan re-reads the entire codebase every run. `--changed` narrows the scan to what moved since last time:

1. Find the latest local tag matching `debt-last-scan-*` (`git tag -l 'debt-last-scan-*'`).
2. `git diff --name-only <that-tag>..HEAD` (if no tag, treat as a full scan).
3. After filing issues, tag the current commit so the next `--changed` run has a fresh marker: `git tag "debt-last-scan-$(date +%Y-%m-%d)"`. Don't push the tag unless the user asks — it's a local bookkeeping marker.

If `--changed` yields zero files, say so and stop — nothing to scan.

### Full scan

Explore full directory tree first (LS, Glob) — or, in `--changed` mode, just the file set above. For a large tree, spawn several cheap locator sweeps in parallel (split by directory or category; prefer `cavecrew-investigator` if that type exists, else Task/Explore on Haiku). Then read every non-trivial source file yourself, including the sites any subagent cites. Don't skip files that look tidy — debt hides in overlooked corners.

### What to look for

**Missing tests** → Medium or High
- Source files with no corresponding test file
- Functions or classes with no visible test coverage
- Critical code paths (auth, payments, data mutations) with zero test coverage → High

**Complexity** → Medium
- Functions over ~50 lines
- Deeply nested conditionals (3+ levels)
- Functions with 5 or more parameters

**Dead code** → Low or Medium
- Unused exports or functions never imported elsewhere
- Unreachable code blocks
- Large sections of commented-out code

**TODO/FIXME/HACK/XXX comments** → Low or Medium
- Surface all of them; judge priority by comment content
- Use Medium if comment indicates real correctness or reliability problem

**Hardcoded configuration** → Medium
- URLs, ports, timeouts, or limits hardcoded in logic (not in config files)
- Values that belong in env vars or named constants

**Duplicated code** → Medium
- Similar functions or logic blocks that could be shared
- Copy-paste patterns across files

**Missing TypeScript / type safety** → Low
- `any` casts in TypeScript codebases
- Missing return types on exported functions
- Untyped parameters

**Outdated patterns** → Low or Medium
- Deprecated API usage within codebase's own framework or libraries
- Old async patterns (callbacks where promises are project standard)
- Inconsistent patterns across files (some use one approach, others different)

## Priority mapping

| Priority | Criteria |
|----------|----------|
| Highest | Hardcoded secret-adjacent config (credentials, tokens, keys) |
| High    | Critical paths with zero test coverage, severe complexity making bugs likely |
| Medium  | Most debt: complexity, duplication, hardcoded config, missing types |
| Low     | TODOs, dead code, style inconsistencies |

## Creating Jira issues

Rank findings first, create in order: Highest → High → Medium → Low.
Priority order keeps backlog correctly sorted. Ask if issues go under existing epic or unassigned.

**Summary format:** `[DebtType] Short description`
Example: `[Missing Tests] No test coverage for auth token validation`

**Description template:**
```
## Issue
<what the debt is>

## Location
`path/to/file.ext:line` (or function/class name)

## Impact
<why this matters — what risks does it create>

## Suggested fix
<specific action to take>
```

```bash
jira issue create -tTask \
  -s"[DebtType] Short description" \
  -b"## Issue\n...\n\n## Location\n...\n\n## Impact\n...\n\n## Suggested fix\n..." \
  --priority Medium \
  --label "tech-debt"
```

## Output Format

One line per finding when listing to the user:

`file:line: [severity] problem — fix.`

Then the summary block once all issues are created:

```
Found N debt items — X highest, Y high, Z medium, W low.
Created N Jira issues.
```

List each created issue key and summary.

## Rules

- Read every source file — no sampling or skipping
- One issue per debt item, not one per file or category
- Never create issue for pattern not actually present in this codebase
- Keep descriptions concrete: file path, line reference, exact problem, specific fix

## Related skills

- **bugs** — for finding runtime bugs rather than structural debt
- **jira-cli** — full CLI reference for issue management
- **product-planning** — for planning refactor work after debt audit
