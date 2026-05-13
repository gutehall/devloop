---
name: debt
description: Scan the entire codebase for tech debt and create prioritized Jira issues tagged "tech-debt". Use this skill whenever the user runs /debt, asks to find tech debt, wants a code quality audit, or wants debt tracked in Jira.
---

# Tech Debt Scanner

Exhaustively scan the codebase for tech debt and create one Jira issue per finding, ordered by severity.

## Setup

Before scanning:

1. `jira project list --plain` — confirm the active project key
2. Note the project key (e.g. `PROJ`) for issue creation

## Scanning

Explore the full directory tree first (LS, Glob), then read every non-trivial source file.
Don't skip files that look tidy — debt hides in overlooked corners.

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
- Surface all of them; judge priority by the comment content
- Use Medium if the comment indicates a real correctness or reliability problem

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
- Deprecated API usage within the codebase's own framework or libraries
- Old async patterns (callbacks where promises are the project standard)
- Inconsistent patterns across files (some use one approach, others a different one)

## Priority mapping

| Priority | Criteria |
|----------|----------|
| Highest | Hardcoded secret-adjacent config (credentials, tokens, keys) |
| High    | Critical paths with zero test coverage, severe complexity making bugs likely |
| Medium  | Most debt: complexity, duplication, hardcoded config, missing types |
| Low     | TODOs, dead code, style inconsistencies |

## Creating Jira issues

Rank all findings first, then create issues in order: Highest → High → Medium → Low.
Creating in priority order ensures the backlog is correctly sorted.

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

## Output

When all issues are created:

```
Found N debt items — X highest, Y high, Z medium, W low.
Created N Jira issues.
```

List each created issue key and summary.

## Rules

- Read every source file — do not sample or skip
- One issue per debt item, not one per file or category
- Never create an issue for a pattern that is not actually present in this codebase
- Keep descriptions concrete: file path, line reference, exact problem, specific fix

## Related skills

- **bugs** — for finding runtime bugs rather than structural debt
- **jira-cli** — full CLI reference for issue management
- **product-planning** — for planning refactor work after the debt audit
