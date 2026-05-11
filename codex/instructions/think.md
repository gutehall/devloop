# /think - Reason through a problem before planning

Use this before `/plan` to explore ideas, challenge assumptions, and land on a clear direction — without touching Jira yet.

## Usage

```
/think                        # Open-ended exploration
/think "Add X feature"        # Focused reasoning about a specific idea
```

## What this is

A structured conversation to think through what you actually want to build and why — before creating any issues. No Jira reads or writes. No commitments. Just thinking.

When you're done, the output is a clear brief you can hand straight to `/plan`.

---

## Flow

### 1. Understand the starting point

If input was provided, use it as the starting point. If not, ask:

> "What's on your mind? Describe the problem, idea, or area you want to think through."

### 2. Ask the three grounding questions

Before offering any opinions, make sure you understand:

- **What's the problem?** What is broken, missing, or slow right now?
- **Who is affected?** Who experiences this problem and how often?
- **What does good look like?** How will you know when it's solved?

If the user's input already answers a question, skip it. Don't ask all three at once — let it be a conversation.

### 3. Explore the space

Once grounded, dig into:

- **Why now?** What's making this worth doing at this moment?
- **What's the simplest version?** What's the minimum that would be valuable?
- **What could go wrong?** Risks, unknowns, dependencies, edge cases.
- **What are the alternatives?** Is there a simpler path, a different framing, or something already built?
- **What are you not doing?** What's explicitly out of scope, and why?

Go deep on what the user cares about. Surface tensions and tradeoffs they may not have considered. Challenge assumptions gently but directly.

### 4. Synthesize

When the conversation has enough clarity, produce a brief summary:

```
## What we're building
[1-2 sentences]

## Why
[The core problem and who it affects]

## What good looks like
[Acceptance criteria in plain language]

## What we're not doing
[Explicit scope boundaries]

## Open questions
[Anything still unresolved that /plan will need to decide]
```

### 5. Hand off

End with:

> "Ready to turn this into Jira issues? Run `/plan` and share this brief, or I can start it for you."

---

## Rules

- No Jira CLI calls — this command is thinking-only
- Don't rush to solutions; understanding the problem is the goal
- Push back on vague or assumed requirements
- Keep the conversation focused — if scope creeps, name it
- The brief is the deliverable, not the conversation
