# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Two parallel sets of workflow automation — both powered by Claude Code, differing only by issue tracker — that implement the same development loop: pick issue → branch → implement → PR → merge → repeat.

| Variant | Directory | Issue tracker | Commands location |
|---------|-----------|--------------|-------------------|
| Claude + Linear (primary) | `claude/` | Linear (via MCP + CLI) | `claude/commands/` |
| Claude + Jira | `claude-jira/` | Jira (via Atlassian MCP + CLI) | `claude-jira/commands/` |

Linear is the daily-driver variant. The Jira variant exists so teams on Jira can adopt the same loop with the same AI tool.

## Structure

```
claude/
  commands/           # Slash command specs (.md prompt files loaded by Claude Code)
  skills/             # Reusable skill modules referenced from commands
    linear-cli/       # Linear CLI reference and best practices
    github-cli/       # GitHub CLI reference (gh)
    product-planning/ # Product thinking methodology
    diagnostic/       # Structured debugging and diagnosis protocol
    sit/              # Stop, Inspect, Think — mid-task self-audit
    bugs/             # Bug scanning methodology
    debt/             # Tech debt scanning methodology
    code-review/      # PR review methodology
    (+ more)

claude-jira/
  commands/           # Same command set, adapted to use Jira CLI instead of Linear
  skills/             # Same skill modules, adapted for Jira instead of Linear
    jira-cli/         # Jira CLI reference (instead of linear-cli)
    (rest mirrors claude/skills/)
```

Both variants install into the project's `.claude/` directory — a project uses one or the other, not both.

## How commands and skills work

**Commands** (`claude/commands/*.md`, `claude-jira/commands/*.md`) are prompt files — they are instructions written *to* Claude Code, not documentation for humans. They are prescriptive: exact CLI commands, exact MCP tool names, exact output formats. Each command is self-contained.

**Skills** (`*/skills/*/SKILL.md`) are reusable modules. A command invokes a skill by name: e.g. `Follow the diagnostic-thinking skill.` Skills have YAML frontmatter:

```yaml
---
name: skill-name
description: When to invoke this skill (used by the AI to decide relevance)
allowed-tools: Bash(linear:*), Bash(gh:*)  # optional tool restrictions
---
```

## MCP tool names

Each variant registers its own MCP server:

```bash
# Linear variant
claude mcp add --transport http linear-server https://mcp.linear.app/mcp
# → tool prefix: mcp__claude_ai_Linear__

# Jira variant
claude mcp add --transport sse atlassian-server https://mcp.atlassian.com/v1/sse
# → tool prefix: mcp__claude_ai_Atlassian__
```

If a server is registered under a different name, all references in the corresponding `commands/` and `skills/` directories must be updated to match.

The Atlassian MCP gives Claude richer read access for Jira (search issues, fetch comments, follow links). The `jira` CLI is still required for writes that the MCP doesn't cover (status transitions, branch creation, sprint management).

## Key differences between the two variants

- `claude/` uses Linear MCP (`mcp__claude_ai_Linear__*`) + the `linear` CLI
- `claude-jira/` uses Atlassian MCP (`mcp__claude_ai_Atlassian__*`) + the `jira` CLI
- Both reference `gh` for GitHub
- Skills are largely identical between variants; the only structural difference is `linear-cli` (in `claude/`) vs `jira-cli` (in `claude-jira/`)

## Working in this repo

- Write commands as instructions to Claude Code, not as human documentation
- Keep each command self-contained — no runtime dependencies on other commands
- When adding or changing a command in one variant, mirror the change in the other variant where the methodology applies
- Skills are shared methodology; keep them tool-agnostic where possible, or fork with a variant-specific note

## Testing changes

No test suite. Validate by installing commands in a test project and running the slash command in Claude Code:

```bash
# Linear variant
cp claude/commands/* /path/to/project/.claude/commands/
cp -r claude/skills/* /path/to/project/.claude/skills/

# Jira variant
cp claude-jira/commands/* /path/to/project/.claude/commands/
cp -r claude-jira/skills/* /path/to/project/.claude/skills/
```
