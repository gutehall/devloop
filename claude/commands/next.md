# /next - Find and start the next piece of work

Works for any type of issue — code, documents, decks, reviews, planning, ops tasks. Supports **project** mode (one branch, multiple issues) or **issue** mode (one branch per issue).

## Usage

```
/next                      # Ask project vs issue, then proceed
/next project              # Project mode (skip scope question)
/next issue                # Issue mode (skip scope question)
/next project "Phase 1"    # Named project
/next issue ISSUE-12       # Specific issue
```

---

## Choose scope (always ask first)

Unless the user already chose via `project` or `issue` in the command:

> **Work at project level or on a single issue?**
> - **Project** — one branch for the whole project; `/next` picks the highest-priority issue; `/done` ships everything in one PR
> - **Issue** — one branch per issue; `/done` ships that issue only

Wait for the answer before continuing. Do not load issues or create branches until scope is confirmed.

**Skip the question** when:
- The command includes `project` or `issue` (e.g. `/next project`, `/next issue ISSUE-12`)
- The current branch clearly indicates mode: `<project-slug>-YYYY-MM-DD` → project; `TEAM-123-*` → issue

---

## Path: Project

Pick the highest-priority unblocked issue in a Linear project and work on a shared project branch. Run `/next project` again to advance to the next issue on the same branch.

### Resolve the project

1. **If a project name is provided:** use it (aliases from `.linear` work).
2. **If an issue ID was provided with project scope:** run `linear issue show <id>`, read its project, start on that issue (skip priority selection).
3. **Otherwise:** read the default project from config (`.linear` `project=`). If none, run `linear projects` and ask the user to pick one.

### Load project and pick the next issue

1. Run `linear project show "<project>"` for context, milestones, and progress.
2. **If no issue override:** run `linear issues --project "<project>" --unblocked`.
3. **Select the highest-priority issue** — do not present a pick list. Priority order: urgent → high → medium → low → none. Break ties by CLI sort order.
4. If `--unblocked` returns nothing, run `linear issues --project "<project>" --open` and pick the highest-priority open issue. If still empty, see [No work available](#no-work-available).
5. Run `linear issue update <id> --status "In Review"` — you are now reading/learning the issue.
6. Run `linear issue show <id>` — read description and acceptance criteria.

Set the issue to **In Progress** (`linear issue start <id>`, or `linear issue update <id> --status "In Progress"`) once you begin actually working it.

Announce:

```
Mode: Project
Project: Phase 1 (58% complete)
Working on: ISSUE-12 — Add caching layer [urgent]
```

### Project branch (code work)

Branch name: `<project-slug>-YYYY-MM-DD` (derive slug from project name; max 40 chars; lowercase, spaces → `-`, strip special chars).

- If already on matching project branch for today, stay on it.
- Otherwise branch from `main` (default; use `develop` if the repo has no `main`):
  ```bash
  git checkout main
  git pull
  git checkout -b <project-slug>-YYYY-MM-DD
  ```
- Do **not** run `linear branch <id>`.

Commit messages: `ISSUE-12: Short description`

When ready to ship the project branch, tell the user to run `/done project`.

---

## Path: Issue

Find an unblocked issue, branch per issue, implement one issue at a time. `/done issue` ships only that issue.

### If no issue ID is provided

1. Run `linear issues --unblocked`
2. Present interactive options:
   - Up to **3 issues** (CLI sorts yours first)
   - **Always** include "Product planning"
   - If >3 issues, note how many more
   - User can type a specific issue ID

### Starting work on an issue

When the user selects an issue (or provided `/next issue ISSUE-12`):

1. Run `linear issue update <id> --status "In Review"` — you are now reading/learning the issue
2. Run `linear issue show <id>` — read description, acceptance criteria

Then, **when you begin actually working** (writing code or producing the deliverable):

3. Run `linear issue start <id>` — assigns to you + sets **In Progress**

Announce:

```
Mode: Issue
Working on: ISSUE-12 — Add caching layer
```

### Issue branch (code work)

- Branch from `main` (default; use `develop` if the repo has no `main`):
  ```bash
  git checkout main
  git pull
  linear branch <id>
  ```

When ready to ship, tell the user to run `/done issue`.

---

## Work Type Detection (both paths)

Classify from title, description, and labels:

**Code work** — implementation, bug fix, feature, refactor, migration, API, infrastructure, test; or repo present.

**Non-code work** — document, report, deck, presentation, plan, review, research, process, comms, meeting, analysis, spec, strategy.

If ambiguous, ask: "Is this code work or non-code work?"

### Path A: Code Work

1. Set up the branch per scope (project or issue rules above)
2. Read description and acceptance criteria (issue is **In Review** while you read)
3. Explore relevant code
4. Move the issue to **In Progress**, then implement — minimal solution, follow existing patterns

**Implementation rules:** read before coding; focused changes only; no unrelated refactors; check acceptance criteria; flag scope creep.

### Path B: Non-Code Work

1. Read description and acceptance criteria (issue is **In Review** while you read)
2. Identify the deliverable
3. Move the issue to **In Progress**, then help produce it
4. When ready, tell the user to run `/done` with the same scope they used (`/done project` or `/done issue`)

No git branch for non-code work.

---

## Product planning (issue mode only)

If the user chooses "Product planning", ask what to focus on (backlog, features, next phase) and follow the product-planning skill.

---

## No work available (project mode)

1. Run `linear issues --project "<project>" --open`
2. All blocked → show blockers; suggest `/plan`
3. No open issues → offer `/plan` or `linear project complete "<project>"`
4. Linear not configured → say to run `linear login`

## No unblocked issues (issue mode)

1. Run `linear issues --open`
2. All blocked → show blockers; offer `/plan`
3. Empty backlog → offer Product planning or `/plan "..."`
4. Linear not configured → say to run `linear login`

## Notes

- Scope question comes **before** loading work; branching is automatic from `main`
- Set default project with `linear project open "<name>"` for faster project mode
- Match `/done` scope to `/next` scope: `/done project` vs `/done issue`
