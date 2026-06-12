# /grind - Autonomous one-cycle: pick → implement → ship

One full development cycle in a single command: `/next` + implement + `/done`, with **no interactive questions**. Built to be wrapped in `/loop` so a whole epic drains unattended.

```
/loop /grind project       # drain an epic, one issue per cycle, until empty
/loop /grind issue         # one issue per branch per cycle
```

Run bare (`/grind project`) to do exactly one cycle and stop.

## Why this exists

`/next` and `/done` are interactive — they ask scope, work type, base branch. Under `/loop` those questions block forever. `/grind` resolves every decision automatically from branch + config, and where it cannot, it **stops the loop** instead of hanging or guessing.

---

## Scope (never ask)

Resolve scope in this order — do **not** prompt:

1. Explicit arg: `/grind project` or `/grind issue`
2. Current branch: `<epic-slug>-YYYY-MM-DD` → project; `PROJ-123-*` → issue
3. Default epic in config → project mode

If none resolve → **STOP LOOP**, print: `grind: scope unresolved — run /grind project or /grind issue once to seed.`

**PR model:** `/grind` ships **one PR per issue every cycle**, in both scopes — even in project (epic) mode. This is intentional and differs from interactive `/done project`, which batches a whole branch into a single PR. Autonomous draining wants per-issue PRs so each gets its own CI gate and a failure isolates to one issue. "Project mode" here only changes *which queue* issues are picked from, not how they ship.

---

## Cycle

### 1. Pick the issue (follow `/next`, non-interactive)

**Project mode (epic):**
```bash
jira issue view <epic-key>
jira issue list --epics <epic-key> -s"To Do" --plain
```
Pick the single **highest-priority** issue in **To Do** (Highest → High → Medium → Low → Lowest; ties by CLI order). No pick list. Do **not** fall back to other statuses/resolutions — if empty, stop.

**Issue mode:**
```bash
jira issue list -s"To Do" -a"$(jira me)" --plain
```
Pick the highest-priority To Do issue assigned to you.

**If nothing in To Do** → **STOP LOOP**, print:
`grind: no issues in "To Do" — <scope>. Stopping loop.`
(Do not pick "Product planning" — that needs a human.)

### 2. Read and classify

```bash
jira issue assign <key> "$(jira me)"
jira issue view <key>
```
(Leave the issue in **To Do** while reading — do not move it to In Review.)

Classify work type from summary/description/labels:
- **Code work** → continue
- **Non-code work** (doc, deck, plan, research, comms) → **SKIP**: move it **out of the To Do queue** so the next cycle does not re-pick the same issue (which would spin the loop forever):
  ```bash
  jira issue move <key> "Backlog"
  jira issue edit <key> --label needs-human --no-input
  ```
  Print `grind: <key> is non-code — moved to Backlog (needs-human), skipping.` Continue loop; do not implement.
- **Ambiguous** → SKIP same as non-code (move to Backlog + `needs-human`). Never guess on autonomous runs.

### 3. Branch (code work)

**Project mode:** if already on `<epic-slug>-YYYY-MM-DD` for today, stay. Else:
```bash
git checkout main && git pull
git checkout -b <epic-slug>-YYYY-MM-DD
```

**Issue mode:**
```bash
git checkout main && git pull
git checkout -b <key>-<slug>      # slug from summary, lowercase, dashes
```

(`main` default; `develop` only if repo has no `main`.)

### 4. Implement

```bash
jira issue move <key> "In Progress"
```

Implement the **minimal** solution against the acceptance criteria. Rules:
- Read before coding; follow existing patterns
- Focused changes only — no unrelated refactors
- If acceptance criteria cannot be met without a product decision → **STOP LOOP**, leave issue In Progress, print `grind: <key> needs a decision — <what>. Stopping loop.`
- Run the project's tests/build if present. Failing tests you cannot fix → **STOP LOOP**.

### 5. Pre-ship gate (mandatory, before any push)

