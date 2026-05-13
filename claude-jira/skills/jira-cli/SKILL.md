---
name: jira-cli
description: Manage Jira issues, sprints, and epics from the command line. Full CLI reference for automating Jira project management.
---

# Jira CLI

A terminal client for Jira Cloud and Jira Server.

Install: `go install github.com/ankitpokhrel/jira-cli/cmd/jira@latest`
Or: `brew install jira-cli`

## First-Time Setup

```bash
jira init
```

Interactive setup that:
1. Asks for your Jira server URL (e.g. `https://yourcompany.atlassian.net`)
2. Asks for your email and API token (create at https://id.atlassian.com/manage-profile/security/api-tokens)
3. Asks which project to default to
4. Asks for board type (scrum or kanban)
5. Saves config to `~/.config/.jira/.config.yml`

## Configuration

Config file: `~/.config/.jira/.config.yml`

```yaml
server: https://yourcompany.atlassian.net
login: your@email.com
project: PROJ
board:
  type: scrum
  id: 1
```

Environment vars:
- `JIRA_API_TOKEN` — API token (overrides config file)
- `JIRA_AUTH_TYPE` — `bearer` or `basic`

## Quick Reference

```bash
# Identity
jira me                                   # Show current user account ID

# Issues
jira issue list                           # Open issues in configured project
jira issue list -a"$(jira me)"            # Your assigned issues
jira issue list -s"To Do"                 # Filter by status
jira issue list -s"In Progress"
jira issue list --sprint "Sprint 12"      # Issues in a sprint
jira issue list --epics PROJ-100          # Child issues of an epic
jira issue list --label bug               # Filter by label
jira issue list --type Bug                # Filter by type (Bug/Story/Task/Epic)
jira issue list --priority High           # Filter by priority
jira issue list --resolution Unresolved   # Open issues only
jira issue list --plain                   # Suppress interactive UI
jira issue view PROJ-123                  # Full issue details + comments
jira issue create -tStory -s"Title" -b"Description"
jira issue create -tBug -s"Bug title" -b"Desc" --priority High
jira issue create -tTask -s"Task" --parent PROJ-100
jira issue create -tEpic -s"Epic title" -b"Epic description"
jira issue create -tStory -s"Title" --label "bug,auth"
jira issue move PROJ-123 "In Progress"    # Transition status
jira issue assign PROJ-123 "$(jira me)"   # Assign to yourself
jira issue comment add PROJ-123 "Text"    # Add a comment
jira issue edit PROJ-123 --summary "New summary"
jira issue edit PROJ-123 --priority High
jira issue edit PROJ-123 --label "bug,security"
jira issue link PROJ-123 PROJ-456 "blocks"    # Link as blocking
jira issue unlink PROJ-123 PROJ-456
jira open PROJ-123                        # Open in browser

# Sprints
jira sprint list --board-id <id>          # All sprints
jira sprint list --board-id <id> --plain  # Clean output
jira sprint add --board-id <id> PROJ-123  # Add issue to sprint

# Epics
jira epic list                            # Epics in current project
jira epic list --plain
jira epic create -s"Epic title" -b"Description"

# Boards and projects
jira board list                           # All boards
jira board list --type scrum
jira project list                         # All accessible projects
```

## Issue Types

| Type | Use for |
|------|---------|
| Epic | Large initiative spanning multiple sprints |
| Story | User-facing feature or improvement |
| Bug | Defect or incorrect behavior |
| Task | Technical work, ops, chore |
| Sub-task | Child of any of the above |

## Priority Levels

| Priority | Meaning |
|----------|---------|
| Highest | Critical, blocking production or a release |
| High | Important, should be this sprint |
| Medium | Normal priority |
| Low | Nice to have |
| Lowest | Trivial |

## Common Statuses

Statuses are defined per workflow. Typical Scrum values:
- `Backlog` — not yet pulled into a sprint
- `To Do` — in sprint, not started
- `In Progress` — being worked on
- `In Review` — under code review or QA
- `Done` — complete

Run `jira issue move PROJ-1 --help` or view an issue to see valid transition targets for your workflow.

## Git Conventions

Always link git work to Jira issues:

```bash
# Branch naming — include issue key
git checkout -b PROJ-123-short-description

# Commit messages — include issue key for traceability
git commit -m "PROJ-123: Add cache invalidation on logout"

# PR title — include issue key for GitHub-Jira dev panel linking
gh pr create --title "PROJ-123: Add caching layer" \
  --body "## Summary\n...\n\nCloses PROJ-123"
```

### GitHub-Jira Integration

When the Jira GitHub app is installed, PRs containing `Closes PROJ-123` in the body auto-transition the issue to Done on merge.

Smart Commits (requires Jira DVCS connector):
```bash
git commit -m "PROJ-123 #in-progress Start implementation"
git commit -m "PROJ-123 #done #comment Fixes the root cause"
git commit -m "PROJ-123 #time 2h Investigating the issue"
```

## Workflow Guidelines

### Getting oriented

```bash
jira board list                           # Find your board ID
jira sprint list --board-id <id>          # See active sprint
jira issue list --sprint "Sprint 12" --resolution Unresolved --plain
jira issue list -a"$(jira me)" --plain    # Your issues
```

### Starting work

```bash
jira issue list -s"To Do" -a"$(jira me)" --plain   # What's assigned and ready
jira issue view PROJ-123                            # Read the full issue
jira issue move PROJ-123 "In Progress"              # Transition to in progress
jira issue assign PROJ-123 "$(jira me)"             # Assign to yourself
git checkout -b PROJ-123-short-description          # Create branch
```

### When a task is larger than expected

Break it into child issues:

```bash
jira issue create -tTask -s"Step 1: Research approach" --parent PROJ-123
jira issue create -tTask -s"Step 2: Implement core logic" --parent PROJ-123
jira issue create -tTask -s"Step 3: Add tests" --parent PROJ-123
jira issue move PROJ-124 "In Progress"              # Start the first child
```

### When you hit a blocker

```bash
# Link the blocking issue
jira issue link PROJ-123 PROJ-456 "blocks"
# PROJ-456 now blocks PROJ-123

# Add a comment explaining the blocker
jira issue comment add PROJ-123 "Blocked on PROJ-456: need API credentials from infra team"
```

### Adding notes while working

```bash
jira issue comment add PROJ-123 "Found root cause in auth.ts:142"
jira issue edit PROJ-123 --body "Updated: discovered X, trying Y approach"
```

### Completing work (no GitHub integration)

```bash
jira issue move PROJ-123 "Done"
```

Do not auto-close issues without confirming with the developer first.

### Sprint management

```bash
jira sprint list --board-id <id>           # See all sprints and their state
jira sprint add --board-id <id> PROJ-123   # Add an issue to the active sprint
```

## Parent Context and Epics

Epics group related stories. When creating stories under an epic:

```bash
# Create child issue under an epic
jira issue create -tStory -s"Add login page" --parent PROJ-100

# View all child issues of an epic
jira issue list --epics PROJ-100 --plain
```

## Related Skills

- **product-planning** — for thinking through a problem before creating tickets. Use when the user has an idea or vague direction, not a ready-to-implement task.
- **github-cli** — for PR creation, review, and CI checks once the Jira issue is in progress.
- **quick-start** — for immediate work that doesn't need a Jira ticket first.
