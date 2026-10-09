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
- If you want more control on a big task, ask for the plan as part of the conversation rather than as a gate before any work happens.

## Keep the project's memory on paper

Agents forget between sessions. Files don't. Keep three small documents in your project:

1. **A rules file** (`AGENTS.md` or `CLAUDE.md`): what the agent must always do or never do. IW4L keeps one, named `AGENT.md`. See [`templates/AGENTS-starter.md`](../templates/AGENTS-starter.md).
2. **A development log** (`MODLOG.md`): what changed, how it was tested, what's still broken. OWCraft keeps one and links it from its README. See [`templates/MODLOG-template.md`](../templates/MODLOG-template.md).
3. **A design doc** (`docs/DESIGN.md`): how the project works, in plain language. SkyCraft's is the best example of this in the whole space.

Ask the agent to update them as it goes.

## Stick to one chat

Work in a single chat for the length of a task. This is the cheapest way to work and it is what the people getting results actually do.

The reason is caching. Everything you have already established, the files the agent has read, the format notes it worked out, sits in a cache that bills at about a tenth of the normal input price. Stay in the same chat and that context keeps paying the cheap rate. Start a new one and the agent is back to paying full price for all of it, then has to rediscover what it already knew.

So the practical rules:

- **One chat per task.** Do not open a second chat to "start clean" or because the last one feels messy. Messy is fine. The agent knows more than you think it does.
- **Pick a model and stay on it.** Swapping models mid-task throws away the cache and burns a chunk of your allowance for nothing.
- **Keep typing in the same chat.** An idle chat can fall out of cache, so if you are stepping away for a while, that is a good moment to think about what you actually want next.

## When a chat really is stuck

Some chats do get wedged: the agent loops on the same failing idea, or it has committed to an approach that is not working and keeps defending it. In that case a handoff file is worth the fresh start, because you are escaping a bad loop rather than routine housekeeping.

1. Ask the agent to write the project's status and the exact problem it's stuck on into a detailed `.md` file.
2. Open a **fresh chat** and give it that file.
3. Ask it to read the file and suggest ways to troubleshoot.

A template is in [`templates/STATUS-handoff.md`](../templates/STATUS-handoff.md). The same file earns its keep for a different reason: it forces the agent to write down what it thinks is happening, and you often spot the wrong assumption in that summary before the agent does.

## Saving usage

- Stay in one chat. Cache does the work, and it only works if you stay put.
- Do not swap models partway through a task.
- Ask the agent, early on, to set up a documentation standard so it stops rediscovering your project on every session.
- Don't treat any of this as a rule. Plans and models change.

## How long things take

It depends on the model, how hard it thinks, and how big the task is. Anything from about 5 minutes to many hours is normal for a single piece of work.

## When a new chat is still the right call

Two cases, and only two:

- **Starting a genuinely different project.** Nothing useful carries over, and a chat full of one game's details makes the agent reach for them in the next one. Keep a `STATUS.md` per project so each one starts from a known state.
- **Switching agents or models.** Different tools read the same files differently, so a handoff file gives the new one the same starting point. Expect to pay full input price again for the context, so do it at a natural break rather than mid-problem.

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

Start there, then build it. Don't ask me again before the first one works.
```

Holding off on the first turn is the one part worth the wait. It costs you one turn and saves you from a confident plan built on a wrong assumption, which is the common way beginners lose a whole evening. After that, get on with it.

## Let the agent test what it can, and you test the rest

See [guide 5](05-testing-and-troubleshooting.md). The short version: the agent is bad at judging visuals and "feel." Make it log numbers and events, and you do the playtesting.

---

## The prompting debate

Both camps are quoted below because the disagreement is real. The short-and-loose side is the one people who ship projects land on, and it is what this guide recommends.

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

The loose side wins, for a specific reason rather than on taste. The models are trained to be given an instruction in ordinary language and left to work out the implementation. Pointing at a method instead of a result sends them down the first path that occurs to them, and they will then defend it. You wanted the outcome; you specified the route.

So:

- **Be precise about the goal and the evidence.** This is the part that matters and both camps agree on it. Say what you wanted, what happened, paste the logs.
- **Be loose about the implementation.** Do not dictate the method. The main documented failure is tunnel-vision on a specified approach.
- **Write it the way you would say it.** Spelling, grammar and capitals do not matter. These models are trained on ordinary writing with errors in it, so a messy dictated message lands exactly as well as a polished one, and a mangled prompt will still be understood. Do not spend time fixing your prose.
- **Link an example rather than describing one.** Pointing at SkyCraft or IW4L does more than any explanation you could write.
- **When you are genuinely looping, change the context instead of the prompt.** A [`STATUS-handoff.md`](../templates/STATUS-handoff.md) file in a fresh chat breaks a stuck run. Save it for that, not for routine tidying.
- **Long-running projects need notes more than they need prompt craft.** Once you are 200 commits in, the rules file and the log matter more than any individual prompt.

There is no system to learn. The elaborate methods that circulate, the orchestration layers and the prompt templates, are mostly ways of feeling in control. Type what you want and let it work.

If you want to settle it your own way, log the same task twice with a loose prompt and a detailed one, and record how many turns each took, whether it worked, what it consumed and what it got wrong. Post it on the Discord and the best ones get folded into this page.

### A few things that came up in the thread

- **Voice dictation.** Several people dictate instead of typing, which gets you a long rambling prompt for free and removes the temptation to over-edit it.
- **Ink-and-paper is cheaper than you think.** The prompt itself is a tiny part of what a turn costs. The expensive part is re-reading everything you already established, which is why staying in one chat with a warm cache beats starting fresh. See [Stick to one chat](#stick-to-one-chat).
- **Let it test, but stop it from looking.** It will try to visually verify the game, which it cannot do. See [guide 5](05-testing-and-troubleshooting.md).
- **Explain your constraints, not your implementation.** "It needs to work on a 2015 laptop" is context. "Use a thread pool" is an instruction.

---

<sub>[Spot a mistake? [Edit this page on GitHub](https://github.com/trevaintdead/ai-game-modding-guides/edit/main/guides/04-prompting-and-workflow.md).](https://github.com/trevaintdead/ai-game-modding-guides/edit/main/guides/04-prompting-and-workflow.md) &middot; [Open an issue](https://github.com/trevaintdead/ai-game-modding-guides/issues/new) &middot; Part of [AI Game Modding Guides](https://github.com/trevaintdead/ai-game-modding-guides)</sub>
