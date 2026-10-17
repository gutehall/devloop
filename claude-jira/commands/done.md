# /done - Complete work and ship

> **Project conventions:** (1) If this repo has a skill whose description says it defines coding rules, verify/test commands, or pre-merge gates — invoke it. (2) Else if `CLAUDE.md`, `AGENTS.md`, or `CONTRIBUTING.md` exists at the repo root (then the nearest subdirectory you are editing) — follow those. (3) Else continue.

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

- Epic branch (`<epic-slug>-YYYY-MM-DD`) or `PROJ-123-*` issue branch with git repo → code work → **Ship sequence** below
- No matching branch → non-code work → **Path B**
- If ambiguous, ask

---

## Ship sequence (code work — both scopes)

Mandatory quality gate before any push (step 4: G1 commit, G2 verify, G3 size-gated review). Do not push until G1–G3 pass; their results feed the PR body. Where a step differs by scope it is split **Issue:** / **Project:** — otherwise it is identical for both.

### 1. Identify what you're shipping

**Issue:** from branch name (`PROJ-123` pattern) or the provided key. If the branch does not match: `git log --oneline -5`, then ask "Which issue does this complete? (e.g., PROJ-12)".

**Project:** map `<epic-slug>-YYYY-MM-DD` back via `jira epic list --plain`; if an epic key was provided, use it; otherwise ask which epic this branch completes. Then gather its issues:

```bash
jira issue view <epic-key>
jira issue list --epics <epic-key> -s"In Progress" --plain
```

Union the In Progress children with `[A-Z]+-[0-9]+` patterns from commit messages. If `jira issue view <epic-key>` was already fetched earlier this session and no issue status/comment changed since, reuse it (see prior-work skill's session-cache note).

### 2. Detect base branch

`<base>` = origin HEAD branch (`git remote show origin | grep 'HEAD branch'`), else `main`; never `develop`. Compute it once here and reuse it for every step below — do not re-derive it later in this run.

### 3. Show work summary

```bash
git log --oneline <base>..HEAD
git diff --stat <base>..HEAD
```

Reuse this output later in this run instead of re-running it. **Project:** also list every issue key in this push.

### 4. Run the quality gate

Follow the **ship-gate** skill in **interactive** mode (G1 commit, G2 verify, G3 size-gated review). All three checks must pass before anything is pushed.

### 5. Push branch

```bash
git push -u origin HEAD
```

### 6. Create the PR

If a PR already exists for this branch (`gh pr view`), report that URL instead of creating another. Otherwise `gh pr create --title "..." --body "..."` and print the PR URL immediately.

- **Title:** issue `PROJ-12: <summary>`; project `<Epic>: <summary>`
- **Body sections:** `## Summary` (what changed and why); `## Test plan` (the real G2 commands and results — never an unchecked box); one `Closes <ID>` per issue; `## Known issues` only for project scope (unresolved suggestions or scope observations from G3 not fixed here)

### 7. Wait for CI

```bash
gh pr checks --watch
```

- Pass → continue to merge
- Fail → stop; fix and run `/done` again with the same scope
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

(Issue mode: also `git log --oneline -5` to confirm the merge landed.)

### 10. Close out

**Issue:** the PR body's `Closes PROJ-42` auto-transitions the Jira issue on merge (when GitHub-Jira integration is enabled). Without integration, run `jira issue move PROJ-42 "Done"` after merge. Smart Commits (`PROJ-42 #done #comment …`) also work if the DVCS connector is enabled. If you used a worktree, show the cleanup command (`git worktree remove <path>`).

**Project:** if unresolved children remain, list them and offer `/next project`. If none remain, `jira issue move <epic-key> "Done"` (or the workflow's epic-done transition). Without integration, `jira issue move <key> "Done"` per child after merge.

### Code rules (both scopes)

- One PR per branch from `/done`; use `/done project` after `/next project`, `/done issue` after `/next issue`
- PR title and body must include each issue key; `Closes PROJ-ID` per issue for auto-transition
- Do **not** transition an issue to Done before merge — integration transitions on merge
- No commits → skip PR and note it
- **Push rejected (both scopes):** follow the **github-cli** skill's "Push rejected (diverged history)" procedure. On conflict, stop, list files, tell the user to resolve and re-run `/done`. Never force-push.

---

## Path B: Non-Code Work

Ask scope if not already chosen, then:

**Project:** summarize epic deliverables, artifact links via `jira issue comment add`, `jira issue move` Done per child, epic Done if complete.

**Issue:** summarize deliverable, artifact comment via `jira issue comment add`, `jira issue move <key> "Done"`.

Offer `/next` with matching scope. No git, no PR.

---

## Notes

- Scope question comes **before** git operations
- `/done project` pairs with `/next project`; `/done issue` with `/next issue`
- Check branch pattern when work type is unclear
- The quality gate (G1–G3) is prompt-level discipline — make it unbypassable with branch protection on `main` (required checks + 1 approving review). See the github-cli skill's "Branch protection" section.
