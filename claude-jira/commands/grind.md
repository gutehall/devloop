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
jira issue move <key> "In Review"
jira issue assign <key> "$(jira me)"
jira issue view <key>
```

Classify work type from summary/description/labels:
- **Code work** → continue
- **Non-code work** (doc, deck, plan, research, comms) → **SKIP**: move status back, print `grind: <key> is non-code — skipping, needs a human.` Continue loop; do not implement.
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
- If acceptance criteria cannot be met without a product decision → **STOP LOOP**, leave issue In Progress, print `grind: <key> needs a decision — <what>. Stopping loop.`
- Run the project's tests/build if present. Failing tests you cannot fix → **STOP LOOP**.

### 5. Ship (follow `/done`, non-interactive)

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

Push rejected → `git fetch origin && git rebase origin/<base>`. Conflicts → **STOP LOOP**, list files, do not force-push.

Create the PR (reference the issue key in title/body so the Jira GitHub integration links it):
```bash
gh pr create --title "PROJ-12: …" --body "…\n\nPROJ-12"
```
Print PR URL.

### 6. CI + merge

```bash
gh pr checks --watch
```
- Pass / no checks → `gh pr merge --squash --delete-branch`
- **Fail → STOP LOOP**, print `grind: CI failed on <PR> — stopping. Fix and re-run.` Do not pick the next issue on a broken base.

### 7. Transition + return to base

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
| No commits produced | STOP LOOP |
| Rebase conflict | STOP LOOP, no force-push |
| CI fail | STOP LOOP |

Never: prompt the user, guess work type, force-push, or transition to Done before merge.
