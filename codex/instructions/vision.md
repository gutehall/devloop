# /vision - Strategic future planning for a project

Facilitate a deep, honest conversation about where the project is heading — 1 year, 3 years, 5 years out. Surface logical next steps, identify capability gaps, and produce a strategic view worth acting on.

## Usage

```
/vision                          # Open-ended future exploration
/vision "1 year"                 # Focus on the near-term horizon only
/vision "scale"                  # Explore a specific strategic theme
/vision "compete with X"         # Explore competitive positioning
```

## What this is

A structured thinking session about the future — not a ticket-creation exercise. No Jira writes until the user explicitly asks. The output is a strategy document: what the project should become, why, and what would have to be true for it to get there.

This command uses the vision skill. Follow it exactly.

---

## Flow

### 1. Read current state

Before thinking forward, understand where things stand now:

- List open epics and issues in Jira:
  ```bash
  jira board list --plain
  jira epic list --plain
  jira issue list --resolution Unresolved --plain
  ```
- Read `product.md` if it exists at the project root
- Read `README.md` and any architecture docs you can find in the repo
- Check recent git log for what has been actively worked on: `git log --oneline -20`

If the project is unfamiliar, say so and ask the user to describe it in a sentence before continuing.

### 2. Invoke the vision skill

Follow the full vision skill protocol:

> **Follow the vision skill.**

The skill walks through:
- Grounding questions to establish the current position
- Horizon mapping: 1yr / 3yr / 5yr
- Forces of change: technology, users, competition, regulation
- Capability gaps: what would need to exist
- Strategic bets: which futures to commit to
- Improvement themes: weaknesses worth addressing regardless of horizon
- Assumption stress-tests: what if the core assumptions are wrong

Do not skip sections. Surface tensions and tradeoffs the user may not have considered. Disagree with the user when the evidence supports it. The goal is honest strategy, not validation.

### 3. Synthesize

At the end of the conversation, produce a structured vision document:

```markdown
## Vision Summary

### Where we are
[2–3 sentences: current state, strengths, known weaknesses]

### Where we're going

**1 year:** [What the project looks like. What's been built. What users can do.]
**3 years:** [The bigger shift. What new capabilities exist. What's changed about the user or market.]
**5 years:** [The ambitious version. What this could become if the bets pay off.]

### Strategic bets
[2–4 things we're committing to. Each with a one-sentence rationale.]

### Capability gaps
[What doesn't exist yet that would be required to reach the 3yr or 5yr horizon]

### Improvement themes
[Cross-cutting areas — architecture, UX, performance, reliability — worth addressing at any horizon]

### Assumptions to validate
[The things that could be wrong. What early signals to watch for.]

### Next planning horizon
[What to focus on in the next 3–6 months, given all of the above]
```

### 4. Offer to hand off

End with:

> "Ready to turn this into a roadmap? Create Jira epics for the next planning horizon, or I can start it for you."

Suggested Jira creation flow:
```bash
# Create an epic per strategic theme
jira epic create -s"[Theme name]" -b"[Vision context]"

# Break capability gaps into stories
jira issue create -tStory -s"[Capability gap]" --parent [EPIC-KEY]
```

---

## Rules

- No Jira writes during the vision session — this is thinking, not execution
- Don't converge prematurely. If the answer feels obvious, look for what's being ignored.
- Challenge assumptions. Ask "what would have to be true for that to happen?"
- Surface the risks. A future without risks is not honest strategy.
- One horizon at a time. Don't let all three collapse into a blur.
- Keep the conversation moving — short exchanges are better than walls of text
- The vision document is the deliverable, not the conversation
