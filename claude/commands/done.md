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
> - **Project** — one PR closing all issues worked on this project branch
> - **Issue** — one PR closing the issue on this issue branch

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

### 4. Stage and commit

Commit uncommitted changes. Prefer `ISSUE-12: …` per change; else `<project-slug>: <summary>`.

### 5. Push branch

```bash
git push -u origin HEAD
```

### 6. Create one project PR

Title: `<Project Name>: <short summary>`

Body — one `Closes ISSUE-ID` per issue:

```bash
gh pr create --title "Phase 1: Caching and auth improvements" --body "$(cat <<'EOF'
## Summary

- What changed across the project

## Issues closed

Closes ISSUE-10
Closes ISSUE-12

## Test plan

- [ ] Tested locally
EOF
)"
```

Print the PR URL immediately.

### 7. Wait for CI

```bash
gh pr checks --watch
```

- Pass → merge
- Fail → stop; fix and run `/done project` again
- No checks → merge immediately

### 8. Merge

```bash
gh pr merge --squash --delete-branch
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

### 4. Stage and commit

Stage and commit any uncommitted changes.

### 5. Push branch

```bash
git push -u origin HEAD
```

### 6. Create issue PR

```bash
gh pr create --title "ISSUE-12: Issue title" --body "$(cat <<'EOF'
## Summary

- What changed and why

## Test plan

- [ ] Tested locally

Closes ISSUE-12
EOF
)"
```

Print the PR URL immediately.

### 7. Wait for CI

```bash
gh pr checks --watch
```

- Pass → merge
- Fail → stop; fix and run `/done issue` again
- No checks → merge immediately

### 8. Merge

```bash
gh pr merge --squash --delete-branch
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
