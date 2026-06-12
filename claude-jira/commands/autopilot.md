# /autopilot - Allowlisted autonomous one-cycle: pick → implement → ship

`/grind`, but **hard-gated to issues labelled `auto-claude`**. Built for an unattended Claude Code instance: it may only ever touch work a human has explicitly opted in by adding the `auto-claude` label. Everything else is invisible to it.

```
/loop /autopilot project       # drain the allowlisted queue in an epic, one issue per cycle
/loop /autopilot issue          # one allowlisted issue per branch per cycle
```

Run bare (`/autopilot project`) to do exactly one cycle and stop.

> **Safety contract:** the automated instance is configured to run **only** `/autopilot` — never `/grind` or `/next`. `/autopilot` never reads, branches from, comments on, transitions, or implements any issue that does not carry the `auto-claude` label. If it ever picks up an issue without the label, it **stops the loop** rather than proceeding.

## Why this exists

`/grind` will pick up any issue in the ready queue. For an unsupervised instance that is too broad — a human must be able to decide, per issue, "yes, the bot may do this one." The `auto-claude` label is that switch. `/autopilot` is `/grind` with that switch enforced on every query and re-checked before any write.

---

## Scope (never ask)

Resolve scope in this order — do **not** prompt:

1. Explicit arg: `/autopilot project` or `/autopilot issue`
2. Current branch: `<epic-slug>-YYYY-MM-DD` → project; `PROJ-123-*` → issue
3. Default epic in config → project mode

If none resolve → **STOP LOOP**, print: `autopilot: scope unresolved — run /autopilot project or /autopilot issue once to seed.`

**PR model:** ships **one PR per issue every cycle** in both scopes (same as `/grind`). "Project mode" only changes *which queue* issues are picked from, not how they ship.

---

## Cycle

### 1. Pick the issue (allowlisted only)

The queue is **`To Do` AND labelled `auto-claude`** — never anything else.

**Project mode (epic):**
```bash
jira issue view <epic-key>
jira issue list --epics <epic-key> -s"To Do" -l"auto-claude" --plain
```
Pick the single **highest-priority** issue (Highest → High → Medium → Low → Lowest; ties by CLI order). No pick list. Do **not** fall back to other statuses, resolutions, or unlabelled issues — if empty, stop.

**Issue mode:**
```bash
jira issue list -s"To Do" -l"auto-claude" -a"$(jira me)" --plain
```
Pick the highest-priority To Do issue assigned to you.

**If nothing in the allowlisted queue** → **STOP LOOP**, print:
`autopilot: no issues in "To Do" labelled auto-claude — <scope>. Stopping loop.`
(Do not pick "Product planning" — that needs a human.)

### 2. Read, verify the label, and classify

```bash
jira issue assign <key> "$(jira me)"
jira issue view <key>
```
(Leave the issue in **To Do** while reading — do not move it to In Review.)

**Hard gate — verify the allowlist before doing anything else.** Confirm the issue shown actually carries the `auto-claude` label. If it does **not** (filter returned a stale or unexpected result) → **STOP LOOP**, touch nothing further, print:
`autopilot: <key> is not labelled auto-claude — refusing to work it. Stopping loop.`

Then classify work type from summary/description/labels:
- **Code work** → continue
- **Non-code work** (doc, deck, plan, research, comms) → **SKIP**: move it **out of the To Do queue** so the next cycle does not re-pick it:
  ```bash
  jira issue move <key> "Backlog"
  jira issue edit <key> --label needs-human --no-input
  ```
  Print `autopilot: <key> is non-code — moved to Backlog (needs-human), skipping.` Continue loop; do not implement. (Leave the `auto-claude` label in place so a human can re-queue it after handling.)
- **Ambiguous** → SKIP same as non-code. Never guess on autonomous runs.

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
- If acceptance criteria cannot be met without a product decision → **STOP LOOP**, leave issue In Progress, print `autopilot: <key> needs a decision — <what>. Stopping loop.`
- Run the project's tests/build if present. Failing tests you cannot fix → **STOP LOOP**.

### 5. Pre-ship gate (mandatory, before any push)

