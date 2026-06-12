# /done - Complete work and ship

Works for any type of issue — code, documents, decks, reviews, planning, ops tasks. Supports **project** mode (one PR for the epic branch) or **issue** mode (one PR for a single issue).

## Usage

```
/done                      # Ask project vs issue, then proceed
/done project              # Ship the current epic branch
/done issue                # Ship the current issue
/done project PROJ-100     # Specific epic (when branch is ambiguous)
/done issue PROJ-12        # Specific issue
```

---

## Choose scope (always ask first)

Unless the user already chose via `project` or `issue` in the command:

> **Ship at project level or for a single issue?**
> - **Issue** (recommended) — one PR closing the issue on this issue branch. Smaller diff, its own CI run, far easier to review.
> - **Project** — one PR closing all issues worked on this epic branch. Use only when the issues are inseparable — a multi-issue squashed PR is the hardest artifact to review.

Wait for the answer before pushing, creating a PR, or closing issues.

**Skip the question** when:
- The command includes `project` or `issue`
- The current branch clearly indicates mode: `<epic-slug>-YYYY-MM-DD` → project; `PROJ-123-*` → issue

---

## Work Type Detection

- Epic branch or `PROJ-123-*` issue branch with git repo → code work
- No matching branch → non-code work
- If ambiguous, ask

---

## Quality gate (both code paths — mandatory before any push)

Run all three checks, in order, after the work summary and before pushing. Their results feed the PR body. **Do not push until all three pass.**

### G1. Review the diff, then commit

Read `git status` and the **full** `git diff` (plus `git diff <base>..HEAD` for already-committed work) before staging anything:

- Stage only changes that belong to this issue/epic
- Leave out debug prints, commented-out code, stray files, and unrelated edits — list anything excluded
- Never blind-stage with `git add -A` without reading what it picks up

Then commit what survived. Prefer `PROJ-12: …` per change; an epic branch may use `<epic-slug>: <summary>`.

### G2. Verify locally

Detect the project's test and build commands (`package.json` scripts, `Makefile`, `pyproject.toml`, CI workflow files) and run them.

- Tests fail or build breaks → **stop. Do not push.** Fix it or report to the user.
- No test/build tooling exists → say so explicitly; it must also be stated in the PR body.

Record the exact commands and their results — they go into the PR body's test plan.

### G3. Self-review the code

Run the **code-review** skill (Phases 1–2: Understand → Audit) against the full diff (`git diff <base>..HEAD`):

- **Critical/High** findings → fix now, re-run G2, then continue
- **Medium** → fix now, or list under "Known issues" in the PR body — never silently drop
- **Low** → note in the PR body or propose a follow-up issue

---

## Path: Project — ship the epic

### Resolve the epic

1. From branch: map `<epic-slug>-YYYY-MM-DD` via `jira epic list --plain`
2. If epic key provided, use it
3. Otherwise ask which epic this branch completes

### 1. Gather epic issues

```bash
jira issue view <epic-key>
jira issue list --epics <epic-key> -s"In Progress" --plain
git log --oneline <base>..HEAD
```

Union In Progress children with `[A-Z]+-[0-9]+` from commits.

### 2–3. Base branch and work summary

Detect `<base>` (`git remote show origin | grep 'HEAD branch'`, default `main`), then show `git log --oneline <base>..HEAD` and `git diff --stat <base>..HEAD`. List every issue key in this push.

### 4. Run the quality gate

Complete **G1–G3** above. All three must pass before anything is pushed.

### 5. Push branch

```bash
git push -u origin HEAD
```

### 6. Create one project PR

Title: `<Epic summary>: <short summary>`

Body — one `Closes PROJ-ID` per issue, and a test plan filled with the **real commands and results from G2** (never an unchecked checkbox):

```bash
gh pr create --title "Auth System: Caching fixes" --body "$(cat <<'EOF'
## Summary

- What changed across the epic

## Issues closed

Closes PROJ-10
Closes PROJ-12

## Test plan

- `npm test` — 42 passed, 0 failed
- `npm run build` — clean

## Known issues

- (Medium/Low findings from G3 not fixed in this PR, if any)
EOF
)"
```

