# /next - Find and start the next piece of work

Works for any type of issue — code, documents, decks, reviews, planning, ops tasks. Supports **project** mode (one branch, multiple issues) or **issue** mode (one branch per issue).

## Usage

```
/next                      # Ask project vs issue, then proceed
/next project              # Project mode (skip scope question)
/next issue                # Issue mode (skip scope question)
/next project "Phase 1"    # Named project
/next issue ISSUE-12       # Specific issue
/next initiative "Name"    # Next unblocked issue across a Linear initiative's projects
```

---

## Choose scope (always ask first)

Unless the user already chose via `project` or `issue` in the command:

> **Work at project level or on a single issue?**
> - **Issue** (recommended) — one branch per issue; `/done` ships that issue only. Smaller PRs, individually CI-gated, easier to review.
> - **Project** — one branch for the whole project; `/next` picks the highest-priority issue; `/done` ships everything in one PR. Use only when the issues are inseparable.

Wait for the answer before continuing. Do not load issues or create branches until scope is confirmed.

**Skip the question** when:
- The command includes `project`, `issue`, or `initiative` (e.g. `/next project`, `/next issue ISSUE-12`, `/next initiative "Name"`)
- The current branch clearly indicates mode: `<project-slug>-YYYY-MM-DD` → project; `TEAM-123-*` → issue

---

## Path: Project

Pick the highest-priority issue in **Ready for build** in a Linear project and work on a shared project branch. Run `/next project` again to advance to the next issue on the same branch.

### Resolve the project

1. **If a project name is provided:** use it (aliases from `.linear` work).
2. **If an issue ID was provided with project scope:** run `linear issue show <id>`, read its project, start on that issue (skip priority selection).
3. **Otherwise:** read the default project from config (`.linear` `project=`). If none, run `linear projects` and ask the user to pick one.

### Load project and pick the next issue

