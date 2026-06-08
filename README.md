# DevLoop

A set of Claude Code slash commands that implement the full development loop — pick work → branch → implement → PR → merge — entirely inside Claude Code, with no browser or second terminal window. Work at **project** level (one branch, many issues) or **issue** level (one branch per issue).

Two variants, same workflow, different issue tracker:

| Variant | Issue tracker | Directory |
|---------|--------------|-----------|
| **Claude + Linear** (primary) | [Linear](https://linear.app) | `claude/` |
| **Claude + Jira** | [Jira](https://www.atlassian.com/software/jira) | `claude-jira/` |

---

## Claude + Linear

The daily-driver variant. Linear MCP + the `linear` CLI for issue tracking, `gh` for GitHub.

**Daily loop:**

```
/standup → /next → implement → /done → /next → repeat
```

Or run it unattended: `/loop /grind project` repeats the whole cycle until the project is drained.

**What you get:**

- `/standup` — yesterday / today / blocked, from Linear + GitHub
- `/next` — project mode (whole Linear project on one branch) or issue mode (one branch per issue)
- `/done` — ship project (one PR, multiple issues) or ship a single issue; CI, merge, pull main
- `/grind` — one autonomous cycle (pick → implement → ship), no prompts; wrap in `/loop` to drain a whole project unattended
- `/think`, `/vision` — reason through a problem or set strategic direction before planning
- `/plan` — create Linear issues inline via MCP, with acceptance criteria
- `/pr` — open a PR for review without merging
- `/issues`, `/triage`, `/estimate`, `/split`, `/scope` — backlog browsing and grooming
- `/bugs`, `/debt`, `/deps` — scan the codebase and create Linear issues
- `/review`, `/sync`, `/whatchanged`, `/release` — review and ship cycles
- `/diagnose`, `/sit` — structured thinking before/during work

How it talks to Linear: Linear's hosted MCP server (`mcp.linear.app`) gives Claude structured access to issues, projects, cycles, comments, and documents. The `linear` CLI handles the things the MCP doesn't cover (branch creation, quick `--unblocked` listings).

[Skip to installation →](#claude--linear-installation)

---

## Claude + Jira

Same loop, different tracker. The Atlassian MCP server + the `jira` CLI for issue tracking, `gh` for GitHub.

**Daily loop:**

```
/standup → /next → implement → /done → /next → repeat
```

Or run it unattended: `/loop /grind project` repeats the whole cycle until the project is drained.

The command set mirrors Claude + Linear — `/standup`, `/next`, `/done`, `/grind`, `/think`, `/vision`, `/plan`, `/pr`, `/issues`, `/triage`, `/estimate`, `/split`, `/scope`, `/bugs`, `/debt`, `/deps`, `/review`, `/sync`, `/whatchanged`, `/release`, `/diagnose`, `/sit` — but each command is rewritten to call `jira` instead of `linear`/MCP.

**Key behavior differences vs Linear:**

| | Claude + Linear | Claude + Jira |
|---|---|---|
| Branch creation (issue mode) | `linear branch ISSUE-1` | `git checkout -b PROJ-1-slug` |
| Branch creation (project mode) | `<project-slug>-YYYY-MM-DD` | `<epic-slug>-YYYY-MM-DD` |
| `/next` / `/done` scope | Project or single issue (asked each time) | Epic (project) or single issue |
| Start issue | `linear issue start` | `jira issue move` + `jira issue assign` |
| Close issue | Auto-closed by `Closes FIN-X` in PR body (Linear's GitHub integration) | `jira issue move "Done"` after merge, or via the Jira GitHub app |
| Sprints | Cycles + milestones | `jira sprint list/add` |
| Grouping | Projects | Epics |
| Priorities | Urgent / High / Medium / Low | Highest / High / Medium / Low / Lowest |
| Tracker integration | Linear MCP + CLI | Atlassian MCP + Jira CLI |

[Skip to installation →](#claude--jira-installation)

---

# Installation

Pick one variant per project — install either `claude/` (Linear) or `claude-jira/` (Jira), not both.

## Claude + Linear installation

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

`linear login` will:
1. Ask where to save credentials (project or global)
2. Open Linear API settings in your browser
3. Prompt you to paste your API key
4. Let you select your team
5. Save config

Config is layered: `~/.linear` (global) is loaded first, then `./.linear` (project-level) overrides it. Env vars (`LINEAR_API_KEY`, `LINEAR_TEAM`) are used as fallbacks.

**Multiple teams:** if you work across several Linear teams, set `team=TEAMKEY` in each project's `.linear` file. Run `linear whoami` to confirm which team is active.

### 3. Linear MCP server

```bash
claude mcp add --transport http linear-server https://mcp.linear.app/mcp
```

Then run `/mcp` once you've opened a Claude Code session to go through the authentication flow.

### 4. GitHub CLI

```bash
brew install gh
gh auth login
```

### 5. Windows variants

All the same tools work on Windows in PowerShell — use `Copy-Item -Recurse` instead of `cp -r`, and `winget install --id GitHub.cli` (or `scoop install gh`) for the GitHub CLI. Everything else is identical.

---

## Claude + Jira installation

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

Then run `/mcp` once you've opened a Claude Code session to go through the OAuth flow — Atlassian opens in your browser, you grant access to the Jira site, and the token is stored.

The Atlassian MCP gives Claude richer access for reads (search issues, fetch comments, follow links across projects/boards) than the CLI alone. The slash commands in `claude-jira/` still use the `jira` CLI for writes (status moves, assignments, branch creation), so both are required.

### 4. GitHub CLI

```bash
brew install gh
gh auth login
```

### 5. GitHub-Jira integration (optional)

Install the Jira GitHub app in your Jira workspace. Once connected, PRs that include `Closes PROJ-42` in the body auto-transition the issue to Done on merge. Without it, run `jira issue move PROJ-42 "Done"` manually after merge.

---

# Daily Workflow

### Morning

```
/standup
```
Shows what you completed yesterday, what's in progress, and any blocked issues. Pulls from both the tracker and GitHub.

```
/next
```
Claude asks whether to work at **project** or **issue** level (same style as the `main` vs `develop` branch question). Skip with `/next project` or `/next issue`.

- **Project** — loads a Linear project (or Jira epic), picks the highest-priority ready issue, uses one branch named `<slug>-YYYY-MM-DD`, advances issue-by-issue with repeated `/next project`
- **Issue** — shows up to 3 unblocked issues to choose from, one branch per issue (`linear branch` / `PROJ-12-slug`)

### Implementation loop

1. Claude reads the issue, explores the codebase, and implements
2. Review what will be committed: `!git status`, `!git diff`
3. Ship it: `/done` (match scope: `/done project` or `/done issue`)
4. Continue: `/next` with the same scope

**Issue mode:** `/done issue` stages, commits, pushes, opens one PR with `Closes FIN-X`, waits for CI, merges, deletes the branch, returns to main.

**Project mode:** `/done project` does the same for the whole project branch — one PR with a `Closes FIN-X` line per issue worked on, then `linear project complete` when the backlog is clear.

If CI fails: `/done` stops and tells you what failed. Fix the code, push to the same branch, run `/done` again with the same scope — it picks up the existing PR.

Use `/pr` instead of `/done` when you want a teammate to review before merging.

### Planning new work

```
/plan "Add retry logic to the analyzer"
```
Claude reads current tracker state, asks clarifying questions if needed, drafts issues with descriptions and acceptance criteria, creates them directly, then suggests `/next project` or `/next issue <ID>` to start immediately.

### End of day (optional)

```
/standup
```
Summary of what shipped, what's still open, any PRs pending review.

---

# Commands Reference

Conventions: examples use Linear issue IDs like `FIN-12`. Substitute `PROJ-12` for the Jira variant.

### `/next` — Start the next piece of work

```
/next                      # Ask: project or issue?
/next project              # Project mode (skip scope question)
/next issue                # Issue mode (skip scope question)
/next project "Phase 1"    # Named Linear project / Jira epic
/next issue FIN-12         # Specific issue
```

**Scope question** (unless you pass `project` or `issue`, or the current branch already implies mode):

> Work at **project** level or on a **single issue**?

| Mode | What happens | Branch |
|------|----------------|--------|
| **Project** | Highest-priority unblocked issue in the project; repeat `/next project` for the next issue on the same branch | `phase-1-2026-05-22` |
| **Issue** | Pick from up to 3 unblocked issues (or pass an ID); one issue per branch | `fin-42-add-caching-layer` |

Then asks `main` or `develop` (code work), marks the issue In Progress, reads it, and starts implementation. Set a default project with `linear project open "Phase 1"` for faster project mode.

### `/think` — Reason through a problem before planning

```
/think                        # Open-ended exploration
/think "Add X feature"        # Focused reasoning about a specific idea
```

A structured conversation to think through what you actually want to build and why — before creating any issues. No tracker reads or writes. Asks grounding questions (what's the problem, who's affected, what does good look like), explores the space (why now, simplest version, what could go wrong, alternatives, what's out of scope), then synthesizes a brief you can hand straight to `/plan`.

### `/vision` — Strategic future planning for a project

```
/vision                          # Open-ended future exploration
/vision "1 year"                 # Focus on the near-term horizon
/vision "scale"                  # Explore a strategic theme
/vision "compete with X"         # Explore competitive positioning
```

Deep, honest conversation about where the project is heading on 1 and 3 year horizons. Reads current state (open projects, `product.md`, README, recent git log), then works through forces of change, capability gaps, strategic bets, and assumption stress-tests. Produces a structured vision document with a "next planning horizon" section that hands off to `/plan`. No tracker writes during the session.

### `/plan` — Plan work and create issues

```
/plan                        # Open-ended planning session
/plan "Add retry logic"      # Focused planning for a specific area
```

Reads current tracker state, drafts issues with titles, descriptions, and acceptance criteria, creates them directly, then suggests `/next project` or `/next issue <ID>`. In issue mode, work stays one PR per issue; in project mode, multiple issues can ship in one PR. L/XL issues get broken into sub-issues.

### `/done` — Ship completed work

```
/done                      # Ask: project or issue?
/done project              # Ship the whole project branch
/done issue                # Ship the current issue branch
/done project "Phase 1"    # Named project (ambiguous branch)
/done issue FIN-12         # Specific issue
```

**Scope question** matches `/next` — use the same mode you started with.

| Mode | PR | Closes |
|------|-----|--------|
| **Issue** | One PR, one issue | `Closes FIN-12` |
| **Project** | One PR for the branch | `Closes FIN-10`, `Closes FIN-12`, … (every issue on the branch) |

Stages, commits, pushes, waits for CI, merges with squash, deletes the branch, returns to main. Project mode runs `linear project complete` when no open issues remain. If push fails due to divergence, rebases and lists conflicts — never force-pushes without explicit instruction.

For non-code work (documents, decks, research): closes issue(s) in Linear directly, optionally attaching links.

### `/grind` — Autonomous one-cycle: pick → implement → ship

```
/grind project             # one cycle: highest-priority issue in the project
/grind issue               # one cycle: highest-priority unblocked issue
/loop /grind project       # drain the whole project, one issue per cycle
/loop /grind issue         # repeat, one branch per issue
```

`/grind` is `/next` + implement + `/done` fused into a single command with **no interactive questions** — built so it can run under `/loop` unattended. It resolves scope from the argument, the current branch, or the default project (never prompts), picks the single highest-priority unblocked issue, branches from `main`, implements the minimal change, then pushes, opens the PR, waits for CI, and merges.

Where a normal `/next`/`/done` would ask a human, `/grind` instead emits a clear **STOP LOOP** signal so the loop ends cleanly rather than hanging:

| Condition | Behavior |
|-----------|----------|
| No unblocked work left | Stop loop (clean finish) |
| Issue is non-code or ambiguous | Skip it, continue loop |
| Needs a product decision | Stop loop, issue left In Progress |
| Tests/build or CI fail | Stop loop — fix and re-run |
| Rebase conflict | Stop loop, never force-pushes |

Project mode reuses one day-branch (`<slug>-YYYY-MM-DD`) across cycles; issue mode is one branch per issue. Loop timing is self-paced — work duration varies, so there is nothing to poll on a clock. Use plain `/next`/`/done` when you want to stay in the loop yourself; use `/grind` when you want Claude to drive the whole cycle.

### `/standup` — Daily standup summary

```
/standup
```

Yesterday / Today (in progress) / Blocked, plus recent commits and merged PRs. On Monday, the lookback extends to 3 days to cover the weekend.

### `/pr` — Open a pull request for current work

```
/pr               # PR for the current branch
/pr --draft       # Open as a draft PR
/pr "my title"    # Override the generated title
```

Stages, commits, pushes, creates a PR with `Closes <ID>`. Use this when you want a teammate to review before merging.

### `/estimate` — Bulk-estimate unestimated issues

```
/estimate                    # Work through all unestimated issues
/estimate FIN-12             # Estimate a specific issue
/estimate --project "P1"     # Scope to a project
```

For each issue: shows context, reads referenced code if needed, suggests a t-shirt size with a rationale. Flags L/XL issues at the end and suggests `/split`.

### `/scope` — Audit a project for gaps

```
/scope                  # Audit the current default project
/scope "Phase 1"        # Audit a specific project
```

Looks for unclear issues, unestimated issues, orphaned issues, scope gaps, oversized issues, and stale in-progress. Never creates or edits anything without confirmation.

### `/split` — Break a large issue into sub-issues

```
/split FIN-12     # Split a specific issue
/split            # Split the current issue (detected from branch)
```

Proposes 3–5 sub-issues with estimates and dependency ordering, creates them with `--parent` and `--blocked-by` set correctly, then offers `/next project` or `/next issue` for the first one.

### `/issues` — Browse and filter issues by project

```
/issues                  # Browse active projects
/issues "Phase 1"        # Jump directly to a named project
```

Lists active projects, then issues with ID, priority, status, title, assignee, estimate. Filters by status and assignee. Typing an issue ID shows full details and offers `/next issue <ID>` or `/next project`.

### `/triage` — Groom the backlog interactively

```
/triage              # All issues missing priority, labels, or estimate
/triage --priority   # Only issues with no priority set
/triage --unlabeled  # Only issues with no labels
```

Presents each untriaged issue with suggested priority, labels, estimate, and project. Accept with Enter, edit individual fields, skip, or quit.

### `/bugs` — Scan the codebase for bugs

```
/bugs              # Full codebase scan
/bugs <path>       # Scan a specific directory or file
```

Reads every source file. Scans seven categories: security, error handling, logic errors, null/undefined access, type safety, resource management, concurrency. Creates one issue per bug in priority order (Urgent → Low), each with file location, impact, and suggested fix, tagged `bug`.

### `/debt` — Scan the codebase for tech debt

```
/debt              # Full codebase scan
/debt <path>       # Scan a specific path
```

Looks for missing tests, overly complex functions, dead code, TODO/FIXME, hardcoded config, duplicated logic, missing type safety, outdated patterns. Creates one issue per finding tagged `tech-debt`. The non-bug counterpart to `/bugs`.

### `/deps` — Audit dependencies

```
/deps              # Full audit (security + outdated)
/deps --security   # Security vulnerabilities only
/deps --outdated   # Outdated packages only
```

Detects all package managers (npm, yarn, pnpm, pip, cargo, go modules, bundler), maps CVE severity to tracker priority, creates one issue per finding with the exact upgrade command.

### `/review` — Review open pull requests

```
/review           # List open PRs and pick one
/review 42        # Review PR #42 directly
```

Shows PR description, CI status, full diff. Summarizes changes and issues spotted. Offers approve / request changes / comment. Skips the approve option when the PR is yours.

### `/sync` — Sync tracker state with GitHub

```
/sync             # Report drift (read-only)
/sync fix         # Report drift and apply fixes
```

Detects merged PRs with open issues, stale in-progress issues with no branch, closed PRs. Useful after time off or sprint boundaries.

### `/whatchanged` — Management progress report

```
/whatchanged
```

Reads a checkpoint at `.claude/whatchanged`, pulls issues that shipped/started/are in flight since then, generates a plain-language report (Delivered / Bug Fixes / In Progress / Coming Up), then updates the checkpoint. First run records the baseline; no report is produced.

### `/release` — Generate changelog and create a GitHub release

```
/release            # Auto-detect version bump from change types
/release patch      # Force patch bump
/release minor      # Force minor bump
/release major      # Force major bump
```

Finds the last git tag, gathers merged PRs and commits since, categorizes (Breaking / Features / Fixes / Performance / Docs / Chores), determines semver, creates the tag, runs `gh release create`. Optionally prepends to CHANGELOG.md.

### `/diagnose` — Root-cause a bug before touching any code

```
/diagnose "login fails with 401 after token refresh"
/diagnose
```

Reads the full error, generates at least 3 mechanical hypotheses with evidence for/against, runs the fastest diagnostic that distinguishes the top two, fixes the confirmed root cause with the minimum change. No code is written until a hypothesis is confirmed.

### `/sit` — Stop, Inspect, Think

```
/sit
```

Stops forward momentum, inspects what's been done, thinks through whether the approach is still right, decides Continue / Correct course / Ask / Stop and report. Trigger automatically after many tool calls without confirmation, when something unexpected happens, before a destructive action, or when scope has grown beyond what was asked.

