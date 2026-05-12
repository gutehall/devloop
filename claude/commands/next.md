# /next - Find the next issue to work on

Works for any type of issue — code, documents, decks, reviews, planning, ops tasks.

## Usage

```
/next           # Show unblocked issues to choose from
/next ISSUE-12  # Skip selection, start working on ISSUE-12 directly
```

## If no issue ID is provided

1. Run `linear issues --unblocked` to get issues ready to work on
2. Parse the output and count the issues
3. Present options using the format below

### Presenting Options

Present interactive options:
- Up to **3 issues** (CLI already sorts yours first)
- **Always** include "Product planning" as an option
- If >3 issues, note how many more
- User can always type a specific issue ID

## Starting Work on an Issue

When the user selects an issue (or provides one directly via `/next ISSUE-12`):

1. Run `linear issue start <id>` to assign and set In Progress
2. Run `linear issue show <id>` to display full context
3. **Detect work type** (see below)
4. Follow the appropriate path

## Work Type Detection

Classify the issue based on its title, description, and labels:

**Code work** — any of: implementation, bug fix, feature, refactor, migration, API, infrastructure, test. Also: working directory contains a git repo.

**Non-code work** — any of: document, report, deck, presentation, plan, review, research, process, comms, meeting, analysis, spec, strategy.

If ambiguous, ask: "Is this code work or non-code work?"

---

## Path A: Code Work

1. Ask: "Should I branch from `main` or `develop`?" — wait for the answer before continuing
2. Check out the chosen base branch: `git checkout <main|develop>`
3. Run `git pull` to sync the local repo before branching
4. Run `linear branch <id>` to create a git branch and check it out
5. Read the full issue description and acceptance criteria
6. Explore relevant code (grep for symbols/files mentioned in the issue)
7. Implement — minimal solution, follow existing patterns, no over-engineering

### Implementation Rules

- Read relevant files before coding
- Minimal solution, focused changes only
- No unrelated refactors
- Follow existing patterns
- Check acceptance criteria before finishing
- If scope expands, stop and flag it
- Make the safest reasonable assumption on ambiguity; document it in a comment or PR

---

## Path B: Non-Code Work

1. Read the full issue description and acceptance criteria
2. Identify the deliverable (document, deck, plan, review, etc.)
3. Help produce it — draft, outline, structure, or content as appropriate
4. When output is ready, tell the user to run `/done` to close the issue

No git branch. No code. Focus entirely on producing the deliverable.

---

## If they choose "Product planning"

Ask what they want to focus on:
- Review and prioritize backlog
- Brainstorm new features
- Plan the next phase

Then follow the product-planning skill guidelines.

## If no unblocked issues are found

1. Run `linear issues --open` to check what's actually there
2. **If all issues are blocked:** show the blocked list with blocking issue IDs. Say: "All issues are blocked. You can resolve a blocker or use `/plan` to add new unblocked work." Offer the blocked list so the user can pick one to unblock.
3. **If the backlog is empty:** say "No open issues found." and offer:
   - "Product planning" to create new issues with `/plan`
   - A prompt to describe something new: `/plan "..."`
4. **If `linear` is not configured:** if the CLI returns an auth error, say "Linear CLI is not set up. Run `linear login` to get started."

## Notes

- Always use long flags (--unblocked, not -u) for clarity
- The CLI already sorts your assigned issues first
- The "Product planning" option ensures there's always something productive to do