1. **Diff review** — read `git status` and the **full** `git diff`. Stage only changes that belong to <key>; drop debug prints, stray files, and unrelated edits. Never blind-stage with `git add -A`.
2. **Local verification** — detect and run the project's test and build commands (re-run after the diff review even if step 4 already ran them). Record the exact commands and results — they go into the PR body. Unfixable failure → **STOP LOOP**.
3. **Self-review** — run the **code-review** skill (Phases 1–2: Understand → Audit) on `git diff <base>..HEAD`. Critical/High findings → fix, then re-run verification. Unfixable Critical/High → **STOP LOOP**, leave issue In Progress, print `autopilot: <key> failed self-review — <finding>. Stopping loop.`

### 6. Ship (non-interactive)

```bash
git remote show origin | grep 'HEAD branch'   # detect <base>, default main
git log --oneline <base>..HEAD
git diff --stat <base>..HEAD
```

No commits → **STOP LOOP**, print `autopilot: <key> produced no commits. Stopping loop.`

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

An unattended instance never runs a direct `gh pr merge` — merging is GitHub's decision via auto-merge + branch protection, or a human's.

```bash
gh pr checks --watch
```
- **Fail → STOP LOOP**, print `autopilot: CI failed on <PR> — stopping. Fix and re-run.` Do not pick the next issue on a broken base.
- **No checks configured → STOP LOOP.** Leave the PR open, print `autopilot: no CI on this repo — refusing to merge <PR>. Review and merge manually.`
- **Pass** → arm auto-merge:
  ```bash
  gh pr merge --auto --squash --delete-branch
  ```
  - Merges immediately (no review required) → continue to step 8.
  - Stays open awaiting approval → **STOP LOOP** (clean finish), print `autopilot: PR <#> armed to auto-merge, awaiting review. Stopping — re-run after approval.` Do not stack further cycles on an unmerged base.
  - Repo does not allow auto-merge (`--auto` rejected) → **STOP LOOP**, leave the PR open, print `autopilot: cannot arm auto-merge on <PR> — review and merge manually. Stopping loop.` Never fall back to a direct merge.

### 8. Transition + return to base

```bash
jira issue move <key> "Done"      # if integration did not auto-transition on merge
git checkout <base> && git pull
```

Each cycle ships its own PR and merges with `--delete-branch`, so the branch is gone after merge. **Project mode:** the next cycle re-creates `<epic-slug>-YYYY-MM-DD` fresh from the updated `main` (step 3). Do **not** reuse or re-push the merged branch — its commits are already squashed onto `main`, so reuse would diverge.

---

## Loop control

Each `/autopilot` run = exactly **one** cycle (one issue shipped or one skip). The loop wrapper repeats it. **End the loop** (omit the next wakeup) on any **STOP LOOP** above. Use **self-paced** loop timing — work duration varies, there is nothing to poll on a clock.

On success, print a one-line receipt and let the loop fire the next cycle:
```
autopilot ✓ PROJ-12 merged (PR #214). Next cycle.
```

---

## Stop conditions (summary)

| Condition | Action |
|-----------|--------|
| No allowlisted (`auto-claude`) issues in To Do | STOP LOOP (clean finish) |
| Picked issue is missing the `auto-claude` label | STOP LOOP, touch nothing |
| Non-code / ambiguous issue | SKIP, continue loop |
| Needs product decision | STOP LOOP, issue left In Progress |
| Tests/build fail (unfixable) | STOP LOOP |
| Critical/High self-review finding (unfixable) | STOP LOOP, issue left In Progress |
| No commits produced | STOP LOOP |
| Rebase conflict | STOP LOOP, no force-push |
| CI fail | STOP LOOP |
| No CI checks configured | STOP LOOP, PR left open for manual review |
| PR awaiting required review | STOP LOOP (clean — auto-merge armed) |
| Auto-merge not allowed by repo settings | STOP LOOP, PR left open — never direct-merge |

Never: prompt the user, work an issue without the `auto-claude` label, pick blocked work, guess work type, force-push, run a direct `gh pr merge`, or transition to Done before merge.
