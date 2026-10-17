---
name: debt
description: Scan the codebase for tech debt and create prioritized Linear issues tagged tech-debt. Only /debt, or when a command names this skill. Do not trigger proactively.
---

# Tech Debt Scanner

Exhaustively scan codebase, create one Linear issue per finding, ordered by severity.

## Setup

Before scanning:

1. `mcp__claude_ai_Linear__list_teams` — identify active team (skip if already known from this session or `.linear` config)
2. `mcp__claude_ai_Linear__list_issue_labels` — note available labels (skip if already known)
3. If no "tech-debt" label exists: `mcp__claude_ai_Linear__create_issue_label` to create one

## Scanning

### `--changed` mode

A full scan re-reads the entire codebase every run. `--changed` narrows the scan to what moved since last time:

1. Find the latest local tag matching `debt-last-scan-*` (`git tag -l 'debt-last-scan-*'`).
2. `git diff --name-only <that-tag>..HEAD` (if no tag, treat as a full scan).
3. After filing issues, tag the current commit so the next `--changed` run has a fresh marker: `git tag "debt-last-scan-$(date +%Y-%m-%d)"`. Don't push the tag unless the user asks — it's a local bookkeeping marker.

If `--changed` yields zero files, say so and stop — nothing to scan.

### Full scan

Explore full directory tree first (LS, Glob) — or, in `--changed` mode, just the file set above. For a large tree, spawn several cheap locator sweeps in parallel (split by directory or category; prefer `cavecrew-investigator` if that type exists, else Task/Explore on Haiku). Then read every non-trivial source file yourself, including the sites any subagent cites. Don't skip tidy-looking files — debt hides in overlooked corners.

### What to look for

**Missing tests** → Medium or High
- Source files with no corresponding test file
- Functions or classes with no visible test coverage
- Critical code paths (auth, payments, data mutations) with zero test coverage → High

**Complexity** → Medium
- Functions over ~50 lines
- Deeply nested conditionals (3+ levels)
- Functions with 5+ parameters

**Dead code** → Low or Medium
- Unused exports or functions never imported elsewhere
- Unreachable code blocks
- Large sections of commented-out code

**TODO/FIXME/HACK/XXX comments** → Low or Medium
- Surface all of them; judge priority by comment content
- Medium if comment indicates a real correctness or reliability problem

**Hardcoded configuration** → Medium
- URLs, ports, timeouts, limits hardcoded in logic (not config files)
- Values that belong in env vars or named constants

**Duplicated code** → Medium
- Similar functions or logic blocks that could be shared
- Copy-paste patterns across files

**Missing TypeScript / type safety** → Low
- `any` casts in TypeScript codebases
- Missing return types on exported functions
- Untyped parameters

**Outdated patterns** → Low or Medium
- Deprecated API usage within the codebase's own framework/libraries
- Old async patterns (callbacks where promises are project standard)
- Inconsistent patterns across files

## Priority mapping

| Priority | Criteria |
|----------|----------|
| 1 Urgent | Hardcoded secret-adjacent config (credentials, tokens, keys) |
| 2 High   | Critical paths with zero test coverage, severe complexity making bugs likely |
| 3 Medium | Most debt: complexity, duplication, hardcoded config, missing types |
| 4 Low    | TODOs, dead code, style inconsistencies |

## Creating Linear issues

Rank findings first, create in order: Urgent → High → Medium → Low. Ask if it should go in existing project or new one.
Priority order keeps backlog correctly sorted.

**Title format:** `[DebtType] Short description`
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

**Labels:** `["tech-debt"]` plus matching existing type labels (e.g. "testing", "refactor"). Don't create extra category labels.

Use `mcp__claude_ai_Linear__save_issue` per issue.

## Output Format

When done:

```
Found N debt items — X urgent, Y high, Z medium, W low.
Created N Linear issues.
```

Then one line per issue: `ISSUE-ID: Title`

## Rules

- Read every source file — no sampling or skipping
- One issue per debt item, not per file or category
- Never create an issue for a pattern not actually present in this codebase
- Keep descriptions concrete: file path, line reference, exact problem, specific fix

## Related skills

- **bugs** — for finding runtime bugs rather than structural debt
- **linear-cli** — full CLI reference for label and issue management
- **product-planning** — for planning refactor work after the debt audit
