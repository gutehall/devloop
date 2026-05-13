# DevLoop

This repo contains two workflow variants — both powered by Claude Code, differing only by issue tracker:

| Variant | Issue tracker | Directory |
|---------|--------------|-----------|
| **Claude + Linear** (primary) | [Linear](https://linear.app) | `claude/` |
| **Claude + Jira** | [Jira](https://www.atlassian.com/software/jira) | `claude-jira/` |

Both implement the same loop: pick issue → branch → implement → PR → merge → repeat.

See [Claude + Jira setup](#claude--jira) for the Jira variant.

---

This documents the development loop for Claude Code + Linear: [Linear](https://linear.app) for issue tracking, [Claude Code](https://claude.ai/code) for implementation, and a set of slash commands that tie them together.

Everything runs inside [Claude Code](https://claude.ai/code) — no browser, no second terminal window.

---

## How Commands Work in Claude Code

There are two types of commands:

**Slash commands** — typed directly in the Claude Code chat. Claude executes them.
```
/next
/done
/plan "Add retry logic"
/standup
```

**Shell commands** — prefixed with `!` to run in the current terminal session without leaving Claude Code.
```
!gh pr checks
!gh pr merge --squash --delete-branch
!git log --oneline
```

---

## Installation

### 1. Get the commands

Clone this repo:

```bash
git clone https://github.com/gutehall/devloop.git
```

The slash commands live in `claude/commands/` and skills in `claude/skills/`. Two options:

**Per-project** — copy `claude` into your project root:
```bash
cp -r devloop/claude /path/to/your/project/.claude
```

**Global** — available in all projects:
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

**Multiple teams:** if you work across several Linear teams, set `team=TEAMKEY` in each project's `.linear` file. The project-level config overrides the global one, so each repo automatically targets the right team. Run `linear whoami` to confirm which team is active.

### 3. Linear MCP Server

The slash commands (`/plan`, `/standup`) talk to Linear via MCP. Configure the Linear MCP server in Claude Code settings so it has access to your Linear workspace.

```bash
claude mcp add --transport http linear-server https://mcp.linear.app/mcp
```

Then run `/mcp` once you've opened a Claude Code session to go through the authentication flow.

### 4. GitHub CLI

`/done` creates PRs via `gh`. Install and authenticate:

```bash
brew install gh
gh auth login
```

### 5. gh aliases

The workflow uses short aliases for common PR commands. Set them up once:

```bash
gh alias set prc 'pr checks'
gh alias set prv 'pr view'
gh alias set prd 'pr diff'
gh alias set prl 'pr list'
gh alias set prm 'pr merge --squash --delete-branch'
gh alias set co 'pr checkout'
```

Verify with `gh alias list`.

---

## Windows Installation

All the same tools work on Windows. Use PowerShell unless noted otherwise.

### 1. Get the commands

```powershell
git clone https://github.com/gutehall/devloop.git
```

**Per-project** — copy into your project:
```powershell
Copy-Item -Recurse devloop\claude .claude
```

**Global** — available in all projects:
```powershell
Copy-Item devloop\claude\commands\* $env:USERPROFILE\.claude\commands\
Copy-Item -Recurse devloop\claude\skills\* $env:USERPROFILE\.claude\skills\
```

### 2. Linear CLI

```powershell
npm install -g @dabble/linear-cli
linear login
```

### 3. Linear MCP Server

```powershell
claude mcp add --transport http linear-server https://mcp.linear.app/mcp
```

### 4. GitHub CLI

Install via winget:
```powershell
winget install --id GitHub.cli
gh auth login
```

Or via Scoop: `scoop install gh`

### 5. gh aliases

```powershell
gh alias set prc 'pr checks'
gh alias set prv 'pr view'
gh alias set prd 'pr diff'
gh alias set prl 'pr list'
gh alias set prm 'pr merge --squash --delete-branch'
gh alias set co 'pr checkout'
```

---

## Daily Loop

```
/standup → /next → implement → /done → /next → repeat
```

| Command | What it does |
|---------|-------------|
| `/start` | Set up a known issue without immediately implementing |
| `/pr` | Open a PR for early review without closing the issue |
| `/blocked` | Mark an issue blocked and keep moving |
| `/work` | Plan and start immediate work without creating a Linear ticket first |
| `/estimate` | Bulk-size unestimated issues |
| `/scope` | Audit a project for gaps before a sprint |
| `/split` | Decompose an L/XL issue into sub-issues |
| `/issues` | Browse and filter issues by project |
| `/review` | Review a teammate's PR |
| `/sync` | Clean up stale issues after a sprint or time off |
| `/bugs` | Scan the codebase and push bug findings into Linear |
| `/debt` | Scan the codebase and push tech debt findings into Linear |
| `/deps` | Audit dependencies |
| `/triage` | Groom the backlog |
| `/retro` | Sprint retrospective |
| `/whatchanged` | Generate a management progress report since the last run |
| `/release` | Cut a release |
| `/onboard` | Orient in a new codebase |
| `/diagnose` | Systematically root-cause a bug before touching any code |
| `/sit` | Force a mid-task self-audit when something feels off |

---

## Daily Workflow — Step by Step

### Morning

**1. Get oriented**

Type in Claude Code:
```
/standup
```
Shows what you completed yesterday, what's in progress, and any blocked issues. Pulls from both Linear and GitHub.

**2. Pick the next issue**

```
/next
```
Claude shows up to 3 unblocked issues (yours first). Select one — it gets marked In Progress in Linear, a git branch is created, and Claude starts reading the issue and relevant code.

---

### The implementation loop — repeat until done for the day

**3. Implement**

Claude reads the issue, explores the codebase, and implements. You review and guide as needed. When the work is done, move to the next step.

**4. Review what will be committed**

```
!git status
!git diff
```

**5. Ship it**

```
/done
```

`/done` handles the full ship cycle:
1. Stages and commits any uncommitted changes
2. Pushes the branch and creates a PR with `Closes FIN-X` in the body
3. Waits for CI (`gh pr checks --watch`) — stops and reports if anything fails
4. Merges with squash and deletes the branch
5. Checks out main and pulls
6. Shows `git log --oneline` to confirm the merge landed

On merge, `Closes FIN-X` automatically moves the Linear issue to Done. No manual step needed.

If CI fails, `/done` stops and tells you what failed. Fix the code, push to the same branch, and run `/done` again — it picks up from the PR that already exists.

**6. Pick the next issue**

```
/next
```

Repeat from step 3.

**Use `/pr` instead when you want review before merging**

`/done` merges immediately after CI. If you want a teammate to review first, use `/pr` to open the PR without merging — then merge manually once approved.

---

### When you need to plan new work

```
/plan "Add retry logic to the analyzer"
```

Claude reads current Linear state, asks clarifying questions if needed, drafts issues with descriptions and acceptance criteria, creates them directly in Linear, then suggests `/next <ID>` to start immediately.

---

### End of day (optional)

```
/standup
```
Quick summary of what shipped, what's still open, any PRs pending review.

---

## Commands Reference

### `/next` — Pick the next issue and start working

```
/next           # Show unblocked issues to choose from
/next FIN-12    # Jump straight to a specific issue
```

What it does:
1. Runs `linear issues --unblocked` to find issues ready to work on
2. Presents up to 3 options (your assigned issues sorted first), plus a "Product planning" option
3. On selection: marks the issue In Progress, creates a git branch, reads the issue, explores relevant code, and begins implementation

The branch name is derived from the issue ID and title: `fin-42-add-caching-layer`.

`/next` creates the branch off the current HEAD. Make sure you're on the main branch (or your team's trunk) before running it — `!git checkout main && git pull` first.

---

### `/plan` — Plan work and create Linear issues

```
/plan                        # Open-ended planning session
/plan "Add retry logic"      # Focused planning for a specific area
```

What it does:
1. Reads current Linear state (open issues, projects, teams) via MCP
2. If the input is vague, asks: what needs to happen, who is it for, how do you know it's done
3. Drafts issues with titles, descriptions, and acceptance criteria
4. Creates them directly in Linear via MCP — no clipboard, no copy-paste
5. Prints each created issue ID and title, then suggests `/next <ID>` for the top priority

Issues are kept small (one PR each). L/XL issues get broken into sub-issues.

---

### `/done` — Ship the current issue end-to-end

```
/done           # Complete the current issue (detected from branch name)
/done FIN-12    # Complete a specific issue
```

What it does:
1. Detects the issue ID from the current branch name
2. Shows a summary: commits and files changed
3. Stages and commits any uncommitted changes
4. Pushes the branch to origin
5. Creates a PR titled `FIN-12: Issue title` with `Closes FIN-12` in the body — prints the URL
6. Waits for CI (`gh pr checks --watch`) — stops if anything fails
7. Merges with squash and deletes the remote branch
8. Checks out the base branch and pulls
9. Shows `git log --oneline` to confirm the merge landed

`Closes FIN-12` triggers Linear's GitHub integration — the issue moves to Done on merge automatically.

If CI fails: `/done` stops and tells you what failed. Fix, push to the same branch, run `/done` again — it reuses the existing PR.

For non-code work (documents, decks, research): closes the issue directly in Linear, optionally attaching a link first.

If push fails due to a diverged branch, `/done` will rebase and list any conflicts for you to resolve — it will not force-push without your explicit instruction.

Use `/pr` instead when you want a teammate to review before merging.

---

### `/standup` — Daily standup summary

```
/standup
```

What it does:
1. Fetches via MCP: issues completed recently, issues currently in progress, blocked issues
2. Fetches recent git commits and open/merged PRs via `gh`
3. Presents a summary:

```
Yesterday:
✓ FIN-X: Issue title

Today (in progress):
→ FIN-X: Issue title

Blocked:
⊘ FIN-X: Issue title

Recent commits:
- repo: N commits
- PR #X merged: title
```

On **Monday**, the lookback window extends to 3 days to cover the weekend.

---

### `/review` — Review open pull requests

```
/review           # List open PRs and pick one to review
/review 42        # Review PR #42 directly
```

What it does:
1. Lists open PRs with author, branch, and age
2. On selection: shows the PR description, CI status, and full diff
3. Summarizes what changed and any issues spotted
4. Offers to approve, request changes, or comment

Note: GitHub does not allow approving your own PR — `/review` skips the approve option when the PR is yours.

---

### `/bugs` — Scan the codebase for bugs and create Linear issues

```
/bugs              # Full codebase scan
/bugs <path>       # Scan a specific directory or file
```

What it does:
1. Reads your Linear team and labels via MCP — creates a "bug" label if one doesn't exist
2. Explores the full directory tree and reads every source file (no sampling)
3. Scans for seven categories: security, error handling, logic errors, null/undefined access, type safety, resource management, concurrency
4. Ranks all findings by severity before creating anything
5. Creates one Linear issue per bug in priority order: Urgent → High → Medium → Low

Each issue includes: a `[Category] Title`, structured description with exact file location, impact, and suggested fix, priority 1–4, and the "bug" label.

Creating in priority order ensures your backlog sorts correctly without manual reordering.

---

### `/debt` — Scan the codebase for tech debt and create Linear issues

```
/debt              # Full codebase scan
/debt <path>       # Scan a specific path
```

What it does:
1. Creates a "tech-debt" label in Linear if one doesn't exist
2. Reads every source file looking for: missing tests, overly complex functions, dead code, TODO/FIXME comments, hardcoded config, duplicated logic, missing type safety, and outdated patterns
3. Ranks all findings by impact before creating anything
4. Creates one Linear issue per finding in priority order, each with file location, impact, and a concrete fix

The non-bug counterpart to `/bugs` — use both together for a full code health audit.

---

### `/deps` — Audit dependencies for vulnerabilities and outdated packages

```
/deps              # Full audit (security + outdated)
/deps --security   # Security vulnerabilities only
/deps --outdated   # Outdated packages only
```

What it does:
1. Detects all package managers in the repo (npm, yarn, pnpm, pip, cargo, go modules, bundler)
2. Runs the appropriate security audit and outdated-check commands for each
3. Maps CVE severity to Linear priority: Critical → Urgent, High → High, Moderate → Medium
4. Creates one Linear issue per finding ordered by severity, each with the exact upgrade command

---

### `/triage` — Groom the backlog interactively

```
/triage              # All issues missing priority, labels, or estimate
/triage --priority   # Only issues with no priority set
/triage --unlabeled  # Only issues with no labels
```

What it does:
1. Fetches all untriaged Linear issues (no priority, no labels, or no estimate)
2. Presents each issue one at a time with suggested priority, labels, estimate, and project
3. Accept with Enter, edit individual fields, skip, or quit — applies confirmed values via MCP immediately
4. Reports how many issues were triaged at the end

---

### `/issues` — Browse and filter issues by project

```
/issues                  # Browse active projects, then issues in the selected project
/issues "Phase 1"        # Jump directly to a named project
```

What it does:
1. Lists active projects (Backlog, Planned, In Progress) as a numbered menu
2. On selection, shows all non-completed issues in a table with ID, priority, status, title, assignee, and estimate
3. Offers filter options: by status (Ready / In Progress / Backlog / All) and by assignee (Mine / Unassigned / All)
4. On typing an issue ID, shows full details and offers `/next <ID>` to start work

---

### `/retro` — Sprint retrospective

```
/retro              # Last 2 weeks
/retro 1w           # Last week
/retro 3w           # Last 3 weeks
/retro 2024-01-01   # Since a specific date
```

What it does:
1. Detects the most recently completed Linear cycle, or falls back to the specified time window
2. Pulls shipped issues, slipped issues (started but not closed), blocked issues, and merged/open PRs
3. Presents a structured retrospective: Shipped / Slipped / Never started / Blocked / Velocity
4. Optionally creates Linear issues for action items (recurring blockers, scope creep patterns, missing tests, etc.)

---

### `/release` — Generate a changelog and create a GitHub release

```
/release            # Auto-detect version bump from change types
/release patch      # Force patch bump
/release minor      # Force minor bump
/release major      # Force major bump
```

What it does:
1. Finds the last git tag; uses the first commit as baseline if no tags exist
2. Gathers all merged PRs and commits since that tag
3. Categorizes changes: Breaking / Features / Fixes / Performance / Docs / Chores
4. Determines the next semver version (or uses the argument), drafts the changelog, and asks for confirmation
5. On confirm: creates the tag, pushes it, and runs `gh release create` with the generated notes
6. Optionally prepends to CHANGELOG.md

---

### `/onboard` — Orient in an unfamiliar codebase

```
/onboard            # Print codebase orientation to chat
/onboard --save     # Same, and write ONBOARDING.md to project root
```

What it does:
1. Discovers the stack: language, framework, package manager, test setup
2. Maps top-level directories and identifies all entry points
3. Reads representative source files to extract patterns: imports, error handling, config, data layer
4. Surfaces gotchas — non-obvious setup steps, known quirks, day-one surprises
5. Outputs a structured orientation document (stack, directory map, patterns, key abstractions, gotchas, quick start)

---

### `/diagnose` — Root-cause a bug before touching any code

```
/diagnose "login fails with 401 after token refresh"
/diagnose                 # Describe the problem interactively
```

What it does:
1. Reads the full error — origin vs surface, recent changes, environment details
2. Generates at least 3 mechanical hypotheses ranked by likelihood, each with evidence for and against
3. Designs and runs the fastest diagnostic that distinguishes the top two candidates
4. Fixes the confirmed root cause with the minimum change required
5. Explains what was wrong, why, and what the fix does

Use this instead of guessing and editing. No code is written until a hypothesis is confirmed.

---

### `/sit` — Stop, Inspect, Think — structured mid-task self-audit

```
/sit
```

What it does:
1. **Stops** all forward momentum — no more tool calls, no next line of code
2. **Inspects** what has been done: restates the original goal, lists concrete actions taken, surfaces assumptions made
3. **Thinks** through whether the approach is still right: scope creep, complexity growth, upcoming risky steps
4. **Decides** one of four outcomes: Continue / Correct course / Ask the user / Stop and report

Output is structured and honest — not a recap, not reassurance. The test is whether the output reads like genuine reflection.

Trigger automatically: after 10+ tool calls without confirmation, when something unexpected happens, before a destructive action, or when scope has grown beyond what was asked.

---

### `/sync` — Sync Linear with GitHub state

```
/sync             # Report drift between Linear and GitHub (read-only)
/sync fix         # Report drift and apply fixes
```

What it does:
1. Fetches in-progress Linear issues and checks the state of their branches and PRs
2. Detects: merged PRs with open issues, stale in-progress issues with no branch, closed PRs
3. Reports a table of findings
4. With `fix`: closes stale issues in Linear (confirms before each change)

Useful after returning from time off or at sprint boundaries when the board has drifted.

---

### `/whatchanged` — Management progress report since the last run

```
/whatchanged
```

What it does:
1. Reads a checkpoint file (`.claude/whatchanged`) written by the previous run — stores a timestamp and commit SHA
2. Pulls Linear issues that shipped, started, or are in flight since that checkpoint, plus merged PRs as supporting context
3. Generates a plain-language report for management: Delivered / Bug Fixes / In Progress / Coming Up — no commit SHAs, no file paths, no branch names
4. Offers to reformat the report for Slack or email
5. Updates the checkpoint so the next run covers only the new period

On the **first run**, no report is produced — the command records the current state as the baseline. Run it again after work has shipped to generate the first report.

---

### `/start` — Set up a known issue without implementing

```
/start            # Pick from unblocked issues
/start FIN-12     # Start a specific issue
```

What it does:
1. Assigns the issue to you and marks it In Progress in Linear
2. Creates and checks out a git branch from the issue ID
3. Shows the full issue with context

Use this when you already know what to work on and just need the context loaded. Unlike `/next`, it doesn't start implementing — it hands control back to you.

---

### `/pr` — Open a pull request for current work

```
/pr               # PR for the current branch
/pr --draft       # Open as a draft PR
/pr "my title"    # Override the generated title
```

What it does:
1. Detects the issue ID from the branch name
2. Stages and commits any uncommitted changes
3. Pushes the branch and creates a PR titled `FIN-12: Issue title` with `Closes FIN-12` in the body

Use this when you want a PR open for review but aren't done yet. For completed work, prefer `/done`.

---

### `/blocked` — Mark an issue as blocked

```
/blocked                     # Block the current issue
/blocked FIN-12              # Block a specific issue
/blocked FIN-12 FIN-99       # FIN-12 is blocked by the existing FIN-99
```

What it does:
1. If you provide a blocking issue ID, links the two issues
2. If the blocker is unknown, creates a new blocker issue with urgent priority and links it
3. Offers `/next` so you can keep moving

Blocked issues disappear from `linear issues --unblocked` automatically.

---

### `/estimate` — Bulk-estimate unestimated issues

```
/estimate                    # Work through all unestimated issues
/estimate FIN-12             # Estimate a specific issue
/estimate --project "P1"     # Scope to a project
```

What it does:
1. Fetches open issues with no estimate set
2. For each: shows the issue, reads referenced code if needed, suggests a t-shirt size with a rationale
3. Accepts your confirmation or adjustment, then applies it with `linear issue update --estimate`

At the end, flags any L/XL issues and suggests `/split`.

---

### `/scope` — Audit a project for gaps

```
/scope                  # Audit the current default project
/scope "Phase 1"        # Audit a specific project
```

What it does:
1. Loads the project's issues and reads `product.md` for intended scope
2. Looks for: unclear issues, unestimated issues, orphaned issues, scope gaps, oversized issues, stale in-progress
3. Presents findings grouped by category with specific issue IDs
4. Offers to act on each finding — never creates or edits anything without confirmation

Use this before a sprint to catch surprises before they happen.

---

### `/split` — Break a large issue into sub-issues

```
/split FIN-12     # Split a specific issue
/split            # Split the current issue (detected from branch)
```

What it does:
1. Reads the issue's description and acceptance criteria
2. Proposes a breakdown of 3–5 sub-issues with estimates and dependency ordering
3. On confirmation, creates the sub-issues with `--parent` and `--blocked-by` set correctly
4. Adds a comment to the parent naming the sub-issues, then offers `/start` for the first one

Always shows the proposed breakdown before creating anything.

---

### `/work` — Plan and start immediate work, no Linear required

```
/work "add rate limiting to the API"
/work                    # Describe what you want to do
```

What it does:
1. Quickly frames the problem (what, where, constraints)
2. Proposes an approach — what changes, which files, scope boundary
3. Creates a descriptive git branch and starts implementing immediately

No Linear ticket needed. If the work grows in scope mid-implementation, `/work` flags it and offers to promote it to a proper issue. Use `/plan` instead when you need scheduling, collaboration, or tracking from the start.

---

## PR Lifecycle Reference

All run inside Claude Code using the `!` prefix. No browser needed.

Aliases are configured globally (`gh alias list` to verify) for the most common commands:

| What | Short | Full command |
|------|-------|-------------|
| Check CI status | `!gh prc` | `!gh pr checks` |
| View PR description | `!gh prv` | `!gh pr view` |
| View full diff | `!gh prd` | `!gh pr diff` |
| List all open PRs | `!gh prl` | `!gh pr list` |
| Merge your own PR (squash) | `!gh prm` | `!gh pr merge --squash --delete-branch` |
| Check out a PR locally | `!gh co 42` | `!gh pr checkout 42` |
| Approve (someone else's PR) | — | `!gh pr review --approve` |
| Request changes | — | `!gh pr review --request-changes -b "reason"` |
| Re-run failed CI | — | `!gh run rerun <run-id> --failed` |
| Watch live CI output | — | `!gh run watch` |
| Clean up stale local branches | — | `!git fetch --prune` |

> Note: GitHub does not allow approving your own PR. Skip the approve step when self-merging.

---

## Useful Linear CLI Commands

Run these in a terminal or with `!` prefix inside Claude Code.

```bash
# Overview
linear roadmap                      # Projects, milestones, progress
linear issues --unblocked           # Ready to work on
linear issues --mine                # Your assigned issues
linear issue show FIN-5             # Full issue details with parent context

# Context (scope all commands to a project/milestone)
linear project open "Phase 1"
linear milestone open "Sprint 3"
linear project close

# Create issues
linear issue create --title "Fix bug" --assign --estimate S
linear issue create --title "Step 2" --parent FIN-5 --estimate M

# Update issues
linear issue update FIN-5 --append "## Notes\n\nFound root cause in auth.ts:142"
linear issue update FIN-5 --check "validation"   # Check off a checklist item

# Blockers
linear issue create --title "Need API key" --blocks FIN-5
# FIN-5 disappears from --unblocked until this is resolved

# Git
linear branch FIN-5                 # Create and check out branch: fin-5-issue-title
```

---

## Estimate Sizes

| Size | Meaning |
|------|---------|
| XS | Trivial, < 1 hour |
| S | Small, a couple hours |
| M | Medium, about a day |
| L | Large, multi-day — consider breaking down |
| XL | Very large — break down before starting |

L/XL issues should be split into sub-issues before work begins.

---

## Claude + Jira

Same workflow, same AI tool, different issue tracker. Commands live in `claude-jira/`. Use this variant if your team is on Jira instead of Linear.

### Installation

**1. Get the commands**

```bash
# Per-project
cp -r claude-jira /path/to/your/project/.claude

# Global
cp claude-jira/commands/* ~/.claude/commands/
cp -r claude-jira/skills/* ~/.claude/skills/
```

Use one variant per project — install either `claude/` (Linear) or `claude-jira/` (Jira), not both.

**2. Jira CLI**

```bash
brew install jira-cli
jira init
```

`jira init` will prompt for your Jira server URL, email, and API token (generate one at `https://id.atlassian.com/manage-profile/security/api-tokens`), then ask which project and board to default to.

**3. GitHub CLI**

```bash
brew install gh
gh auth login
```

**4. GitHub-Jira integration (optional)**

Install the Jira GitHub app in your Jira workspace. Once connected, PRs that include `Closes PROJ-42` in the body auto-transition the issue to Done on merge. Without it, run `jira issue move PROJ-42 "Done"` manually after merge.

### Daily Loop

```
/standup → /next → implement → /done → /next → repeat
```

### Commands

| Command | What it does |
|---------|-------------|
| `/next` | Pick a To Do issue, assign yourself, create branch |
| `/done` | Commit, push, PR, wait for CI, merge, pull main |
| `/plan` | Create Jira issues inline via CLI |
| `/standup` | Daily summary from Jira + git |
| `/start` | Set up a known issue without immediately implementing |
| `/pr` | Open a PR for early review without closing the issue |
| `/blocked` | Mark an issue blocked and keep moving |
| `/work` | Plan and start immediate work without creating a Jira ticket first |
| `/issues` | Browse issues by sprint or epic |
| `/estimate` | Bulk-size unestimated issues |
| `/scope` | Audit a project for gaps before a sprint |
| `/split` | Decompose an L/XL issue into sub-issues |
| `/review` | Review open PRs |
| `/sync` | Detect drift between Jira and branch/PR state |
| `/triage` | Groom untriaged issues interactively |
| `/retro` | Sprint retrospective → action items |
| `/whatchanged` | Generate a management progress report since the last run |
| `/bugs` | Scan codebase → create Bug issues |
| `/debt` | Scan codebase → create Task issues tagged tech-debt |
| `/deps` | Audit dependencies → create issues |
| `/onboard` | Codebase orientation |
| `/release` | Generate changelog → GitHub release |
| `/diagnose` | Root-cause a bug before writing any fix |
| `/sit` | Stop, Inspect, Think — mid-task self-audit |

### Key Differences Between the Two Variants

| | Claude + Linear | Claude + Jira |
|---|---|---|
| Branch creation | `linear branch ISSUE-1` | `git checkout -b PROJ-1-slug` |
| Start issue | `linear issue start` | `jira issue move + jira issue assign` |
| Close issue | GitHub integration auto-closes | `jira issue move "Done"` or GitHub-Jira app |
| Sprints | Cycles + milestones | `jira sprint list/add` |
| Grouping | Projects | Epics |
| Priorities | urgent/high/medium/low | Highest/High/Medium/Low/Lowest |
| Issue tracker integration | Linear MCP + CLI | Jira CLI only |
