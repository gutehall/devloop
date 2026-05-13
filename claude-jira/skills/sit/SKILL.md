---
name: sit
description: Forces Claude Code to pause, reflect, and honestly assess whether the current approach is correct before continuing. Use this skill whenever the user says "sit", "pause and think", "wait, stop", "take a step back", "are you sure about this?", "rethink what you're doing", or any phrase that signals they want Claude to stop and self-audit rather than keep going. Also trigger proactively when many tool calls have been made without user confirmation, when something unexpected has happened, when the task has grown more complex than anticipated, or before any destructive/hard-to-reverse action.
---

# SIT Skill

SIT stands for **Stop, Inspect, Think**. It is a structured self-audit protocol.

## When to use

Trigger this skill when:
- The user says "sit", "pause", "stop and think", "rethink", "wait", "are you sure?", "step back"
- Many consecutive tool calls have been made without checking in
- Something unexpected, ambiguous, or error-like has occurred
- The task has grown in scope or complexity beyond what was originally described
- You are about to do something that is hard to undo
- Two approaches seem equally valid and a choice must be made

## How to use

Work through each phase honestly. Do not skip steps. Do not write optimistic filler.

### 1. STOP
Put down the tools. Do not run the next command. The work can wait.

### 2. INSPECT
Answer in order:
- **What was I actually asked to do?** — one sentence
- **What have I done so far?** — list concrete actions: files modified, commands run, decisions made
- **What assumptions did I make?** — which ones could be wrong?
- **What has gone wrong, or could go wrong?** — errors, uncertainty, shortcuts taken

### 3. THINK
- Am I still solving the right problem? Has scope crept?
- Is the current approach the right one, or is there a simpler path?
- What am I about to do next — and is that clearly the right move?

### 4. DECIDE
Pick one:

| Decision | When |
|----------|------|
| ✅ Continue | Approach is sound, next step is clear |
| 🔀 Correct course | Something was off but fixable without user input — state what changes |
| ❓ Ask the user | Key assumption may be wrong — ask one focused question |
| 🛑 Stop and report | Work has gone sideways — explain what happened |

## Output format

```
## 🪑 SIT — Pausing to reflect

**Original goal:** [one sentence]

**What's been done:**
- [action 1]
- [action 2]

**Assumptions made:**
- [assumption] — [still valid? / uncertain / probably wrong]

**Concerns / observations:**
- [anything off, errors seen, growing complexity]

**Assessment:**
[2–4 sentences on whether the approach is right and why]

**Decision:** [Continue / Correct course / Ask the user / Stop and report]
[If correcting or asking: one clear sentence]
```

## Key principle

SIT is not a summary. It is a genuine reassessment. Honest uncertainty is correct output. False confidence is not.
