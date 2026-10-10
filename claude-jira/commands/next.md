# /next - Find and start the next piece of work

Works for any type of issue — code, documents, decks, reviews, planning, ops tasks. Supports **project** mode (one branch, multiple issues in an epic) or **issue** mode (one branch per issue).

**Project mode** uses an epic as the batch unit — one branch covers every issue in the epic.

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
> - **Issue** (recommended) — one branch per issue; `/done` ships that issue only. Smaller PRs, individually CI-gated, easier to review.
> - **Project** — one branch for the epic; `/next` picks the highest-priority child issue; `/done` ships everything in one PR. Use only when the issues are inseparable.

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
3. If empty, see [No work available](#no-work-available). Do **not** fall back to other statuses/resolutions.
4. **Highest priority** — no pick list. Order: Highest → High → Medium → Low → Lowest.

Announce:

```
Mode: Project (epic)
Epic: PROJ-100 — Auth System
Working on: PROJ-12 — Add caching layer [High]
```

### Start work

1. `jira issue view <id>` — read criteria. Leave it in **To Do** while you read.
2. `jira issue assign <id> "$(jira me)"`

Then, **when you begin actually working** (writing code or producing the deliverable):

3. `jira issue move <id> "In Progress"`

### Project branch (code work)

Branch: `<epic-slug>-YYYY-MM-DD` (from epic summary; max 40 chars; lowercase, hyphenated slug).

- If already on matching epic branch for today, stay on it.
- Otherwise detect `<base>` once (origin HEAD branch, else `main`) and branch from it:
  ```bash
  git checkout <base>
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

1. `jira issue view <key>` — read criteria. Leave it in **To Do** while you read.
2. `jira issue assign <key> "$(jira me)"`

Then, **when you begin actually working** (writing code or producing the deliverable):

3. `jira issue move <key> "In Progress"`

Announce:

```
Mode: Issue
Working on: PROJ-12 — Add caching layer
```

### Issue branch (code work)

- Detect `<base>` once (origin HEAD branch, else `main`) and branch from it:
  ```bash
  git checkout <base>
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

Read criteria (issue stays in **To Do** while reading), then run the **prior-work** skill — has this already been solved (merged PR, existing code, open branch, or duplicate issue)? Default the code search to **1–3 targeted greps/globs on the main thread**. Do **not** spawn a locator subagent for a typical issue. Spawn at most **one** cheap locator (`cavecrew-investigator` if that type exists, else Task/Explore on Haiku) only when the issue is clearly a feature / multi-file change. Never spawn 2–3 in parallel. Verify cited PR/commit/code yourself before close/skip. This search doubles as explore. If it finds a match, present the finding and ask (proceed / close as done / extend existing / mark duplicate / quit) before branching. Then set up branch per scope, explore code (reuse prior-work results), move to **In Progress**, and implement minimally (see **Who implements** below). **Now** apply project conventions: if this repo has a conventions skill, invoke it; else follow root/nearest `CLAUDE.md`, `AGENTS.md`, or `CONTRIBUTING.md` if present. Do not load those during pick / prior-work / branch.

**Implementation rules:** read before coding; focused changes only; no unrelated refactors; check acceptance criteria; flag scope creep.

#### Who implements (sub-agents)

Applies to every scope (project, issue). The lead (this session) implements by default. When the issue touches more than about 3 files:

- Spawn one `context` agent per disjoint place (cap 3; one more round only if a returned pack lists a place under **Missing**). One agent per place; none roams.
- Then the lead may spawn **one** specialist — `backend`, `frontend`, or `docs` — for the bulk of the change. No parallel writers.
- Pass every agent the absolute checkout path, the issue ID and AC, and the conventions you read (agents do not inherit your conversation). Pass specialists the context packs and the files they may change.
- **Lead only, no specialist:** issues labelled `risk:high`, `security`, or `billing`, and anything touching auth, credentials, payments, or data migrations.
- The lead reviews the specialist's diff, then commits, pushes (`git push -u origin HEAD`), and updates Jira. Specialists never do.
- Agent output is a lead, not proof — read cited spans before relying on them.

**Watchable panes (tmux):** give every spawned agent a `name` (`context-<place>`, `backend`, `frontend`, `docs`). With agent teams enabled (`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`) and the session inside tmux (`teammateMode` `tmux` or `auto`), each named agent opens in its own pane so the user can follow it. Otherwise the same calls run as normal subagents — never fail or ask over missing panes.

**Close panes:** named agents stay open and idle after they reply. Stop each with `TaskStop` (`task_id` = its `name`) as soon as it is done: each `context` agent once its pack is read, the specialist once its diff is reviewed. Before telling the user to run `/done`, confirm none is left running. Stopping a name that is already gone is fine.

### Path B: Non-Code Work

Read criteria (issue stays in **To Do** while reading), then run the **prior-work** skill — does this deliverable (or a duplicate issue) already exist? If so, present the finding and ask the next step before producing anything. Then move to **In Progress** and produce the deliverable; run `/done project` or `/done issue` when ready. No git branch.

---

## Product planning (issue mode only)

If the user chooses "Product planning", ask what to focus on (backlog, features, next phase) and follow the product-planning skill.

---

## No work available (project mode)

1. `jira issue list --epics <epic-key> --plain`
2. All blocked → show blockers; suggest `/plan`
3. No open issues in the epic → offer `/plan` or `jira issue move <epic-key> "Done"` to close the epic
4. Jira not configured → say to run `jira init`

## No issues in To Do (issue mode)

1. `jira issue list -a"$(jira me)" --plain`
2. All blocked → show blockers; offer `/plan`
3. Empty backlog → offer Product planning or `/plan "..."`
4. Jira not configured → say to run `jira init`

## Notes

- Scope question comes **before** loading work; branching is automatic from the repo default branch (`origin` HEAD, else `main`)
- Match `/done` scope: `/done project` vs `/done issue`
- `jira me` for self-assignment
