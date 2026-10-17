---
name: ship-gate
description: Shared pre-push quality gate G1–G3 (diff review, local verify, self-review). Only /done, /pr, /grind, /autopilot, or when a command names this skill. Do not trigger proactively.
---

# Ship gate

Mandatory quality gate before any push. Callers pass **mode**:

- **interactive** (`/done`, `/pr`) — stop and report to the user on failure
- **autonomous** (`/grind`, `/autopilot`) — unfixable Critical/High or unfixable test/build failure → **STOP LOOP** (leave the issue In Progress; print `<cmd>: <id> failed <step> — <finding>. Stopping loop.`)

Do not push until G1–G3 pass. Results feed the PR body's test plan and Known issues.

## Project conventions

Canonical statement of the convention-resolution rule. `/plan`, `/done`, and `/pr` point here instead of restating it (`/next` and `/grind` load conventions lazily at their implement step). Apply it before touching code (in this skill, before G2):

1. If this repo has a skill whose description says it defines coding rules, verify/test commands, or pre-merge gates — invoke it (its checks run inside G2).
2. Else if `CLAUDE.md`, `AGENTS.md`, or `CONTRIBUTING.md` exists at the repo root (then the nearest subdirectory you are editing) — follow those.
3. Else detect test/build from tooling and continue. Ignore this block when none of the above exist.

## Base branch

Detect once and reuse for every `git log` / `git diff` in this run — do not re-derive later:

```bash
git remote show origin | grep 'HEAD branch'
```

Default to `main` if origin has no HEAD branch. Never assume `develop`.

Reuse `git log --oneline <base>..HEAD` and `git diff --stat <base>..HEAD` later in the same run (PR body, ship report) instead of re-running them.

## G1. Review the diff, then commit

Read `git status` and the **full** `git diff` (plus `git diff <base>..HEAD` for already-committed work) before staging anything:

- Stage only changes that belong to this issue/project
- Leave out debug prints, commented-out code, stray files, and unrelated edits — list anything excluded
- Never blind-stage with `git add -A` without reading what it picks up

Then commit what survived. Prefer `ISSUE-12: …` per change; a project branch may use `<project-slug>: <summary>`. Interactive `/pr` may ask for the commit message.

## G2. Verify locally

Detect this repo's test and build commands (`package.json` scripts, `Makefile`, `pyproject.toml`, CI workflow files) and run them. **Never hardcode** `npm test` or any product-specific verify matrix — run what this repo actually defines. If a conventions skill or `CLAUDE.md` named extra gates, run those too.

- Tests fail or build breaks → interactive: **stop. Do not push.** Fix it or report to the user. Autonomous: **STOP LOOP** if unfixable.
- `/pr --draft` / user explicitly wants a WIP PR: push with `--draft` and state the failures in the body (interactive only).
- No test/build tooling exists → say so explicitly; it must also be stated in the PR body. Do not fail closed on a missing test command.

Record the exact commands and their results — they go into the PR body's test plan.

## G3. Self-review the code

Spawn a cheap reviewer on the full diff (`git diff <base>..HEAD`).

**Subagents (leads, not proof):** Prefer `cavecrew-reviewer` if that agent type exists; otherwise Task/Explore on Haiku or the cheapest offered model. Demand a compressed `path:line: problem. fix.` receipt — treat every line as a lead, not a confirmed finding. Read the cited span yourself and confirm it before fixing or noting it. Discard a finding that doesn't hold — do not re-spawn to "prove" it. Only if no subagent tool exists at all: review the diff on the main thread. Never spawn a full-size Sonnet agent just to locate files.

**Size gate:** check `git diff --stat <base>..HEAD`. Under 400 lines **and** under 8 files: the verified leads *are* the review — do not load the **code-review** skill into the main thread. At or above either threshold (L/XL): also load the **code-review** skill and run its full Understand → Audit phases against the diff, using the verified leads as a starting point, not a replacement.

For each **verified** finding:

- **Critical/High** → fix now, re-run G2, then continue. Autonomous: unfixable → **STOP LOOP**, leave issue In Progress.
- **Medium** → fix now, or list under "Known issues" in the PR body — never silently drop
- **Low** → note in the PR body or propose a follow-up issue
