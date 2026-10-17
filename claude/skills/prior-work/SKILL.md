---
name: prior-work
description: Check whether a Linear issue is already done (merged PR, branch, code, duplicate) before branching. Only /next, /grind, /autopilot, or when a command names this skill. Do not trigger proactively.
---

# Prior-Work Check

Run **after** reading the issue's description and acceptance criteria, **before** branching or implementing.

## Outcomes

- **Shipped** — a merged PR closes this issue, or the code already meets the acceptance criteria in full.
- **In flight** — an open PR or branch already references or implements this work.
- **Duplicate** — another issue (open, in progress, or done) describes the same work.
- **Partial** — a related solution exists that this work should extend, not rebuild.
- **Clear** — no meaningful prior work.

## Checks (run in parallel)

Use the issue's key terms (feature name, files, symbols, error text).

1. **PRs and branches**
   ```bash
   gh pr list --state all --search "<ISSUE-ID>"
   gh pr list --state all --search "<key terms>"
   git branch -a --list "*<issue-id>*"
   ```
   `mcp__claude_ai_Linear__get_issue` also returns linked PRs — a merged-PR link is the strongest "shipped" signal.
2. **Duplicate issues**
   ```bash
   linear issues --open
   ```
   Compare by title and description. Near-identical Done → likely shipped; open → duplicate or parallel work.
3. **Existing code** — 1–3 targeted greps/globs on the main thread (functions, routes, config keys, UI strings from the AC).

Locator rule: main-thread grep by default; at most one cheap locator, and only when the caller already classified the issue as feature / multi-file.

## Session cache

- `linear issues --open` / `linear issues --project "<project>"` — reuse within the same `/next project` or `/grind` loop until an issue status or comment changes.
- `gh pr list --state all --search "<key terms>"` — cache per distinct search-term set.
- Do not cache across sessions.
- If unsure whether the cache is stale, re-run.

## Judging the match

Prior work counts only if, as-is, it satisfies this issue's acceptance criteria (fully → Shipped or In flight; partway → Partial). A shared topic is not a duplicate. Uncertain between partial and clear → treat as clear, and note what you found.

## Report

Verify the cited PR, commit, or code yourself before reporting it.

```
Prior-work check — ISSUE-12 "Add CSV export to billing"
  Shipped?   PR #188 "Export invoices as CSV" merged — implements src/billing/export.ts
  Duplicate? ISSUE-29 (Done) covers the same export
  → Looks already done.
```

Clear: `Prior-work check: no existing solution found. Proceeding.`

## Interactive callers (/next)

Ask; do not decide:

```
[Enter] Proceed anyway — implement as specified
[c]     Close ISSUE-12 as already done (comment linking PR #188, set Done)
[e]     Extend the existing solution — build on PR #188's code
[d]     Mark duplicate of ISSUE-29 — collapse via the dedupe flow
[q]     Quit
```

- **Close:** `mcp__claude_ai_Linear__save_comment` linking the prior work, then `linear issue update <id> --status "Done"`.
- **Extend:** implement on top of the existing files/PR.
- **Duplicate:** hand off to the **dedupe** skill's collapse action.

## Autonomous callers (/grind, /autopilot)

Never prompt. Never auto-close.

- **Shipped / Duplicate / In flight** → do not implement:
  ```bash
  mcp__claude_ai_Linear__save_comment   # "Prior-work check: appears already done by PR #188 / duplicate of ISSUE-29. Flagged for human review."
  linear issue update <id> --status "Backlog" --label "needs-human"
  ```
- **Partial** → extend the existing code; note what was extended in the PR body.
- **Clear** → proceed.
