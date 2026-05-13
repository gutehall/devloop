# /bugs - Scan the codebase for bugs and create Jira issues

## Usage

```
/bugs              # Full codebase scan
/bugs <path>       # Scan a specific directory or file
```

## Flow

### 1. Setup: Read Jira state

- `jira project list --plain` → confirm the active project key
- Note the project key (e.g. `PROJ`) for issue creation

### 2. Scan: Systematic codebase audit

Explore the full directory tree first, then read every non-trivial source file. Don't skip files because they look clean — bugs hide in boring code.

Scan for these categories in every file:

**Security** → Highest or High
- Hardcoded secrets, API keys, passwords in source
- SQL/command injection via string concatenation or template literals
- Missing input validation at API or CLI boundaries
- Insecure deserialization or eval of external input
- Internal error details exposed to end users

**Error handling** → High or Medium
- Unhandled promise rejections (missing `.catch` or `await` without try/catch)
- Empty catch blocks that swallow exceptions silently
- Missing error checks after I/O operations
- Errors that are logged but not propagated when they should be

**Logic errors** → High or Medium
- Off-by-one errors in loops, slices, or index calculations
- Wrong comparison operators or inverted conditions
- Incorrect boolean logic (check De Morgan's law violations)
- State mutations that affect other code paths unexpectedly
- Missing edge case handling (empty arrays, zero, negative numbers, null inputs)

**Null / undefined access** → Medium
- Property access without null checks where the value may be absent
- Missing optional chaining in chains that can be undefined
- Variables used before assignment

**Type safety** → Medium
- Implicit type coercion producing wrong results
- `parseInt` / `parseFloat` without radix
- Comparisons between values of incompatible types

**Resource management** → Medium or Low
- Database connections or file handles not closed in finally blocks
- Timers or intervals created but never cleared
- Streams not destroyed on error

**Concurrency** → High or Medium
- Shared mutable state accessed from concurrent async paths
- Promises not awaited where ordering matters
- Race conditions in async event handlers

Scan everything. Depth matters more than speed here.

### 3. Prioritize all findings

After the full scan, rank every finding before creating any issues:

| Priority | Criteria |
|----------|----------|
| Highest  | Security vulnerabilities, authentication bypass, data corruption, data loss |
| High     | Crashes, incorrect behavior in critical paths, unhandled exceptions that surface to users |
| Medium   | Logic errors in non-critical paths, missing null checks, incorrect error handling |
| Low      | Edge cases, fragile patterns, minor incorrect behavior unlikely to be hit |

### 4. Create Jira issues — highest priority first

Create all Highest issues, then High, then Medium, then Low.

For each bug:

**Summary**: `[Category] Short description`
Example: `[Security] SQL injection in user search endpoint`

**Description**:
```
## Bug
<what the bug is and why it is wrong>

## Location
`path/to/file.ext:line`

## Impact
<what can go wrong if this is hit>

## Suggested fix
<concrete fix — be specific, not vague>
```

```bash
jira issue create -tBug \
  -s"[Category] Short description" \
  -b"## Bug\n...\n\n## Location\n...\n\n## Impact\n...\n\n## Suggested fix\n..." \
  --priority High \
  --label "bug"
```

### 5. Report

When all issues are created, print a summary:

```
Found N bugs — X highest, Y high, Z medium, W low.
Created N Jira issues.
```

List each created issue key and summary.

## Error handling

- If `jira project list` fails: "Could not reach Jira. Is `jira` configured? Run `jira init`." and stop.
- If issue creation fails: report the error, do not silently skip. Offer to retry.
- If the codebase is empty or has no source files: say so and stop.

## Rules

- Read every source file — do not sample or skip
- Never create an issue for a pattern that is not actually wrong in this codebase (no hypothetical bugs)
- One issue per bug, not one issue per file or category
- Keep descriptions concrete: file path, line reference, exact problem, specific fix
