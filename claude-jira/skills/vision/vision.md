# Vision Protocol

A structured thinking protocol for strategic future planning. Run this when exploring where a project should go over a 1–3 year horizon.

---

## Phase 1: Establish Ground Truth

Before imagining the future, get honest about the present. Ask these questions. Don't move forward until there's a clear answer to each.

**The project:**
- What does this project actually do, in one sentence? (Not the vision — the reality today.)
- Who uses it, and what do they use it *for*? (The actual job, not the stated purpose.)
- What's working well enough that you'd want to preserve it?
- What's broken, slow, or embarrassing that you avoid mentioning?

**The trajectory:**
- What direction has the project been moving in the last 6–12 months?
- Is momentum building, holding steady, or fading?
- What's the most significant thing that was built or shipped recently? Did it land?

**The context:**
- Who are the competitors or alternatives? What do they do better?
- What do users complain about most?
- What would it take to lose? (What would kill this project or make it irrelevant?)

Ground truth check: *Can you state, in two sentences, where the project is right now and why it matters?* If not, stop here and work on that first. A vision built on unclear present-state is not reliable.

---

## Phase 2: Forces of Change

The future doesn't just come from the team's decisions. External forces shape it too. Work through each category.

**Technology:**
- What technologies are maturing that could change how this is built or what it can do?
- What's becoming cheaper, faster, or ubiquitous that wasn't before?
- What tech bets does the current architecture depend on — and what happens if those bets age poorly?

**Users:**
- Are the project's users changing? Younger, older, more technical, less technical?
- What are users starting to expect that didn't exist as an expectation two years ago?
- Is the user base growing, shrinking, or shifting in composition?

**Competition and market:**
- Which competitors are accelerating? What are they doing differently?
- Are new entrants entering the space? From what angle?
- Is the market consolidating (fewer bigger players) or fragmenting (more niches)?

**Regulation and risk:**
- Are there regulatory shifts on the horizon that would affect this space?
- What data, privacy, or compliance risks exist and how are they trending?

**Internal:**
- How will the team size and capability change? Growing, stable, or constrained?
- What technical debt is accumulating that will constrain options?
- What decisions made in the last year are hard to reverse, and how do they shape the future?

---

## Phase 3: Horizon Mapping

Work through each horizon separately. Don't blur them.

### 1-Year Horizon

*The near term. What's realistic given current momentum, team, and constraints.*

- If the current trajectory continues, what does the project look like in 12 months?
- What's the one thing that, if done well, would make the next year clearly successful?
- What should be *stopped*, *paused*, or *handed off* to make room for that?
- What's the biggest risk to the 1-year picture? What could derail it?

Output: A concrete, honest description of 1-year state. Not aspirational — achievable.

### 3-Year Horizon

*The medium term. Where strategy bets start to matter.*

- If you made the right bets, what does the project enable that it can't today?
- What new user needs could emerge that this project is uniquely positioned to serve?
- What would need to be true about the technology, the team, and the product to reach this?
- What's the version of this project that you'd be proud of in 3 years?

Push the user here. "More features" is not a 3-year horizon. Ask: *What fundamentally changes about what this project can do or who it serves?*

Output: 2–3 scenarios for the 3-year state. Identify which one the team is currently pointed at, and whether that's intentional.

---

## Phase 4: Strategic Bets

A strategy is a set of *choices*. Not everything can be important. This phase forces prioritization.

