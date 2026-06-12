# /done - Complete work and ship

Works for any type of issue — code, documents, decks, reviews, planning, ops tasks. Supports **project** mode (one PR for the whole project branch) or **issue** mode (one PR for a single issue).

## Usage

```
/done                      # Ask project vs issue, then proceed
/done project              # Ship the current project branch
/done issue                # Ship the current issue
/done project "Phase 1"    # Named project (when branch is ambiguous)
/done issue ISSUE-12       # Specific issue
```

---

## Choose scope (always ask first)

Unless the user already chose via `project` or `issue` in the command:

> **Ship at project level or for a single issue?**
> - **Issue** (recommended) — one PR closing the issue on this issue branch. Smaller diff, its own CI run, far easier to review.
> - **Project** — one PR closing all issues worked on this project branch. Use only when the issues are inseparable — a multi-issue squashed PR is the hardest artifact to review.

Wait for the answer before pushing, creating a PR, or closing issues.

**Skip the question** when:
- The command includes `project` or `issue`
- The current branch clearly indicates mode: `<project-slug>-YYYY-MM-DD` → project; `TEAM-123-*` → issue

---

## Work Type Detection

- Project branch (`<project-slug>-YYYY-MM-DD`) or issue branch (`TEAM-123-*`) with git repo → code work
- No matching branch → non-code work
- If ambiguous, ask

---

## Quality gate (both code paths — mandatory before any push)

Run all three checks, in order, after the work summary and before pushing. Their results feed the PR body. **Do not push until all three pass.**

### G1. Review the diff, then commit

Read `git status` and the **full** `git diff` (plus `git diff <base>..HEAD` for already-committed work) before staging anything:

- Stage only changes that belong to this issue/project
- Leave out debug prints, commented-out code, stray files, and unrelated edits — list anything excluded
- Never blind-stage with `git add -A` without reading what it picks up

Then commit what survived. Prefer `ISSUE-12: …` per change; a project branch may use `<project-slug>: <summary>`.

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

## Path: Project — ship the whole project

### Resolve the project

1. From branch name: map `<project-slug>-YYYY-MM-DD` back via `linear projects`
2. If a project name is provided, use it
3. Otherwise ask which project this branch completes

### 1. Gather project issues

```bash
linear project show "<project>"
linear issues --project "<project>" --status "In Progress"
git log --oneline <base>..HEAD
```

Union In Progress issues with `TEAM-123` patterns from commit messages.

### 2. Detect base branch

```bash
git remote show origin | grep 'HEAD branch'
```

Default to `main`. Use `<base>` throughout.

### 3. Show work summary

```bash
git log --oneline <base>..HEAD
git diff --stat <base>..HEAD
```

List every issue ID in this project push.

### 4. Run the quality gate

Complete **G1–G3** above. All three must pass before anything is pushed.

### 5. Push branch

```bash
git push -u origin HEAD
```

### 6. Create one project PR

Title: `<Project Name>: <short summary>`

Body — one `Closes ISSUE-ID` per issue, and a test plan filled with the **real commands and results from G2** (never an unchecked checkbox):

```bash
gh pr create --title "Phase 1: Caching and auth improvements" --body "$(cat <<'EOF'
## Summary

- What changed across the project

## Issues closed

Closes ISSUE-10
Closes ISSUE-12

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
git checkout <base>
git pull
```

### 10. Complete the Linear project

- `linear issues --project "<project>" --open` — if none remain: `linear project complete "<project>"`
- If issues remain, list them and offer `/next project`

### Project code rules

- One PR per project branch from `/done project`
- Do not run `linear issue close` before merge — GitHub integration closes on merge
- No commits → skip PR and note it

---

## Path: Issue — ship a single issue

### 1. Detect the issue

From branch name (`TEAM-123` pattern) or the provided ID. If branch does not match:

1. `git log --oneline -5`
2. Ask: "Which issue does this complete? (e.g., FIN-42)"

### 2. Detect base branch

```bash
git remote show origin | grep 'HEAD branch'
```

Default to `main`.

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
gh pr create --title "ISSUE-12: Issue title" --body "$(cat <<'EOF'
## Summary

- What changed and why

## Test plan

- `npm test` — 42 passed, 0 failed
- `npm run build` — clean

Closes ISSUE-12
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
git checkout <base>
git pull
git log --oneline -5
```

### Issue code rules

- PR body **must** contain `Closes <ID>` for Linear GitHub integration
- Do **not** run `linear issue close` before merge
- If integration does not close the issue after merge, run `linear issue close <id>` as fallback
- No commits → skip PR and note it
- Worktree: show cleanup commands after PR creation

### If push is rejected (both code paths)

Follow the **github-cli** skill's "Push rejected (diverged history)" procedure. On conflict, stop, list files, tell the user to resolve and re-run `/done`. Never force-push.

---

## Path B: Non-Code Work

Ask scope if not already chosen, then:

**Project:** identify project, summarize all deliverables, collect artifact links, `linear issue close` per completed issue, `linear project complete` if done.

**Issue:** identify issue, summarize deliverable, artifact link via `mcp__claude_ai_Linear__save_comment`, `linear issue close <id>`.

Offer `/next` with matching scope.

No git, no PR.

---

## Notes

- Scope question comes **before** git operations
- Use `/done project` after `/next project` and `/done issue` after `/next issue`
- When in doubt about work type, check the branch pattern first
- The quality gate (G1–G3) is prompt-level discipline — make it unbypassable with branch protection on `main` (required checks + 1 approving review). See the github-cli skill's "Branch protection" section.