1. **Diff review** — read `git status` and the **full** `git diff`. Stage only changes that belong to <key>; drop debug prints, stray files, and unrelated edits. Never blind-stage with `git add -A`.
2. **Local verification** — detect and run the project's test and build commands (re-run after the diff review even if step 4 already ran them). Record the exact commands and results — they go into the PR body. Unfixable failure → **STOP LOOP**.
3. **Self-review** — run the **code-review** skill (Phases 1–2: Understand → Audit) on `git diff <base>..HEAD`. Critical/High findings → fix, then re-run verification. Unfixable Critical/High → **STOP LOOP**, leave issue In Progress, print `grind: <key> failed self-review — <finding>. Stopping loop.`

### 6. Ship (follow `/done`, non-interactive)

```bash
git remote show origin | grep 'HEAD branch'   # detect <base>, default main
git log --oneline <base>..HEAD
git diff --stat <base>..HEAD
```

No commits → **STOP LOOP**, print `grind: <key> produced no commits. Stopping loop.`

Stage + commit (`PROJ-12: …` per change; project mode may use `<epic-slug>: <summary>`), then:
```bash
git push -u origin HEAD
```

Push rejected → follow the **github-cli** skill's "Push rejected (diverged history)" procedure; **conflicts → STOP LOOP**, list files, never force-push.

Create the PR — reference the issue key in title/body so the Jira GitHub integration links it, with a test plan containing the **real results** from the pre-ship gate:
```bash
gh pr create --title "PROJ-12: …" --body "$(cat <<'EOF'
## Summary

- What changed and why

## Test plan

- `<test command>` — <result>
- `<build command>` — <result>

PROJ-12
EOF
)"
```
Print PR URL.

### 7. CI + merge (never merge directly)

```bash
gh pr checks --watch
```
- **Fail → STOP LOOP**, print `grind: CI failed on <PR> — stopping. Fix and re-run.` Do not pick the next issue on a broken base.
- **No checks configured → STOP LOOP.** Leave the PR open, print `grind: no CI on this repo — refusing to auto-merge <PR>. Review and merge manually.` An autonomous loop never merges unvalidated code; the local pre-ship gate is not a substitute for CI on `main`.
- **Pass** → arm auto-merge so GitHub (branch protection) decides when it lands:
  ```bash
  gh pr merge --auto --squash --delete-branch
  ```
  - Merges immediately (no review required) → continue to step 8.
  - Stays open awaiting approval → **STOP LOOP** (clean finish), print `grind: PR <#> armed to auto-merge, awaiting review. Stopping — re-run after approval.` Do not stack further cycles on an unmerged base.
  - Repo does not allow auto-merge (`--auto` rejected) → checks passed and the pre-ship gate ran, so fall back to `gh pr merge --squash --delete-branch`; suggest enabling auto-merge + branch protection (see the github-cli skill).

### 8. Transition + return to base

```bash
jira issue move <key> "Done"      # if integration did not auto-transition on merge
git checkout <base> && git pull
```

Each cycle ships its own PR and merges with `--delete-branch`, so the branch is gone after merge. **Project mode:** the next cycle re-creates `<epic-slug>-YYYY-MM-DD` fresh from the updated `main` (step 3). Do **not** try to reuse or re-push the merged branch — its commits are already squashed onto `main`, so reuse would diverge.

---

## Loop control

Each `/grind` run = exactly **one** cycle (one issue shipped or one skip). The loop wrapper repeats it. **End the loop** (omit the next wakeup) on any **STOP LOOP** above. Use **self-paced** loop timing — work duration varies, there is nothing to poll on a clock.

On success, print a one-line receipt and let the loop fire the next cycle:
```
grind ✓ PROJ-12 merged (PR #214). Next cycle.
```

---

## Stop conditions (summary)

| Condition | Action |
|-----------|--------|
| No issues in To Do | STOP LOOP (clean finish) |
| Non-code / ambiguous issue | SKIP, continue loop |
| Needs product decision | STOP LOOP, issue left In Progress |
| Tests/build fail (unfixable) | STOP LOOP |
| Critical/High self-review finding (unfixable) | STOP LOOP, issue left In Progress |
| No commits produced | STOP LOOP |
| Rebase conflict | STOP LOOP, no force-push |
| CI fail | STOP LOOP |
| No CI checks configured | STOP LOOP, PR left open for manual review |
| PR awaiting required review | STOP LOOP (clean — auto-merge armed) |

Never: prompt the user, guess work type, force-push, merge with failing or absent CI, or transition to Done before merge.
