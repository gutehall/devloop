# /done - Complete work and ship

> **Project conventions:** follow the **ship-gate** skill's "Project conventions" rule (conventions skill → `CLAUDE.md`/`AGENTS.md`/`CONTRIBUTING.md` → detect from tooling).

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

- Project branch (`<project-slug>-YYYY-MM-DD`) or issue branch (`TEAM-123-*`) with git repo → code work → **Ship sequence** below
- No matching branch → non-code work → **Path B**
- If ambiguous, ask

---

## Ship sequence (code work — both scopes)

Mandatory quality gate before any push: follow the **ship-gate** skill in **interactive** mode (G1 commit, G2 verify, G3 size-gated review). Do not push until G1–G3 pass; their results feed the PR body. Where a step differs by scope it is split **Issue:** / **Project:** — otherwise it is identical for both.

### 1. Identify what you're shipping

**Issue:** from branch name (`TEAM-123` pattern) or the provided ID. If the branch does not match: `git log --oneline -5`, then ask "Which issue does this complete? (e.g., ISSUE-12)".

**Project:** map `<project-slug>-YYYY-MM-DD` back via `linear projects`; if a project name was provided, use it; otherwise ask which project this branch completes. Then gather its issues:

```bash
linear project show "<project>"
linear issues --project "<project>" --status "In Progress"
```

Union the In Progress issues with `TEAM-123` patterns from commit messages. If `linear project show "<project>"` was already fetched earlier this session and no issue status/comment changed since, reuse it (see prior-work skill's session-cache note).

### 2. Detect base branch

Follow the **ship-gate** skill's base-branch rule. Compute `<base>` once here and reuse it for every step below — do not re-derive it later in this run.

### 3. Show work summary

```bash
git log --oneline <base>..HEAD
git diff --stat <base>..HEAD
```

Reuse this output later in this run instead of re-running it. **Project:** also list every issue ID in this push.

### 4. Run the quality gate

Follow the **ship-gate** skill in **interactive** mode. All three checks must pass before anything is pushed.

### 5. Push branch

```bash
git push -u origin HEAD
```

### 6. Create the PR

If a PR already exists for this branch (`gh pr view`), report the existing URL instead of creating a duplicate. The test plan must contain the **real commands and results from G2** — never an unchecked checkbox. Print the PR URL immediately.

**Issue** — title `ISSUE-12: Issue title`:

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

**Project** — title `<Project Name>: <short summary>`; one `Closes ISSUE-ID` per issue; add a Known issues section for unresolved Medium/Low findings from G3:

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
git checkout <base>
git pull
```

(Issue mode: also `git log --oneline -5` to confirm the merge landed.)

### 10. Close out

**Issue:** the PR body's `Closes <ID>` moves the Linear issue on merge. If the integration does not close it, run `linear issue close <id>` as a fallback. If the `linear` CLI cannot resolve the ID ("Entity not found: Issue" — e.g. the issue's team isn't the CLI default, as with `/next initiative`), do the Linear reads and writes with the MCP tools instead: `mcp__claude_ai_Linear__get_issue`, `mcp__claude_ai_Linear__save_comment`, and `mcp__claude_ai_Linear__save_issue` with `state: "Done"` as the post-merge fallback. If you used a worktree, show the cleanup command (`git worktree remove <path>`).

**Project:** `linear issues --project "<project>" --open` — if none remain, `linear project complete "<project>"`. If issues remain, list them and offer `/next project`.

### Code rules (both scopes)

- One PR per branch from `/done`; use `/done project` after `/next project`, `/done issue` after `/next issue`
- PR body **must** contain `Closes <ID>` for each issue (triggers Linear's GitHub integration)
- `/done issue` ships a `/next initiative` pick too (same branch-per-issue flow)
- Do **not** run `linear issue close` before merge — GitHub integration closes on merge
- No commits → skip PR and note it
- **Push rejected (both scopes):** follow the **github-cli** skill's "Push rejected (diverged history)" procedure. On conflict, stop, list files, tell the user to resolve and re-run `/done`. Never force-push.

---

## Path B: Non-Code Work

Ask scope if not already chosen, then:

**Project:** identify project, summarize all deliverables, collect artifact links, `linear issue close` per completed issue, `linear project complete` if done.

**Issue:** identify issue, summarize deliverable, artifact link via `mcp__claude_ai_Linear__save_comment`, `linear issue close <id>`.

Offer `/next` with matching scope. No git, no PR.

---

## Notes

- Scope question comes **before** git operations
- When in doubt about work type, check the branch pattern first
- The quality gate (G1–G3) is prompt-level discipline — make it unbypassable with branch protection on `main` (required checks + 1 approving review). See the github-cli skill's "Branch protection" section.
