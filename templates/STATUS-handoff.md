# STATUS handoff

Use this when a chat is genuinely stuck: the agent is looping on the same failure, or it has committed to an approach that is not working. Ask the agent to fill it in, save it as `STATUS.md`, then open a **fresh chat** and give it the file.

This is an escape hatch, not routine housekeeping. In normal work, stay in one chat and let prompt caching do the work, because a long chat with a warm cache is much cheaper than starting over. See [Why it works](#why-it-works).

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

Two reasons, and neither is about saving tokens:

1. **No sunk cost.** The old chat has already committed to an approach and will keep defending it. A fresh chat has no ego attached.
2. **The agent can plan.** Given a clean statement of the problem it can offer a different approach. In a long chat it tends to keep tweaking the failing thing.

There is also a third benefit that shows up before you even start the new chat: writing the file forces the agent to state what it thinks is happening, and you often spot the wrong assumption in that summary yourself.

What this is **not** for: staying in a fresh chat on purpose. Starting a new chat throws away the prompt cache, so you pay full input price again for everything the agent had already read. That is fine once, to escape a loop, and wasteful as a habit.

## Other times to write one

- Before you stop for the night, so you can pick up tomorrow without rereading the chat
- Before switching models or tools, where a fresh start is worth the cache you lose
- Before asking for help in a forum or Discord thread; the STATUS file *is* the bug report
- When the agent has clearly lost the plot

## Keeping chat size down generally

- Work in one chat and let the cache carry the context. Do not open a new one to tidy up.
- Do not swap models partway through a task; it throws the cache away and buys nothing.
- Ask the agent to update `MODLOG.md` as it works. The log is your long-term memory, so the chat doesn't have to be.
- Keep the rules in `AGENTS.md` rather than retyping them every session.

See [`MODLOG-template.md`](MODLOG-template.md) for the running log, [`AGENTS-starter.md`](AGENTS-starter.md) for the rules file, [`BRIDGE-CONTRACT.md`](BRIDGE-CONTRACT.md) for who owns what, and [`PLAYTEST-report.md`](PLAYTEST-report.md) for recording what you tested.
