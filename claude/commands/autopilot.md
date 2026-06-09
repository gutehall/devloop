# /autopilot - Allowlisted autonomous one-cycle: pick → implement → ship

`/grind`, but **hard-gated to issues labelled `auto-claude`**. Built for an unattended Claude Code instance: it may only ever touch work a human has explicitly opted in by adding the `auto-claude` label. Everything else is invisible to it.

```
/loop /autopilot project       # drain the allowlisted queue in a project, one issue per cycle
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
2. Current branch: `<project-slug>-YYYY-MM-DD` → project; `TEAM-123-*` → issue
3. Default project in `.linear` (`project=`) → project mode

If none resolve → **STOP LOOP**, print: `autopilot: scope unresolved — run /autopilot project or /autopilot issue once to seed.`

**PR model:** ships **one PR per issue every cycle** in both scopes (same as `/grind`). "Project mode" only changes *which queue* issues are picked from, not how they ship.

---

## Cycle

### 1. Pick the issue (allowlisted only)

The queue is **`Ready for build` AND labelled `auto-claude`** — never anything else.

**Project mode:**
```bash
linear project show "<project>"
linear issues --project "<project>" --status "Ready for build" --label "auto-claude"
```
Pick the single **highest-priority** issue (urgent → high → medium → low → none; ties by CLI order). No pick list.

**Issue mode:**
```bash
linear issues --status "Ready for build" --label "auto-claude"
```
Pick the highest-priority issue assigned to you (CLI sorts yours first).

**If nothing in the allowlisted queue** → **STOP LOOP**, print:
`autopilot: no issues in "Ready for build" labelled auto-claude. Stopping loop.`
(Do not fall back to unlabelled issues, blocked issues, or other statuses. Do not pick "Product planning" — that needs a human.)

### 2. Read, verify the label, and classify

```bash
linear issue show <id>
```
(Leave the issue in **Ready for build** while reading — do not set it to In Review.)

**Hard gate — verify the allowlist before doing anything else.** Confirm the issue shown actually carries the `auto-claude` label. If it does **not** (filter returned a stale or unexpected result) → **STOP LOOP**, touch nothing, print:
`autopilot: <id> is not labelled auto-claude — refusing to work it. Stopping loop.`

Then classify work type from title/description/labels:
- **Code work** → continue
- **Non-code work** (doc, deck, plan, research, comms) → **SKIP**: move it **out of the queue** so the next cycle does not re-pick it:
  ```bash
  linear issue update <id> --status "Backlog" --label "needs-human"
  ```
  Print `autopilot: <id> is non-code — moved to Backlog (needs-human), skipping.` Continue loop; do not implement. (Leave the `auto-claude` label in place so a human can re-queue it after handling.)
- **Ambiguous** → SKIP same as non-code. Never guess on autonomous runs.

### 3. Branch (code work)

**Project mode:** if already on `<project-slug>-YYYY-MM-DD` for today, stay. Else:
```bash
git checkout main && git pull
git checkout -b <project-slug>-YYYY-MM-DD
```

**Issue mode:**
```bash
git checkout main && git pull
linear branch <id>
```

(`main` default; `develop` only if repo has no `main`.)

### 4. Implement

```bash
linear issue start <id>          # → In Progress
```

Implement the **minimal** solution against the acceptance criteria. Rules:
- Read before coding; follow existing patterns
- Focused changes only — no unrelated refactors
- If acceptance criteria cannot be met without a product decision → **STOP LOOP**, leave issue In Progress, print `autopilot: <id> needs a decision — <what>. Stopping loop.`
- Run the project's tests/build if present. Failing tests you cannot fix → **STOP LOOP**.

### 5. Ship (non-interactive)

```bash
git remote show origin | grep 'HEAD branch'   # detect <base>, default main
git log --oneline <base>..HEAD
git diff --stat <base>..HEAD
```

No commits → **STOP LOOP**, print `autopilot: <id> produced no commits. Stopping loop.`

Stage + commit (`ISSUE-12: …` per change; project mode may use `<project-slug>: <summary>`), then:
```bash
git push -u origin HEAD
```

Push rejected → follow the **github-cli** skill's "Push rejected (diverged history)" procedure; **conflicts → STOP LOOP**, list files, never force-push.

Create the PR (`Closes <ID>` in body — required for Linear integration):
```bash
gh pr create --title "ISSUE-12: …" --body "…Closes ISSUE-12"
```
Print PR URL.

### 6. CI + merge

```bash
gh pr checks --watch
```
- Pass / no checks → `gh pr merge --squash --delete-branch`
- **Fail → STOP LOOP**, print `autopilot: CI failed on <PR> — stopping. Fix and re-run.` Do not pick the next issue on a broken base.

### 7. Return to base
```bash
git checkout <base> && git pull
```

Each cycle ships its own PR and merges with `--delete-branch`, so the branch is gone after merge. **Project mode:** the next cycle re-creates `<project-slug>-YYYY-MM-DD` fresh from the updated `main` (step 3). Do **not** reuse or re-push the merged branch — its commits are already squashed onto `main`, so reuse would diverge.

---

## Loop control

Each `/autopilot` run = exactly **one** cycle (one issue shipped or one skip). The loop wrapper repeats it. **End the loop** (omit the next wakeup) on any **STOP LOOP** above. Use **self-paced** loop timing — work duration varies, there is nothing to poll on a clock.

On success, print a one-line receipt and let the loop fire the next cycle:
```
autopilot ✓ ISSUE-12 merged (PR #214). Next cycle.
```

---

## Stop conditions (summary)

| Condition | Action |
|-----------|--------|
| No allowlisted (`auto-claude`) issues in Ready for build | STOP LOOP (clean finish) |
| Picked issue is missing the `auto-claude` label | STOP LOOP, touch nothing |
| Non-code / ambiguous issue | SKIP, continue loop |
| Needs product decision | STOP LOOP, issue left In Progress |
| Tests/build fail (unfixable) | STOP LOOP |
| No commits produced | STOP LOOP |
| Rebase conflict | STOP LOOP, no force-push |
| CI fail | STOP LOOP |

Never: prompt the user, work an issue without the `auto-claude` label, pick blocked work, guess work type, force-push, or `linear issue close` before merge.
