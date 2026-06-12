# DevLoop

Claude Code slash commands that run the whole development loop — **pick work → branch → implement → PR → merge** — without leaving the terminal. No browser, no second window.

Work at **project** level (one branch, many issues) or **issue** level (one branch per issue). Two variants, same workflow, different issue tracker:

| Variant | Issue tracker | Directory |
|---------|---------------|-----------|
| **Claude + Linear** (primary) | [Linear](https://linear.app) | `claude/` |
| **Claude + Jira** | [Jira](https://www.atlassian.com/software/jira) | `claude-jira/` |

Pick one per project — install `claude/` **or** `claude-jira/`, not both.

## Contents

- [How it works](#how-it-works)
- [Commands at a glance](#commands-at-a-glance)
- [Linear vs Jira](#linear-vs-jira)
- [Installation](#installation) — [Linear](#linear-installation) · [Jira](#jira-installation)
- [Daily workflow](#daily-workflow)
- [Command reference](#command-reference)

---

## How it works

The daily loop, identical in both variants:

```
/standup → /next → implement → /done → /next → repeat
```

Or hand the whole cycle to Claude and walk away:

```
/loop /grind project       # drain a project, one issue per cycle, until empty
/loop /autopilot project   # same, but only issues labelled auto-claude
```

Under the hood: the tracker's hosted MCP server gives Claude structured read access (issues, projects, comments), and the CLI (`linear` / `jira`) handles writes the MCP doesn't — branch creation, status moves, status-filtered listings. `gh` drives GitHub.

---

## Commands at a glance

| Command | What it does |
|---------|--------------|
| **Core loop** | |
| `/next` | Pick the next ready issue and start — project or issue scope |
| `/done` | Ship: quality gate (diff review, tests, code review), PR, CI, confirmed squash-merge |
| `/grind` | One autonomous cycle (pick → implement → ship), no prompts; wrap in `/loop` |
| `/autopilot` | `/grind` restricted to issues labelled `auto-claude` — for an unsupervised instance |
| `/pr` | Open a PR for review without merging |
| **Day-to-day** | |
| `/standup` | Yesterday / today / blocked, from tracker + GitHub |
| `/sync` | Reconcile tracker state with GitHub (merged PRs, stale issues) |
| `/whatchanged` | Management progress report since the last run |
| **Planning** | |
| `/think` | Reason through a problem before any issues exist |
| `/vision` | Strategic 1–3 year direction for a project |
| `/plan` | Draft and create issues with acceptance criteria |
| `/scope` | Audit a project for gaps and unclear issues |
| `/split` | Break a large issue into ordered sub-issues |
| `/estimate` | Bulk t-shirt-size unestimated issues |
| `/triage` | Groom issues missing priority / labels / estimate |
| `/issues` | Browse and filter issues by project |
| **Code health** | |
| `/bugs` | Scan the codebase for bugs → issues |
| `/debt` | Scan for tech debt → issues |
| `/deps` | Audit dependencies (security + outdated) → issues |
| **Review & ship** | |
| `/review` | Review an open PR (diff, CI, approve / comment) |
| `/release` | Generate a changelog and cut a GitHub release |
| **Thinking** | |
| `/diagnose` | Root-cause a bug before touching code |
| `/sit` | Stop, inspect, think — mid-task self-audit |

Full detail per command in the [Command reference](#command-reference).

---

## Linear vs Jira

The command set and workflow are identical; only the tracker calls differ.

| | Claude + Linear | Claude + Jira |
|---|---|---|
| Tracker integration | Linear MCP + `linear` CLI | Atlassian MCP + `jira` CLI |
| Grouping | Projects | Epics |
| Branch (issue mode) | `linear branch ISSUE-1` | `git checkout -b PROJ-1-slug` |
| Branch (project mode) | `<project-slug>-YYYY-MM-DD` | `<epic-slug>-YYYY-MM-DD` |
| Ready status | `Ready for build` | `To Do` |
| Start issue | `linear issue start` | `jira issue move` + `jira issue assign` |
| Close issue | `Closes FIN-X` in PR body (GitHub integration) | `jira issue move "Done"` after merge, or Jira GitHub app |
| Sprints | Cycles + milestones | `jira sprint list/add` |
| Priorities | Urgent / High / Medium / Low | Highest / High / Medium / Low / Lowest |

---

# Installation

## Linear installation

### 1. Get the commands

```bash
git clone https://github.com/gutehall/devloop.git
```

**Per-project:**
```bash
cp -r devloop/claude /path/to/your/project/.claude
```

**Global:**
```bash
cp devloop/claude/commands/* ~/.claude/commands/
cp -r devloop/claude/skills/* ~/.claude/skills/
```

### 2. Linear CLI

```bash
npm install -g @dabble/linear-cli
linear login
```

`linear login` walks you through saving credentials (project or global), pasting an API key, and selecting a team. Config is layered: `~/.linear` (global) loads first, then `./.linear` (project) overrides it; `LINEAR_API_KEY` / `LINEAR_TEAM` env vars are fallbacks.

**Multiple teams:** set `team=TEAMKEY` in each project's `.linear`. Run `linear whoami` to confirm the active team.

### 3. Linear MCP server

```bash
claude mcp add --transport http linear-server https://mcp.linear.app/mcp
```

Then run `/mcp` in a Claude Code session to authenticate.

### 4. GitHub CLI

```bash
brew install gh
gh auth login
```

### 5. Workflow statuses (required)

The commands filter and transition issues by **exact, case-sensitive status name**. Your Linear team must have these — or change the strings in the command files to match yours:

| Status | Used by | Purpose |
|--------|---------|---------|
| `Ready for build` | `/next`, `/grind` | The queue commands pick from. **Only issues here are picked up.** |
| `In Progress` | `/next`, `/grind`, `/done` | Set when implementation starts. |
| `In Review` | GitHub integration | Issue under PR review — set automatically when the PR opens (if configured). Commands no longer set this while reading. |
| `Backlog` | `/grind` | Where `/grind` parks non-code / ambiguous issues so they leave the queue for a human. |
| `Done` | merge | Set by the GitHub integration when the PR merges. |

`/grind` also tags parked issues with a **`needs-human`** label (and `/autopilot` requires an **`auto-claude`** label) — create these, or change the strings in the command files. Verify with `linear issues --status "Ready for build"` — it should return without error.

### 6. Branch protection (recommended)

The commands run quality gates before every push (diff review, local tests, code-review pass), but those gates are prompt-level. Protect `main` so they're enforced server-side too — required CI checks plus one approving review, and enable auto-merge so `/grind` and `/autopilot` can arm PRs instead of merging directly. The exact `gh` commands are in the **github-cli** skill's "Branch protection" section.

### 7. Windows

Same tools work in PowerShell — use `Copy-Item -Recurse` instead of `cp -r`, and `winget install --id GitHub.cli` (or `scoop install gh`). Everything else is identical.

---

## Jira installation

### 1. Get the commands

```bash
git clone https://github.com/gutehall/devloop.git
```

**Per-project:**
```bash
cp -r devloop/claude-jira /path/to/your/project/.claude
```

**Global:**
```bash
cp devloop/claude-jira/commands/* ~/.claude/commands/
cp -r devloop/claude-jira/skills/* ~/.claude/skills/
```

### 2. Jira CLI

```bash
brew install jira-cli
jira init
```

`jira init` prompts for your Jira server URL, email, and API token (generate one at <https://id.atlassian.com/manage-profile/security/api-tokens>), then asks which project and board to default to.

### 3. Atlassian MCP server

```bash
claude mcp add --transport sse atlassian-server https://mcp.atlassian.com/v1/sse
```

Then run `/mcp` in a Claude Code session for the OAuth flow. The MCP gives Claude richer reads (search, comments, cross-project links); the `jira` CLI handles writes (status moves, assignments, branch creation). Both are required.

### 4. GitHub CLI

```bash
brew install gh
gh auth login
```

### 5. GitHub–Jira integration (optional)

Install the Jira GitHub app. Once connected, PRs with `Closes PROJ-42` in the body auto-transition the issue to Done on merge. Without it, run `jira issue move PROJ-42 "Done"` after merge.

### 6. Workflow statuses (required)

The commands filter and transition issues by **exact status name**. Your Jira workflow must have these — or change the strings in the command files:

| Status | Used by | Purpose |
|--------|---------|---------|
| `To Do` | `/next`, `/grind` | The queue commands pick from. **Only issues here are picked up.** |
| `In Progress` | `/next`, `/grind`, `/done` | Set when implementation starts. |
| `In Review` | GitHub app | Issue under PR review — set when the PR opens (if the app transitions it). Commands no longer set this while reading. |
| `Backlog` | `/grind` | Where `/grind` parks non-code / ambiguous issues so they leave the queue for a human. |
| `Done` | merge | Set after merge (GitHub app or `jira issue move`). |

`/grind` tags parked issues `needs-human` (and `/autopilot` requires `auto-claude`). Jira status names are workflow-specific — confirm yours with `jira issue list -s"To Do"` and edit the command files if they differ.

### 7. Branch protection (recommended)

The commands run quality gates before every push (diff review, local tests, code-review pass), but those gates are prompt-level. Protect `main` so they're enforced server-side too — required CI checks plus one approving review, and enable auto-merge so `/grind` and `/autopilot` can arm PRs instead of merging directly. The exact `gh` commands are in the **github-cli** skill's "Branch protection" section.

---

# Daily workflow

### Morning

```
/standup
```
What you finished yesterday, what's in progress, what's blocked — from tracker + GitHub. On Monday the lookback extends to cover the weekend.

```
/next
```
Claude asks **project** or **issue** scope (skip with `/next project` / `/next issue`):

- **Project** — loads a project/epic, picks the highest-priority ready issue, one branch `<slug>-YYYY-MM-DD`, advance with repeated `/next project`
- **Issue** — pick from up to 3 ready issues (or pass an ID), one branch per issue

### Implementation loop

1. Claude reads the issue, explores the code, implements
2. Review what'll be committed: `!git status`, `!git diff`
3. Ship: `/done` (match scope — `/done project` or `/done issue`)
4. Continue: `/next` with the same scope

If CI fails, `/done` stops and reports what broke. Fix, push to the same branch, re-run `/done` — it reuses the existing PR. Use `/pr` instead of `/done` when you want a teammate to review before merge.

### Planning new work

```
/plan "Add retry logic to the analyzer"
```
Claude reads tracker state, asks clarifying questions, drafts issues with acceptance criteria, creates them, and suggests `/next` to start.

---

# Command reference

Conventions: examples use Linear IDs like `FIN-12`. Substitute `PROJ-12` for Jira.

### `/next` — Start the next piece of work

```
/next                      # Ask: project or issue?
/next project              # Project mode (skip scope question)
/next issue                # Issue mode (skip scope question)
/next project "Phase 1"    # Named Linear project / Jira epic
/next issue FIN-12         # Specific issue
```

| Mode | What happens | Branch |
|------|--------------|--------|
| **Project** | Highest-priority ready issue in the project; repeat `/next project` for the next on the same branch | `phase-1-2026-05-22` |
| **Issue** | Pick from up to 3 ready issues (or pass an ID); one branch per issue | `fin-42-add-caching-layer` |

- Only issues in the ready status (`Ready for build` / `To Do`) are picked up — see [Workflow statuses](#5-workflow-statuses-required).
- The issue **stays in that status while Claude reads it**, then moves to `In Progress` when implementation starts.
- Branches automatically from `main`. Set a default project with `linear project open "Phase 1"`.

### `/done` — Ship completed work

```
/done                      # Ask: project or issue?
/done project              # Ship the whole project branch
/done issue                # Ship the current issue branch
/done project "Phase 1"    # Named project (ambiguous branch)
/done issue FIN-12         # Specific issue
```

Match the scope you started `/next` with.

| Mode | PR | Closes |
|------|-----|--------|
| **Issue** | One PR, one issue | `Closes FIN-12` |
| **Project** | One PR for the branch | `Closes FIN-10`, `Closes FIN-12`, … (every issue on the branch) |

- Runs a quality gate before pushing — full diff review (no blind staging), local test/build run, and a code-review-skill pass; Critical/High findings are fixed before anything leaves the machine.
- Pushes, waits for CI, then **asks before merging** (squash + delete branch). No CI checks → merges only with explicit confirmation. Repos with required reviews → arms auto-merge instead.
- The PR's test plan contains the commands actually run and their results — not an empty checklist.
- Project mode collects issues to close from `In Progress` + IDs in the branch's commits; `Closes <ID>` moves each to `Done` on merge, then runs `linear project complete` when none remain open.
- Push diverged → rebases and lists conflicts; never force-pushes without explicit instruction.
- Non-code work (docs, decks, research): closes the issue(s) directly, optionally attaching links.

### `/grind` — Autonomous one-cycle

```
/grind project             # one cycle: highest-priority ready issue in the project
/grind issue               # one cycle: highest-priority ready issue
/loop /grind project       # drain the whole project, one issue per cycle
```

`/grind` fuses `/next` + implement + `/done` into a single command with **no prompts** — built to run under `/loop` unattended. It resolves scope from the argument, branch, or default project, picks the top ready issue, branches from `main`, implements the minimal change, runs the pre-ship gate (diff review, local tests, code-review pass), pushes, opens a PR, waits for CI, and **arms auto-merge** — GitHub completes the merge once branch protection is satisfied. It never merges directly with absent CI.

Where `/next`/`/done` would ask a human, `/grind` emits a **STOP LOOP** signal so the loop ends cleanly instead of hanging:

| Condition | Behavior |
|-----------|----------|
| No ready work left | Stop loop (clean finish) |
| Issue is non-code or ambiguous | Move to Backlog (+`needs-human`), continue loop |
| Needs a product decision | Stop loop, issue left In Progress |
| Tests/build or CI fail | Stop loop — fix and re-run |
| Unfixable Critical/High self-review finding | Stop loop, issue left In Progress |
| Repo has no CI checks | Stop loop, PR left open for manual review |
| PR awaiting required review | Stop loop (clean — auto-merge armed) |
| Rebase conflict | Stop loop, never force-pushes |

- Ships **one PR per issue every cycle** in both scopes (own CI gate, isolated failures) — unlike `/done project`, which batches a branch into one PR.
- Project branch `<slug>-YYYY-MM-DD` is recreated fresh from `main` each cycle, not reused.
- Loop timing is self-paced. Use `/grind` to let Claude drive; use plain `/next`/`/done` to stay in the loop yourself.

### `/autopilot` — Allowlisted autonomous cycle

```
/autopilot project          # one cycle, allowlisted issues in the project only
/autopilot issue            # one cycle, one allowlisted issue per branch
/loop /autopilot project    # drain the allowlisted queue unattended
```

`/grind` with one hard rule: **it only works issues labelled `auto-claude`.** Every queue lookup filters on the label, and before any branch/commit/transition it re-confirms the picked issue carries it — if not, it stops and touches nothing.

- Point an **unattended instance at `/autopilot` only** — never `/grind`/`/next`, which would reach the whole backlog.
- A human opts an issue in by adding the `auto-claude` label; the bot is structurally unable to act on anything else.
- Create the `auto-claude` label first. Empty queue = clean stop, not an error.
- **Never runs a direct merge.** It arms auto-merge after CI passes and lets GitHub (branch protection) or a human complete it; if auto-merge isn't available, the PR stays open and the loop stops.
- Everything else (scope, per-issue PRs, pre-ship gate, STOP-LOOP conditions, non-code skip) matches `/grind`.

### `/pr` — Open a pull request

```
/pr               # PR for the current branch
/pr --draft       # Open as a draft PR
/pr "my title"    # Override the generated title
```

Reviews the full diff, runs local tests/build, runs a code-review pass, then commits, pushes, and creates a PR with `Closes <ID>` and a real test plan. Use when you want a teammate to review before merging. Test failures block the push unless you explicitly ask for a `--draft`.

### `/standup` — Daily standup summary

```
/standup
```

Yesterday / Today (in progress) / Blocked, plus recent commits and merged PRs. Monday lookback extends to 3 days.

### `/sync` — Reconcile tracker with GitHub

```
/sync             # Report drift (read-only)
/sync fix         # Report drift and apply fixes
```

Detects merged PRs with open issues, stale in-progress issues with no branch, and closed PRs. Useful after time off or sprint boundaries.

### `/whatchanged` — Management progress report

```
/whatchanged
```

Reads a checkpoint at `.claude/whatchanged`, pulls issues shipped / started / in flight since then, generates a plain-language report (Delivered / Bug Fixes / In Progress / Coming Up), and updates the checkpoint. First run records the baseline only.

### `/think` — Reason through a problem before planning

```
/think                     # Open-ended exploration
/think "Add X feature"     # Focused reasoning
```

A structured conversation to figure out what to build and why — no tracker reads or writes. Asks grounding questions, explores the space (why now, simplest version, risks, alternatives, out-of-scope), then synthesizes a brief for `/plan`.

### `/vision` — Strategic future planning

```
/vision                    # Open-ended future exploration
/vision "1 year"           # Near-term horizon
/vision "scale"            # A strategic theme
/vision "compete with X"   # Competitive positioning
```

Reads current state (open projects, `product.md`, README, recent git log), then works through forces of change, capability gaps, strategic bets, and assumption stress-tests. Produces a vision document with a hand-off to `/plan`.

### `/plan` — Plan work and create issues

```
/plan                      # Open-ended planning session
/plan "Add retry logic"    # Focused planning
```

Reads tracker state, drafts issues with titles, descriptions, and acceptance criteria, creates them, then suggests `/next`. Issue mode stays one PR per issue; project mode can ship multiple in one PR. L/XL issues get split into sub-issues.

### `/scope` — Audit a project for gaps

```
/scope                  # Audit the default project
/scope "Phase 1"        # Audit a specific project
```

Finds unclear, unestimated, orphaned, oversized, and stale-in-progress issues, plus scope gaps. Never edits anything without confirmation.

### `/split` — Break a large issue into sub-issues

```
/split FIN-12     # Split a specific issue
/split            # Split the current issue (from branch)
```

Proposes 3–5 sub-issues with estimates and dependency ordering, creates them with `--parent` and `--blocked-by`, then offers `/next` for the first.

### `/estimate` — Bulk-estimate issues

```
/estimate                    # Work through all unestimated issues
/estimate FIN-12             # Estimate a specific issue
/estimate --project "P1"     # Scope to a project
```

Per issue: shows context, reads referenced code if needed, suggests a t-shirt size with rationale. Flags L/XL at the end and suggests `/split`.

### `/triage` — Groom the backlog

```
/triage              # Issues missing priority, labels, or estimate
/triage --priority   # Only issues with no priority
/triage --unlabeled  # Only issues with no labels
```

Presents each untriaged issue with suggested priority, labels, estimate, and project. Accept, edit a field, skip, or quit.

### `/issues` — Browse and filter issues

```
/issues                  # Browse active projects
/issues "Phase 1"        # Jump to a named project
```

Lists active projects, then issues with ID, priority, status, title, assignee, estimate. Filter by status/assignee; type an ID for full detail and a `/next` hand-off.

### `/bugs` — Scan the codebase for bugs

```
/bugs              # Full codebase scan
/bugs <path>       # Scan a specific path
```

Reads every source file across seven categories (security, error handling, logic, null access, type safety, resources, concurrency). Creates one issue per bug in priority order, each with location, impact, and suggested fix, tagged `bug`.

### `/debt` — Scan for tech debt

```
/debt              # Full codebase scan
/debt <path>       # Scan a specific path
```

Looks for missing tests, complex functions, dead code, TODO/FIXME, hardcoded config, duplication, weak typing, outdated patterns. One issue per finding, tagged `tech-debt`. The non-bug counterpart to `/bugs`.

### `/deps` — Audit dependencies

```
/deps              # Full audit (security + outdated)
/deps --security   # Vulnerabilities only
/deps --outdated   # Outdated packages only
```

Detects all package managers (npm, yarn, pnpm, pip, cargo, go, bundler), maps CVE severity to priority, creates one issue per finding with the exact upgrade command.

### `/review` — Review an open pull request

```
/review           # List open PRs and pick one
/review 42        # Review PR #42 directly
```

Shows description, CI status, full diff; summarizes changes and issues spotted; offers approve / request changes / comment. Skips approve when the PR is yours.

### `/release` — Changelog + GitHub release

```
/release            # Auto-detect version bump
/release patch      # Force patch / minor / major
```

Finds the last tag, gathers merged PRs and commits since, categorizes (Breaking / Features / Fixes / Performance / Docs / Chores), determines semver, tags, and runs `gh release create`. Optionally prepends to CHANGELOG.md.

### `/diagnose` — Root-cause before touching code

```
/diagnose "login fails with 401 after token refresh"
/diagnose
```

Reads the full error, generates ≥3 mechanical hypotheses with evidence, runs the fastest diagnostic to distinguish the top two, then fixes the confirmed root cause with the minimum change. No code until a hypothesis is confirmed.

### `/sit` — Stop, Inspect, Think

```
/sit
```

Pauses, inspects what's been done, judges whether the approach is still right, and decides: Continue / Correct course / Ask / Stop and report. Triggers automatically after many unconfirmed tool calls, on something unexpected, before a destructive action, or when scope has grown.
