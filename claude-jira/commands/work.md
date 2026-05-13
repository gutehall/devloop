# /work - Plan and start immediate work, no Jira required

For tasks you want to start right now — small improvements, quick experiments, immediate fixes — without creating a ticket first.

## Usage

```
/work "add rate limiting to the API"
/work                    # Describe what you want to do
```

## Flow

Follow the quick-start skill.

## When to use /work vs /plan

| | `/work` | `/plan` |
|---|---|---|
| Start | Immediately | After planning |
| Jira ticket | Optional, after the fact | Yes, before starting |
| Best for | Small tasks, experiments, quick fixes | Larger initiatives, collaborative work |
| Output | Working code | Jira issues + `/next` |

## After the work is done

If you want to track it in Jira retroactively:
- Know the issue: `jira issue view <key>` then `/done`
- Need a new ticket: describe what you built and run `/plan` to create it, then immediately close it with `jira issue move <key> "Done"`

Or skip Jira entirely — not everything needs a ticket.
