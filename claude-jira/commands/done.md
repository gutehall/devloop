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
> - **Project** — one PR closing all issues worked on this epic branch
> - **Issue** — one PR closing the issue on this issue branch

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

### 2–5. Base branch, summary, commit, push

Same as Linear project path: detect `<base>`, show log/diff, commit, `git push -u origin HEAD`.

### 6. Create one project PR

Title: `<Epic summary>: <short summary>`

Body — one `Closes PROJ-ID` per issue:

```bash
gh pr create --title "Auth System: Caching fixes" --body "$(cat <<'EOF'
## Summary

- What changed across the epic

## Issues closed

Closes PROJ-10
Closes PROJ-12

## Test plan

- [ ] Tested locally
EOF
)"
```

### 7–9. CI, merge, return to base

```bash
gh pr checks --watch
gh pr merge --squash --delete-branch
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

### 3–5. Summary, commit, push

`git log`, `git diff --stat`, commit, `git push -u origin HEAD`.

### 6. Create issue PR

```bash
gh pr create --title "PROJ-12: Issue summary" --body "$(cat <<'EOF'
## Summary

- What changed and why

## Test plan

- [ ] Tested locally

Closes PROJ-12
EOF
)"
```

### 7–9. CI, merge, return to base

Same watch/merge/checkout/pull pattern as project path.

### Issue code rules

- PR title and body must include issue key; `Closes PROJ-42` for auto-transition
- Without integration: `jira issue move PROJ-42 "Done"` after merge
- Smart Commits: `PROJ-42 #done #comment …` if DVCS connector is enabled

### If push is rejected (both code paths)

Rebase on `<base>`; stop on conflicts; no force-push unless requested.

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