1. Run `linear project show "<project>"` for context, milestones, and progress.
2. **If no issue override:** run `linear issues --project "<project>" --status "Ready for build"`.
3. **Select the highest-priority issue** — do not present a pick list. Priority order: urgent → high → medium → low → none. Break ties by CLI sort order.
4. If **Ready for build** returns nothing, see [No work available](#no-work-available). Do **not** fall back to other statuses.
5. Run `linear issue show <id>` — read description and acceptance criteria. Leave the issue in **Ready for build** while you read; do not change status yet.

Set the issue to **In Progress** (`linear issue start <id>`) once you begin actually working it. (`In Review` is for an open PR under review, not for reading.)

Announce:

```
Mode: Project
Project: Phase 1 (58% complete)
Working on: ISSUE-12 — Add caching layer [urgent]
```

### Project branch (code work)

Branch name: `<project-slug>-YYYY-MM-DD` (derive slug from project name; max 40 chars; lowercase, spaces → `-`, strip special chars).

- If already on matching project branch for today, stay on it.
- Otherwise detect `<base>` once (origin HEAD branch, else `main`) and branch from it:
  ```bash
  git checkout <base>
  git pull
  git checkout -b <project-slug>-YYYY-MM-DD
  ```
- Do **not** run `linear branch <id>`.

Commit messages: `ISSUE-12: Short description`

When ready to ship the project branch, tell the user to run `/done project`.

---

## Path: Initiative

Pick the next unblocked **Ready for build** issue across all projects of a Linear initiative, then work it like issue mode (one branch, one PR). Requires an initiative name; none given → run `mcp__claude_ai_Linear__list_initiatives` and ask the user to pick.

### Pick the issue

1. **Team.** Use `.linear` `team=` (or the team of the initiative's issues if only one). Pass `team` on every Linear MCP query. Prefer MCP over the `linear` CLI for listing here — the CLI's default team may differ.
2. **Projects, once:** `mcp__claude_ai_Linear__list_projects` with `initiative: "<name>"`, `fields: ["name", "status", "priority"]`. Drop Completed and Canceled. Order: In Progress first, then Planned, then Backlog; ties by priority, then list order.
3. **Walk the projects in that order.** For each: `mcp__claude_ai_Linear__list_issues` with `team`, `project`, `state: "Ready for build"`, `fields: ["id", "title", "priority", "assignee"]`. **Ready for build only** — never fall back to other statuses.
4. **Skip issues assigned to someone else.** Keep an issue only if unassigned or assigned to the current user (resolve once with `mcp__claude_ai_Linear__get_user` `query: "me"`). Print `skip <id> — assigned to <name>`. Do not filter by `assignee: "me"` in the query — that drops unassigned issues.
5. **Skip blocked issues.** For each candidate, `mcp__claude_ai_Linear__get_issue` with `includeRelations: true`. For each `blockedBy` entry, `get_issue` with `fields: ["status", "statusType"]`. Blocked = any blocker not `completed`/`canceled`. Print `skip <id> — blocked by <blocker> (<status>)`.
6. **Stop at the first project with an unblocked issue.** Do not load later projects once one yields a pick.
7. **Pick the highest priority** there (urgent → high → medium → low → none), **assigned-to-me first**: if any unblocked candidate is yours, choose only among those; if none is, name the highest-priority unassigned candidate and ask (`AskUserQuestion`) whether to take it; on no, report blockers and stop. The `get_issue` from step 5 is the one read of the issue — do not fetch it again.

No unblocked Ready issue anywhere → list blocked candidates and their blockers; suggest `/plan`.

Announce:

```
Mode: Initiative (<name>)
Project: <project> [In Progress]
Working on: ISSUE-123 — <title> [high]
Skipped: ISSUE-120 (blocked by ISSUE-118, In Progress)
```

### Initiative branch (code work)

One branch per issue, in the current repo. Detect `<base>` once (origin HEAD branch, else `main`):

```bash
git checkout <base>
git pull
git checkout -b <gitBranchName from get_issue>
git config --local push.autoSetupRemote true   # plain `git push` works on the new branch
```

Commit messages: `ISSUE-123: Short description`. PR body: `Closes ISSUE-123`.

Set the issue In Progress with `mcp__claude_ai_Linear__save_issue` (`id`, `state: "In Progress"`, `assignee: "me"`) when you begin working it.

**Then the project:** if the picked project's live status is Backlog or Planned, move it with `mcp__claude_ai_Linear__save_project` (`id`, `state: "In Progress"`) and print `Project <name>: <old> → In Progress`. Any other status → leave it. Only after work on the issue actually begins. Never move a project back; completion stays manual.

Then run **Path A: Code Work** steps 2, 4, 5 (prior-work check, explore, implement, including **Who implements**) — or **Path B** for non-code issues (no branch).

When ready to ship, tell the user to run `/done issue`.

---

## Path: Issue

Find a **Ready for build** issue, branch per issue, implement one issue at a time. `/done issue` ships only that issue.

### If no issue ID is provided

1. Run `linear issues --status "Ready for build"`
2. Present interactive options:
   - Up to **3 issues** (CLI sorts yours first)
   - **Always** include "Product planning"
   - If >3 issues, note how many more
   - User can type a specific issue ID

### Starting work on an issue

When the user selects an issue (or provided `/next issue ISSUE-12`):

1. Run `linear issue show <id>` — read description, acceptance criteria. Leave it in **Ready for build** while you read.

Then, **when you begin actually working** (writing code or producing the deliverable):

2. Run `linear issue start <id>` — assigns to you + sets **In Progress**

Announce:

```
Mode: Issue
Working on: ISSUE-12 — Add caching layer
```

### Issue branch (code work)

- Detect `<base>` once (origin HEAD branch, else `main`) and branch from it:
  ```bash
  git checkout <base>
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

1. Read description and acceptance criteria (issue stays in **Ready for build** while you read)
2. **Prior-work check** — run the **prior-work** skill: has this already been solved (merged PR, existing code, open branch, or duplicate issue)? Default the code search to **1–3 targeted greps/globs on the main thread** (symbols/filenames from the AC). Do **not** spawn a locator subagent for a typical issue — that setup cost is why `/next` got expensive. Spawn at most **one** cheap locator (`cavecrew-investigator` if that type exists, else Task/Explore on Haiku) only when the issue is clearly a feature / multi-file change. Never spawn 2–3 in parallel here. This search doubles as explore — do not re-search in step 4 unless there are gaps. Before presenting a "shipped" or "duplicate" finding, verify the cited PR/commit/code yourself. Then present the finding and ask (proceed / close as done / extend existing / mark duplicate / quit) before branching. Do not skip this.
3. Set up the branch per scope (project or issue rules above)
4. Explore relevant code — reuse step 2. No extra locators unless step 2 left a gap on a multi-file issue.
5. Move the issue to **In Progress**, then implement (see **Who implements** below) — minimal solution, follow existing patterns. **Now** apply project conventions: if this repo has a conventions skill, invoke it; else follow root/nearest `CLAUDE.md`, `AGENTS.md`, or `CONTRIBUTING.md` if present. Do not load those during pick / prior-work / branch.

**Implementation rules:** read before coding; focused changes only; no unrelated refactors; check acceptance criteria; flag scope creep.

#### Who implements (sub-agents)

Applies to every scope (project, issue, initiative). The lead (this session) implements by default. When the issue touches more than about 3 files:

- Spawn one `context` agent per disjoint place (cap 3; one more round only if a returned pack lists a place under **Missing**). One agent per place; none roams.
- Then the lead may spawn **one** specialist — `backend`, `frontend`, or `docs` — for the bulk of the change. No parallel writers.
- Pass every agent the absolute checkout path, the issue ID and AC, and the conventions you read (agents do not inherit your conversation). Pass specialists the context packs and the files they may change.
- **Lead only, no specialist:** issues labelled `risk:high`, `security`, or `billing`, and anything touching auth, credentials, payments, or data migrations.
- The lead reviews the specialist's diff, then commits, pushes (`git push -u origin HEAD`), and updates Linear. Specialists never do.
- Agent output is a lead, not proof — read cited spans before relying on them.

**Watchable panes (tmux):** give every spawned agent a `name` (`context-<place>`, `backend`, `frontend`, `docs`). With agent teams enabled (`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`) and the session inside tmux (`teammateMode` `tmux` or `auto`), each named agent opens in its own pane so the user can follow it. Otherwise the same calls run as normal subagents — never fail or ask over missing panes.

**Close panes:** named agents stay open and idle after they reply. Stop each with `TaskStop` (`task_id` = its `name`) as soon as it is done: each `context` agent once its pack is read, the specialist once its diff is reviewed. Before telling the user to run `/done`, confirm none is left running. Stopping a name that is already gone is fine.

### Path B: Non-Code Work

1. Read description and acceptance criteria (issue stays in **Ready for build** while you read)
2. **Prior-work check** — run the **prior-work** skill: does this deliverable (or a duplicate issue) already exist? If so, present the finding and ask the next step before producing anything new.
3. Identify the deliverable
4. Move the issue to **In Progress**, then help produce it
5. When ready, tell the user to run `/done` with the same scope they used (`/done project` or `/done issue`)

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

## No issues in Ready for build (issue mode)

1. Run `linear issues --open`
2. All blocked → show blockers; offer `/plan`
3. Empty backlog → offer Product planning or `/plan "..."`
4. Linear not configured → say to run `linear login`

## Notes

- Scope question comes **before** loading work; branching is automatic from the repo default branch (`origin` HEAD, else `main`)
- Set default project with `linear project open "<name>"` for faster project mode
- Match `/done` scope to `/next` scope: `/done project` vs `/done issue`
