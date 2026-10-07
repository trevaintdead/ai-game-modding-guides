# 4. Prompting and Workflow

## There are no magic prompts

You tell the agent what you want, and it does it. People who have shipped these projects link an example repo, say "I want this for [my games]," and let the agent work.

They disagree about how much detail belongs in the first message. Both sides are below, so you can try each.

## Two schools of thought

**Short and loose.** A huge, perfectly written first prompt can be a trap. You dictate by voice, ramble a bit, and send a messy message. The reasoning: the agent knows the efficient path, and over-specifying can send it down the wrong one. A prompt running in a long session right now is roughly "there's currently still some stuff missing, right? ok let's add it."

**Detailed with context.** Others argue that more context produces a better result, and that good prompting saves time and usage. They care about efficiency, especially on plans with usage limits. Nobody has settled this, and it would make a good thing to test and write up.

One point from this side is worth taking seriously because it undercuts the "write a better first prompt" instinct: a prompt is a tiny fraction of a chat's context. What you say in turn one barely matters by turn fifty. That argues for fixing the chat, not rewriting the prompt.

### What both sides agree on

- **Be specific about the problem, not the implementation.** "This looks bad, fix it" gives the agent nothing. "The door doesn't open when I press E next to it, and the log says X" does.
- **Give the goal and the evidence.** Say what you wanted, what happened, and paste the logs.
- **Models tunnel-vision.** If the agent is stuck on the wrong approach, say so and point it somewhere else.
- **A good example project beats a long explanation.** Linking SkyCraft or IW4L does more than describing them.

## Working in small steps

Asking for the whole game at once usually goes badly. These habits help:

- Ask for one thing at a time and have the agent tell you how to run it and what you should see.
- Commit to Git after every working step. Version control lets you undo mistakes.
- Ask the agent to write a plan first if the task is big or you want more control.

## Keep the project's memory on paper

Agents forget between sessions. Files don't. Keep three small documents in your project:

1. **A rules file** (`AGENTS.md` or `CLAUDE.md`): what the agent must always do or never do. IW4L keeps one, named `AGENT.md`. See [`templates/AGENTS-starter.md`](../templates/AGENTS-starter.md).
2. **A development log** (`MODLOG.md`): what changed, how it was tested, what's still broken. OWCraft keeps one and links it from its README. See [`templates/MODLOG-template.md`](../templates/MODLOG-template.md).
3. **A design doc** (`docs/DESIGN.md`): how the project works, in plain language. SkyCraft's is the best example of this in the whole space.

Ask the agent to update them as it goes.

## The handoff trick for stuck chats

When a chat gets confused or very long:

1. Ask the agent to write the project's status and the exact problem it's stuck on into a detailed `.md` file.
2. Open a **fresh chat** and give it that file.
3. Ask it to read the file and suggest ways to troubleshoot.

This helps most with less capable models. Closing and reopening chats to clear old context works too. A template is in [`templates/STATUS-handoff.md`](../templates/STATUS-handoff.md).

## Saving usage

- Long chats carry all their history, so they use more of your limit. Starting fresh with a handoff file helps.
- Close and reopen chats when you switch topics.
Ask the agent, early on, to set up a documentation standard and to keep token efficiency in mind without losing functionality.
- Don't treat any of this as a rule. Plans and models change.

## How long things take

It depends on the model, how hard it thinks, and how big the task is. Anything from about 5 minutes to many hours is normal for a single piece of work.

## When you hand over to a fresh chat

The handoff trick above solves a stuck chat. There are two other moments where it pays to start clean:

- **Starting a new project.** Nothing useful carries over between unrelated projects, and a long chat full of one game's details makes the agent reach for them in the next one. `STATUS.md` per project beats one continuous conversation.
- **Switching agents or models.** Different tools read the same files differently. A handoff file gives the new one the same starting point.

## A worked starter prompt

Every example project here uses the same shape. Point at a reference, name the substitution, and ask for a read-only report first:

```
I want to build [project type] like [reference project] ([link]), but for [your games].

Clone it locally and read its README and any docs/ files so you understand the
architecture. I want the same approach.

[Game A] is installed at [path]. [Game B] is installed at [path].

Before you build anything, tell me:
- what loaders, APIs or SDKs exist for these games
- does either have online play or anti-cheat? (We don't touch those.)
- what's the smallest thing I can build first to prove this works

Don't change any code yet. Just report what you found.
```

The last line is the one that matters. It costs you one turn and saves you from a confident plan built on a wrong assumption.

## Let the agent test what it can, and you test the rest

See [guide 5](05-testing-and-troubleshooting.md). The short version: the agent is bad at judging visuals and "feel." Make it log numbers and events, and you do the playtesting.

---

