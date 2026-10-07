# STATUS handoff

Use this when a chat is stuck, confused, or very long. Ask the agent to fill it in, save it as `STATUS.md`, then open a **fresh chat** and give it the file.

This is the highest-value trick in the whole workflow. It works because long chats carry their entire history on every turn, which burns your usage limit and hands the model a pile of context that includes every wrong turn you took. A fresh chat plus this file gives it only what matters.

## Prompt to get the agent to write it

```
Write a detailed STATUS.md for this project. Include the items below. Be specific,
and be honest about what we've verified versus what we only assume.
```

## Template

```markdown
# STATUS

## Goal
[What the project is trying to do, in one or two sentences]

## Setup
- Games and exact versions: [...]
- Agent / model: [...]
- Other tools and loaders: [...]
- Where things are: [paths]

## What works (tested)
- [...]

## What doesn't work yet
- [...]

## Which game owns the player
[Which side is authoritative for player position, if this is a passthrough mod.
Write "not applicable" for a rewrite.]

## The current problem
[Exactly what's going wrong: what we did, what we expected, what happened]

## Evidence
[Relevant log lines, errors, numbers]

## What we've already tried
- [Approach 1]: [result]
- [Approach 2]: [result]

## Ideas not tried yet
- [...]

## Files that matter
- [file]: [why]
```

## Prompt for the fresh chat

```
Read STATUS.md and the project's AGENTS.md. Don't change any code yet. Explain the
problem back to me in your own words, then suggest several different ways to
troubleshoot it, starting with the ones we haven't tried.
```

## Why it works

Three reasons:

1. **Your usage limit.** Long chats re-read the whole transcript every turn. A fresh chat with a 2 KB file is much cheaper than turn 400 of an argument.
2. **No sunk cost.** The old chat has already committed to an approach and will keep defending it. A fresh chat has no ego attached.
3. **The agent can plan.** Given a clean statement of the problem it can offer a different approach. In a long chat it tends to keep tweaking the failing thing.

This helps most with the less capable models, but everyone uses it.

## Other times to write one

- Before you stop for the night, so you can pick up tomorrow without rereading the chat
- Before switching models or tools
- Before asking for help in a forum or Discord thread; the STATUS file *is* the bug report
- When the agent has clearly lost the plot

## Keeping chat size down generally

- Start a fresh chat when you switch topics, not only when you're stuck.
- Ask the agent to update `MODLOG.md` as it works. The log is your long-term memory, so the chat doesn't have to be.
- Keep the rules in `AGENTS.md` rather than retyping them every session.

See [`MODLOG-template.md`](MODLOG-template.md) for the running log, [`AGENTS-starter.md`](AGENTS-starter.md) for the rules file, [`BRIDGE-CONTRACT.md`](BRIDGE-CONTRACT.md) for who owns what, and [`PLAYTEST-report.md`](PLAYTEST-report.md) for recording what you tested.
