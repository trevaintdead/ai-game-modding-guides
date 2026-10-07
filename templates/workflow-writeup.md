# Workflow Write-Up Template

Use this when you're sharing a finished project. It is the thing this community is short of: almost nobody documents *how* they got there.

A feature list proves the thing works. A workflow write-up is what lets someone else do it too. Write one even if your project is small and imperfect. A rough honest one beats a polished marketing page.

Copy to your repo as `WORKFLOW.md`, or post it as a thread in #share-your-projects.

---

## Why this exists

The commonest complaint about AI-assisted projects is that people say "just tell the AI to do it" and stop there. That is most of the method, and the useful part is everything around it:

- which games you picked and why
- what you checked *before* starting
- which dead ends cost you the most time
- what worked

None of that fits in a README.

## Template

```markdown
# How I made [Project Name]

## TL;DR

[3-6 bullets. What it is, what stack, roughly how long it took, and the one
thing you'd tell someone starting this.]

## The result

- **Games:** [Game A] v[version] + [Game B] v[version]
- **Stack:** [SKSE C++ plugin / Fabric Java mod / Rust + Bevy]
- **Built with:** [agent and model]
- **Time:** [hours, roughly]
- **Cost:** [plan tier, roughly]
- **OS:** [Windows, because that's what it needed]

[Screenshot or GIF]

## What works

- [Feature]
- [Feature]

## What doesn't work

- [Honest list. This section is the most valuable part of the document.]

## Before you start: what to check

The research I did that saved me time. Be specific enough to act on.

1. **Does Game A have a mod loader?** [Which one, which version, where from.]
   Without this there's no easy path; you'd be reverse engineering.
2. **Existing projects I read first:** [links + one line on what each taught you]
3. **Existing mods for Game A:** [links]
4. **File formats / docs I found:** [links]
5. **Version traps:** [what I had to downgrade or match exactly]

## The plan

[What you built first, second, third, and why that order. Show the milestones
you aimed at. This is the part people can copy.]

1. [Milestone 1: e.g. "plugin loads and writes a log line"]
2. [Milestone 2: e.g. "position crosses from B to A"]
3. [Milestone 3: e.g. "B's objects spawn in A's world"]
4. ...

## Which game owns the player

[Say which side is authoritative for player position and physics, and why. This
is the decision everyone gets wrong, and reversing it later means rewriting
both halves. SkyCraft made Minecraft authoritative for the player even though
Skycraft is the game you're looking at.]

## Transport and architecture

[How the two halves talk. Shared memory, socket, file, IPC? What messages go
across and at what rate? Keep this concrete; it's the part people copy. If the
gameplay game still renders offscreen, say so here rather than claiming it runs
headless.]

## What I prompted, roughly

[Actual prompts you used, with the successful ones intact and the bad ones too.
Do not clean these up. Show that the first prompt didn't work.]

**This one worked:**
```
[prompt]
```

**This one was a waste:**
```
[prompt]
```
Why it was a waste: [explanation]

## Dead ends

The most valuable section. What didn't work, and what it cost.

- **[Approach]:** [why it failed]. Cost: [time / tokens / a broken build]
- **[Approach]:** [why it failed]

## Problems I hit, and what fixed them

| Symptom | Cause | Fix |
|---------|-------|-----|
| [error or behaviour] | [root cause] | [what you changed] |

## What I'd do differently

[Honest retrospective. This is what makes the document trustworthy and what
someone else will thank you for.]

## Credits

- [Project]: [what you reused, license]
- [Person]: [help you got]
- [Modding community for Game A]: [loader / docs]

## Legal

Unofficial fan project, not affiliated with or endorsed by the publisher.
No game assets included; players supply their own copies. Built with AI
coding agents.
```

## Good examples of sections people actually find useful

**The dead-ends section.** Specific, costly, and impossible to get anywhere else. "Tried a file-based transport first, spent two hours on Windows file locking, switched to shared memory" saves the next person two hours.

**Real prompts, unedited.** Especially the failures. It shows the method is iterative, which is the honest truth and also the reassurance a beginner needs.

**The "what I'd do differently".** Signals experience, and turns a flex into a lesson.

**Version traps.** Ultra-specific and universally useful. Which version you needed, and which downloader got you there.

## What not to include

- Game assets, screenshots of copyrighted content beyond fair use, or decompiled code
- Long transcripts of the whole session. Excerpts.
- Advertising. Post it there and it gets removed.
- Anything from a game you're not allowed to mod. See [guide 6](../guides/06-rules-legal-and-publishing.md).
- ISOs or dumps you didn't make yourself, even if you own the game on disc

## A note on unfinished work

#share-your-projects takes work-in-progress posts as well as finished ones. A half-working project with an honest "what doesn't work" section is more useful than nothing, and it's how people find collaborators. Say what you've got and what's broken.

Only post if you made it. No assets, no leaked material, and a repo link rather than a direct download.