## The prompting debate

**This section is open. It's meant to be argued with.**

This is genuinely unsettled, and naming that is more useful than picking a side. Here are the arguments in full. Read both, try both, and post your results on the Discord.

> **One member:** "guys theres no tricks or special prompts, you literally just tell the ai to do stuff and itll do it. Thats all i do"
>
> "if you write some big detailed prompt exactly how you want it, then its gonna be worse than letting the ai wing it. The ai knows the best and most efficient path to the outcome you want."
>
> "the more specific the worse by far my man"
>
> "I dont get what you mean man, better prompting method? How would that work? You realise the prompt is like 0.1% of the context of a chat"

> **The same member**, later, citing Andrej Karpathy: *"One pattern I find useful for working with LLMs is a nice long ramble session. Sometimes the LLM needs more bits to understand what you're trying to achieve, but you're too lazy to type them."* ([source](https://x.com/karpathy/status/2079610838143623371))

> **Another member:** "These are literally inference machines they require context. The more context you provide the better."
>
> "The more specific and accurate your prompt is, the better the weights will be set. The faster and more efficient the model is at doing the asked task."

> **Another:** "AI tunnel visions on implementations a lot."
>
> "In this context I agree, in a context of a professional its the opposite."

> **Another:** "ive found more specific prompts can cause tunnel vision on the wrong things, well in some cases."
>
> "my current prompt running right now is 'there's currently still some stuff missing right? ok lets add it'"

> **Another:** "i'd start asking it inside your ide to create a documentation standard. tell it that youre concerned with token efficiency, but you don't want to sacrifice functionality."

### A note on these quotes

Names are removed on purpose. These are real people's Discord messages, and quoting them publicly without asking isn't worth the convenience.

They also aren't verbatim: Discord's own capitalisation has been tidied up in a couple of places. The wording is otherwise unchanged, but don't treat them as transcripts.

The Karpathy quote is second hand. It was pasted into chat and the link matches the text that was pasted, but nobody has read the original post.

### What the disagreement is actually about

Read side by side, those quotes contain two separate arguments:

**1. Does over-specifying pick the wrong approach?** The short-and-loose side says yes. Dictate the implementation and the agent commits to it, because models tunnel-vision on the first idea they form. The context side says to describe the *goal* precisely and let the agent choose the route. Precision about the outcome isn't the same as dictating the method.

**2. Does it save time?** The context side's strongest argument is usage limits. If a vague prompt makes the agent wander and you burn your 5-hour window on dead ends, "efficient" wins, even if the vague prompt would have got there eventually. The 0.1% point cuts against the whole debate: prompt length is probably not the thing worth optimising.

### Where this lands

Not a verdict. Something you can try either way:

- **Be precise about the goal and the evidence.** Both sides agree on this, and it's the thing that actually matters.
- **Be loose about the implementation.** The main documented failure mode is tunnel-vision, so pointing at a method rather than a result carries real risk.
- **When you're stuck, change the context instead of rewriting the prompt.** Start a fresh chat with a [`STATUS-handoff.md`](../templates/STATUS-handoff.md) file. That works regardless of which school you're in.
- **Long-running projects need both.** The loose approach suits a two-hour project. Once you're 200 commits in, the rules file and the log matter more than any individual prompt.

### Try it and tell us

A controlled comparison would be welcome here. Log the same task twice with a loose prompt and a detailed one, and record:

- how many turns it took
- whether it reached a working result
- how much of your usage limit it consumed
- what it got wrong

Post it on the Discord and the best ones get folded into this page. A GitHub issue works too, if you'd rather have it written down somewhere permanent.

### A few things that came up in the thread

- **Voice dictation.** Several people dictate instead of typing, which gets you a long rambling prompt for free and removes the temptation to over-edit it.
- **Ink-and-paper is cheaper than you think.** A prompt is a small part of a chat's cost, but a 200-turn chat is not. Starting fresh and handing over a file is the real saving.
- **Let it test, but stop it from looking.** It will try to visually verify the game, which it cannot do. See [guide 5](05-testing-and-troubleshooting.md).
- **Explain your constraints, not your implementation.** "It needs to work on a 2015 laptop" is context. "Use a thread pool" is an instruction.

---

<sub>[Spot a mistake? [Edit this page on GitHub](https://github.com/trevaintdead/ai-game-modding-guides/edit/main/guides/04-prompting-and-workflow.md).](https://github.com/trevaintdead/ai-game-modding-guides/edit/main/guides/04-prompting-and-workflow.md) &middot; [Open an issue](https://github.com/trevaintdead/ai-game-modding-guides/issues/new) &middot; Part of [AI Game Modding Guides](https://github.com/trevaintdead/ai-game-modding-guides)</sub>
