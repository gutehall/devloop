# /grind - Autonomous one-cycle: pick → implement → ship

One full development cycle in a single command: `/next` + implement + `/done`, with **no interactive questions**. Built to be wrapped in `/loop` so a whole project drains unattended.

```
/loop /grind project       # drain a project, one issue per cycle, until empty
/loop /grind issue         # one issue per branch per cycle
```

Run bare (`/grind project`) to do exactly one cycle and stop.

## Why this exists

`/next` and `/done` are interactive — they ask scope, work type, base branch. Under `/loop` those questions block forever. `/grind` resolves every decision automatically from branch + config, and where it cannot, it **stops the loop** instead of hanging or guessing.

---

## Scope (never ask)

Resolve scope in this order — do **not** prompt:

1. Explicit arg: `/grind project` or `/grind issue`
2. Current branch: `<project-slug>-YYYY-MM-DD` → project; `TEAM-123-*` → issue
3. Default project in `.linear` (`project=`) → project mode

If none resolve → **STOP LOOP**, print: `grind: scope unresolved — run /grind project or /grind issue once to seed.`

---

## Cycle

### 1. Pick the issue (follow `/next`, non-interactive)

**Project mode:**
```bash
linear project show "<project>"
linear issues --project "<project>" --unblocked
```
Pick the single **highest-priority** unblocked issue (urgent → high → medium → low → none; ties by CLI order). No pick list.

**Issue mode:**
```bash
linear issues --unblocked
```
Pick the highest-priority unblocked issue assigned to you (CLI sorts yours first).

**If nothing unblocked** → **STOP LOOP**, print:
`grind: no unblocked work — N blocked, M open. Stopping loop.`
(Do not fall back to blocked issues. Do not pick "Product planning" — that needs a human.)

### 2. Read and classify

```bash
linear issue update <id> --status "In Review"
linear issue show <id>
```

Classify work type from title/description/labels:
- **Code work** → continue
- **Non-code work** (doc, deck, plan, research, comms) → **SKIP**: set status back, print `grind: <id> is non-code — skipping, needs a human.` Continue loop (try next cycle); do not implement.
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
- If acceptance criteria cannot be met without a product decision → **STOP LOOP**, leave issue In Progress, print `grind: <id> needs a decision — <what>. Stopping loop.`
- Run the project's tests/build if present. Failing tests you cannot fix → **STOP LOOP**.

### 5. Ship (follow `/done`, non-interactive)

```bash
git remote show origin | grep 'HEAD branch'   # detect <base>, default main
git log --oneline <base>..HEAD
git diff --stat <base>..HEAD
```

No commits → **STOP LOOP**, print `grind: <id> produced no commits. Stopping loop.`

Stage + commit (`ISSUE-12: …` per change; project mode may use `<project-slug>: <summary>`), then:
```bash
git push -u origin HEAD
```

Push rejected → `git fetch origin && git rebase origin/<base>`. Conflicts → **STOP LOOP**, list files, do not force-push.

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
- **Fail → STOP LOOP**, print `grind: CI failed on <PR> — stopping. Fix and re-run.` Do not pick the next issue on a broken base.

### 7. Return to base
```bash
git checkout <base> && git pull
```

**Project mode:** do **not** delete the project branch between cycles — `/loop` reuses it for the day. Re-checkout it next cycle.

---

## Loop control

Each `/grind` run = exactly **one** cycle (one issue shipped or one skip). The loop wrapper repeats it. **End the loop** (omit the next wakeup) on any **STOP LOOP** above. Use **self-paced** loop timing — work duration varies, there is nothing to poll on a clock.

On success, print a one-line receipt and let the loop fire the next cycle:
```
grind ✓ ISSUE-12 merged (PR #214). Next cycle.
```

---

## Stop conditions (summary)

| Condition | Action |
|-----------|--------|
| No unblocked work | STOP LOOP (clean finish) |
| Non-code / ambiguous issue | SKIP, continue loop |
| Needs product decision | STOP LOOP, issue left In Progress |
| Tests/build fail (unfixable) | STOP LOOP |
| No commits produced | STOP LOOP |
| Rebase conflict | STOP LOOP, no force-push |
| CI fail | STOP LOOP |

Never: prompt the user, pick blocked work, guess work type, force-push, or `linear issue close` before merge.
