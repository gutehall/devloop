# /next - Find and start the next piece of work

Works for any type of issue — code, documents, decks, reviews, planning, ops tasks. Supports **project** mode (one branch, multiple issues in an epic) or **issue** mode (one branch per issue).

In Jira, **project mode** uses an epic as the batch unit (Linear's equivalent of a project).

## Usage

```
/next                      # Ask project vs issue, then proceed
/next project              # Project/epic mode (skip scope question)
/next issue                # Issue mode (skip scope question)
/next project PROJ-100     # Specific epic
/next issue PROJ-12        # Specific issue
```

---

## Choose scope (always ask first)

Unless the user already chose via `project` or `issue` in the command:

> **Work at project level or on a single issue?**
> - **Project** — one branch for the epic; `/next` picks the highest-priority child issue; `/done` ships everything in one PR
> - **Issue** — one branch per issue; `/done` ships that issue only

Wait for the answer before continuing. Do not load issues or create branches until scope is confirmed.

**Skip the question** when:
- The command includes `project` or `issue`
- The current branch clearly indicates mode: `<epic-slug>-YYYY-MM-DD` → project; `PROJ-123-*` → issue

---

## Path: Project (epic)

Pick the highest-priority ready issue in an epic on a shared project branch. Run `/next project` again for the next issue on the same branch.

### Resolve the epic

1. **If an epic key or name is provided:** use it (`jira epic list --plain` to resolve names).
2. **If an issue key was provided with project scope:** `jira issue view <key>`, read parent epic, start on that issue (skip priority selection).
3. **Otherwise:** `jira epic list --plain` — pick active epic or ask the user.

### Load epic and pick the next issue

1. `jira issue view <epic-key>`
2. `jira issue list --epics <epic-key> -s"To Do" --plain`
3. If empty: `jira issue list --epics <epic-key> --resolution Unresolved --plain`
4. **Highest priority** — no pick list. Order: Highest → High → Medium → Low → Lowest.

Announce:

```
Mode: Project (epic)
Epic: PROJ-100 — Auth System
Working on: PROJ-12 — Add caching layer [High]
```

### Start work

1. `jira issue move <id> "In Progress"`
2. `jira issue assign <id> "$(jira me)"`
3. `jira issue view <id>`

### Project branch (code work)

Branch: `<epic-slug>-YYYY-MM-DD` (from epic summary; max 40 chars; slug rules as Linear project slug).

- If already on matching epic branch for today, stay on it.
- Otherwise ask: "Branch from `main` or `develop`?" — wait, then:
  ```bash
  git checkout <main|develop>
  git pull
  git checkout -b <epic-slug>-YYYY-MM-DD
  ```

Commit messages: `PROJ-12: Short description`

When ready, tell the user to run `/done project`.

---

## Path: Issue

Find a To Do issue, branch per issue, implement one at a time. `/done issue` ships only that issue.

### If no issue key is provided

1. `jira issue list -s"To Do" -a"$(jira me)" --plain`
2. If empty: `jira issue list -s"To Do" --plain`
3. Present interactive options:
   - Up to **3 issues**
   - **Always** include "Product planning"
   - If >3, note how many more
   - User can type a specific key

### Starting work

1. `jira issue move <key> "In Progress"`
2. `jira issue assign <key> "$(jira me)"`
3. `jira issue view <key>`

Announce:

```
Mode: Issue
Working on: PROJ-12 — Add caching layer
```

### Issue branch (code work)

- Ask: "Branch from `main` or `develop`?" — wait, then:
  ```bash
  git checkout <main|develop>
  git pull
  ```
- Derive slug from summary (lowercase, dashes, max 50 chars):
  ```bash
  git checkout -b PROJ-12-<slug>
  ```

When ready, tell the user to run `/done issue`.

---

## Work Type Detection (both paths)

**Code work** — implementation, bug fix, feature, refactor, etc.; git repo present; or type Bug/Story/Task.

**Non-code work** — document, deck, plan, review, research, comms, analysis, spec, strategy.

If ambiguous, ask: "Is this code work or non-code work?"

### Path A: Code Work

Set up branch per scope, read criteria, explore code, implement minimally.

### Path B: Non-Code Work

Produce the deliverable; run `/done project` or `/done issue` when ready. No git branch.

---

## Product planning (issue mode only)

If the user chooses "Product planning", follow the product-planning skill.

---

## No work available

**Project:** no To Do children → show status; offer `/plan`.

**Issue:** no To Do → show In Progress/blocked; offer `/plan`.

**Jira not configured:** say to run `jira init`.

## Notes

- Scope question comes **before** branch question
- Match `/done` scope: `/done project` vs `/done issue`
- `jira me` for self-assignment
