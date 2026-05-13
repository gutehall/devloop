# SIT — Stop, Inspect, Think

A structured pause protocol for Claude Code. When invoked, Claude stops all forward momentum and performs a honest self-audit before deciding whether to continue, correct course, or ask for clarification.

---

## The SIT Protocol

### 1. STOP
Put down the tools. Do not run the next command. Do not write the next line of code.
Take a breath (metaphorically). The work can wait thirty seconds.

---

### 2. INSPECT — Look back at what has been done

Answer these questions honestly, in order:

**What was I actually asked to do?**
- Restate the original goal in one sentence, as if explaining it to someone new.
- If you can't do this clearly, that is a red flag.

**What have I done so far?**
- List the concrete actions taken: files created/modified, commands run, decisions made.
- Be specific. "I edited some files" is not good enough.

**What assumptions did I make?**
- What was never explicitly stated but was treated as given?
- Which of those assumptions could reasonably be wrong?

**What has gone wrong, or could go wrong?**
- Any errors, unexpected outputs, or surprising results so far?
- Any steps that felt uncertain but were pushed through anyway?
- Any shortcuts taken that might cause problems later?

---

### 3. THINK — Evaluate the path forward

**Am I still solving the right problem?**
- Has scope crept? Has the approach drifted from the intent?
- Would the user, seeing what has been done, feel it matches their request?

**Is the current approach the right one?**
- Is there a simpler, safer, or more direct route?
- Has new information emerged that should change the plan?
- Is the current path getting more complex with each step — a sign it may be wrong?

**What are the next steps, and are they sound?**
- State what you were about to do next.
- Is that step clearly the right move, or does it require a leap of faith?
- If uncertain: stop and surface the uncertainty to the user instead of pushing forward.

---

### 4. DECIDE — Commit to an honest outcome

After the inspection, pick one:

| Decision | When to use it |
|----------|---------------|
| ✅ **Continue** | The approach is sound, assumptions are valid, and the next step is clear. |
| 🔀 **Correct course** | Something was off, but you can fix it without user input. State what changes. |
| ❓ **Ask the user** | A key assumption may be wrong, or two paths are genuinely equal. Ask one focused question. |
| 🛑 **Stop and report** | The work has gone sideways and continuing would make things worse. Explain what happened. |

---

## Output Format

When SIT runs, structure the output like this:

```
## 🪑 SIT — Pausing to reflect

**Original goal:** [one sentence]

**What's been done:**
- [action 1]
- [action 2]
- ...

**Assumptions made:**
- [assumption 1] — [still valid? / uncertain / probably wrong]
- ...

**Concerns / observations:**
- [anything that felt off, errors seen, complexity growing, etc.]

**Assessment:**
[2–4 sentences on whether the approach is right and why]

**Decision:** [Continue / Correct course / Ask the user / Stop and report]
[If correcting or asking: one clear sentence on what changes or what the question is]
```

---

## Tone and Honesty

SIT is not a performance of reflection. It is actual reflection.

- Do not write "Everything looks good!" if things are uncertain.
- Do not justify past decisions just because they were made. Sunk cost is not a reason.
- Do not ask three questions when one will do.
- Be direct. The user reads the output of SIT to get a genuine status, not reassurance.

If a step was wrong: say so plainly. This is not a failure — catching it early is the whole point.

---

## When SIT is most valuable

- After 10+ tool calls with no user confirmation
- When a task turns out to be more complex than expected
- When something unexpected has happened (error, missing file, surprising output)
- Before a destructive or hard-to-reverse action
- When the next step is unclear or two approaches seem equally valid
- When you notice scope creep — doing more than was asked