Print the PR URL immediately.

### 7. Wait for CI

```bash
gh pr checks --watch
```

- Pass → continue to merge
- Fail → stop; fix and run `/done project` again
- **No checks configured** → there is no CI gate; the quality gate (G1–G3) was the only validation. Tell the user this repo has no CI, suggest branch protection (see the github-cli skill), and do **not** merge without their explicit confirmation.

### 8. Merge (confirm first)

Show the PR URL, CI result, and quality-gate results, then ask: **"Merge now?"** Do not merge without a yes.

```bash
gh pr merge --squash --delete-branch
```

If branch protection requires an approving review, arm auto-merge instead and finish — GitHub merges once a reviewer approves:

```bash
gh pr merge --auto --squash --delete-branch
```

### 9. Return to base

```bash
git checkout <base> && git pull
```

### 10. Complete the epic

- Unresolved children remain → list them; offer `/next project`
- None remain → `jira issue move <epic-key> "Done"` (or workflow epic-done transition)

### Project code rules

- One PR per epic branch from `/done project`
- Without GitHub-Jira integration: `jira issue move <id> "Done"` per issue after merge

---

## Path: Issue — ship a single issue

### 1. Detect the issue

From branch (`PROJ-123-*`) or provided key. Fallback: ask which issue.

### 2. Detect base branch

```bash
git remote show origin | grep 'HEAD branch'
```

### 3. Show work summary

```bash
git log --oneline <base>..HEAD
git diff --stat <base>..HEAD
```

### 4. Run the quality gate

Complete **G1–G3** above. All three must pass before anything is pushed.

### 5. Push branch

```bash
git push -u origin HEAD
```

### 6. Create issue PR

Test plan must contain the **real commands and results from G2** — never an unchecked checkbox:

```bash
gh pr create --title "PROJ-12: Issue summary" --body "$(cat <<'EOF'
## Summary

- What changed and why

## Test plan

- `npm test` — 42 passed, 0 failed
- `npm run build` — clean

Closes PROJ-12
EOF
)"
```

Print the PR URL immediately.

### 7. Wait for CI

```bash
gh pr checks --watch
```

- Pass → continue to merge
- Fail → stop; fix and run `/done issue` again
- **No checks configured** → there is no CI gate; the quality gate (G1–G3) was the only validation. Tell the user this repo has no CI, suggest branch protection (see the github-cli skill), and do **not** merge without their explicit confirmation.

### 8. Merge (confirm first)

Show the PR URL, CI result, and quality-gate results, then ask: **"Merge now?"** Do not merge without a yes.

```bash
gh pr merge --squash --delete-branch
```

If branch protection requires an approving review, arm auto-merge instead and finish — GitHub merges once a reviewer approves:

```bash
gh pr merge --auto --squash --delete-branch
```

### 9. Return to base

```bash
git checkout <base> && git pull
```

### Issue code rules

- PR title and body must include issue key; `Closes PROJ-42` for auto-transition
- Without integration: `jira issue move PROJ-42 "Done"` after merge
- Smart Commits: `PROJ-42 #done #comment …` if DVCS connector is enabled

### If push is rejected (both code paths)

Follow the **github-cli** skill's "Push rejected (diverged history)" procedure. On conflict, stop, list files, re-run `/done`. Never force-push.

---

## Path B: Non-Code Work

**Project:** summarize epic deliverables, artifact links via `jira issue comment add`, `jira issue move` Done per child, epic Done if complete.

**Issue:** summarize, artifact comment, `jira issue move <key> "Done"`.

Offer `/next` with matching scope.

---

## Notes

- Scope question comes **before** git operations
- `/done project` pairs with `/next project`; `/done issue` with `/next issue`
- Check branch pattern when work type is unclear
- The quality gate (G1–G3) is prompt-level discipline — make it unbypassable with branch protection on `main` (required checks + 1 approving review). See the github-cli skill's "Branch protection" section.
