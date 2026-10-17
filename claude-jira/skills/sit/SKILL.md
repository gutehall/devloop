---
name: sit
description: Stop, Inspect, Think: honest mid-task check of whether the current approach is right. Only /sit, or when a command names this skill. Do not trigger proactively.
---

# SIT Skill

SIT stands for **Stop, Inspect, Think**. A structured self-audit protocol for Claude Code.

## When to use

Trigger this skill only when:
- The user types "/sit" or says "sit", "pause", "stop and think", "rethink", "wait", "are you sure?", "step back"

## How to use

1. **Read the protocol**: Load `sit.md` from this skill folder — full SIT protocol with all inspection questions and output format.

2. **Run the protocol honestly**: Work through each section. Don't skip steps. Don't write optimistic filler.

3. **Output the structured reflection**: Use format defined in `sit.md`.

4. **Commit to a decision**: End with one of four outcomes — Continue, Correct course, Ask the user, or Stop and report.

## Reference file

→ Read [`sit.md`](./sit.md) for the full protocol, reflection questions, output template, and decision table.

## Key principle

SIT is not a summary. It's a genuine reassessment. Test: *would the user, reading the output, trust Claude actually stopped and thought — or does it read like going-through-the-motions?*

Honest uncertainty is correct output. False confidence is not.