Ask:
- Given the two horizons, what are the 2–4 things this project is *committing to*?
- What is this project *not* doing — and is that explicit or just neglect?
- Which bets are load-bearing? (Things that, if wrong, change the whole picture.)
- Which bets are reversible? (Things you can walk back if they don't work.)

A bet is not a feature. It is a directional commitment: *We are building toward X because we believe Y.* Each bet should be statable in one sentence with a rationale.

Output: A short list of named bets with rationale. Explicitly note which are load-bearing vs. reversible.

---

## Phase 5: Capability Gaps

The gap between the current state and the 3-year horizon reveals what would need to be built, hired, or decided.

For each horizon gap:
- What technical capabilities don't exist yet that would be required?
- What process, tooling, or infrastructure would need to change?
- What does the team need to learn or hire for?
- What decisions are currently deferred that will need to be made?

Don't just list features. Capability gaps include: architecture decisions, missing integrations, team skills, operational maturity, and even product-market positioning.

Output: A prioritized list of gaps, roughly categorized by: *must close in 1yr*, *need by 3yr*.

---

## Phase 6: Improvement Themes

Beyond new capabilities, every project has systemic improvement areas — things that don't fit on a feature roadmap but matter for long-term health.

Work through each area:
- **Performance and reliability:** Where does the system strain under load? What would break at 10x scale?
- **Developer experience:** Where is the codebase slowing the team down? What's the most painful part to change?
- **User experience:** What do users struggle with most? Where does the product feel unfinished?
- **Security and compliance:** What's the current risk posture? What's quietly accumulating?
- **Architecture:** What structural decisions would be better if made differently? What's the highest-interest technical debt?
- **Observability and operations:** Can the team tell when things are going wrong? How fast can they respond?

Output: The top 3–5 improvement themes, each with a one-line rationale for why they matter for the long term.

---

## Phase 7: Assumption Stress-Test

Every plan has load-bearing assumptions. The goal here is to find them and name them explicitly, so they can be watched.

For each major element of the vision, ask:
- What would have to be true for this to work?
- What's the most likely way this turns out to be wrong?
- What would be the early signal that this assumption is failing?
- If this assumption turned out to be wrong, what would change about the plan?

Typical categories:
- *User assumptions* — "Users will want this", "Users will pay for this", "Users can learn this"
- *Technology assumptions* — "This tech will mature", "This approach will scale", "This dependency will remain viable"
- *Market assumptions* — "The competitive landscape stays roughly as it is", "The regulatory environment won't change"
- *Team assumptions* — "The team can build this", "We can hire for this gap", "This approach fits our current skill set"

Output: A named list of key assumptions, flagged as *confident*, *uncertain*, or *risky*, with the early signal to watch.

---

## Output Format

After working through the phases, produce:

```markdown
## Vision Summary — [Project Name]
*Generated: [date]*

### Where we are
[2–3 sentences: current state, what's working, what isn't]

### Where we're going

**1 year:** [Concrete, achievable state. One paragraph.]
**3 years:** [The bigger shift. What's fundamentally different. One paragraph.]

### Strategic bets
| Bet | Rationale | Type |
|-----|-----------|------|
| [name] | [one sentence] | load-bearing / reversible |

### Capability gaps
**Close in 1yr:** [...]
**Need by 3yr:** [...]

### Improvement themes
1. [Theme] — [why it matters long-term]
2. ...

### Key assumptions
| Assumption | Confidence | Early signal to watch |
|------------|------------|----------------------|
| [assumption] | confident / uncertain / risky | [signal] |

### Next planning horizon
[What to focus on in the next 3–6 months, given all of the above. One paragraph.]
```

---

## Tone and Honesty

A vision session is not a hype exercise. These standards apply throughout:

- **Name the risks.** A plan without risks named is not a plan.
- **Push back on easy answers.** If the vision comes together too cleanly, something is being glossed over.
- **Don't agree to be agreeable.** If the user's stated direction seems misaligned with the evidence, say so.
- **Uncertainty is honest output.** "We don't know" is a valid and important finding.
- **Specificity over generality.** "Better UX" is not a strategic direction. "Reduce the time to first value from 30 minutes to under 5" is.
- **Short-term and long-term are different conversations.** Keep them separate. Don't let quarterly pressures collapse the 3-year view.

---

## When vision is most valuable

- Before a major planning cycle (quarterly, annual)
- When the team feels like it's building without direction
- When a new competitor or technology shift demands a strategic response
- When the backlog has grown large and priorities feel unclear
- After a significant milestone, to reorient for the next phase
- When the product has drifted from its original purpose and needs a course check
