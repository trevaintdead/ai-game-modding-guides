# 0. Start Here

You have never done this before and want to know what's involved. This page is the map. The other guides fill in the details.

Stuck on something specific to your setup? Ask on the Discord.

## What you do

The agent writes the code. You tell it what you want, it writes and builds, and you test the result by playing. Your job:

- decide what you want
- describe it and describe problems clearly
- playtest, because the agent can't see or feel a game
- keep the project organized so you can recover when something goes wrong

The core of it is simple: install the games, open an agent, give it an example project, and say what you want. In practice that turns into a long series of problems to fix with the agent. Expect that.

## Pick your path

| I want to... | Read | Rough idea |
|--------------|------|------------|
| Put one game's gameplay inside another (Minecraft in Skyrim, Skate 3 in GTA) | [Passthrough mods](02-passthrough-mods.md) | Both games run at once and talk to each other |
| Rebuild a game's engine so it runs on its own | [Rust rewrites](03-rust-rewrites-and-ports.md) | Bigger job. Reads your game files at runtime |
| Play what others made | #share-your-projects on the Discord | Use the project's own install instructions |

Start with a passthrough mod if you're unsure. You see something working sooner. If you can't tell which kind of project your idea is, [guide 14](14-choosing-a-route.md) sorts the seven common routes.

### Three pages worth reading first

These answer the questions people ask most:

1. **[Which loaders and script extenders exist](08-mod-loaders-and-script-extenders.md)**: decides whether your game idea is even realistic. This is the question people ask most.
2. **[A full passthrough walkthrough](09-worked-example-passthrough-mod.md)**: the whole process end to end, with the prompts.
3. **[Posting your project](10-posting-your-project.md)**: what a finished project needs before others can use it.

## The steps, in order

1. **Pick your games.** Check that they're single-player or offline, and that you own them.
2. **Search for existing work first.** Look for mod loaders, existing mods, or decomp projects for your games. Skipping this is the most common way to waste an evening.
3. **Set up an AI agent** on your PC. See [guide 1](01-choose-and-set-up-an-ai-agent.md). Costs and model choice are in [guide 11](11-models-and-cost.md).
4. **Install the games** and make sure they run normally.
5. **Open the agent in a new, empty project folder** and use a starter prompt from guide 2 or 3.
6. **Playtest and report back.** Describe what happened, paste logs.
7. **Keep notes** so a fresh chat can pick up where the old one left off. See [guide 4](04-prompting-and-workflow.md).
8. **Share it** as a GitHub repo, with no game files in it. See [guide 6](06-rules-legal-and-publishing.md) and [guide 10](10-posting-your-project.md).

## Honest expectations

- **Time:** a task can take anywhere from a few minutes to many hours, depending on the model and how hard you make it think. An Elden Ring plus Spider-Man mashup took about 3-4 hours of back-and-forth to get working, and the result was janky but functional. Treat that as one data point rather than a typical runtime.
- **Cost:** agents use paid plans or API credits, and plans have usage limits. See [guide 11](11-models-and-cost.md) for what to actually spend.
- **Coding knowledge:** you don't need it to start, but a little helps. You can always ask the agent to explain what it did.
- **Rough edges:** early projects are experimental. Back up your saves.
- **Some things won't work.** Online games with anti-cheat are off the table. See [guide 6](06-rules-legal-and-publishing.md).

## If you only do one thing first

Don't start with the game you wish you could mod. Start with a game you already own that has a good loader, and get something small working end to end. That teaches you the loop: recon, one working slice, playtest, commit, repeat.

A finished thing that does almost nothing teaches you more than an ambitious thing you abandon on day two.

## Before you start: checklist

- [ ] I own the games and they're installed
- [ ] They are single-player or offline
- [ ] The host game has a [mod loader or script extender](08-mod-loaders-and-script-extenders.md)
- [ ] I've picked an AI agent rather than a chat website
- [ ] I created a folder for this project
- [ ] I know I'll playtest myself
- [ ] I've read the rules in [guide 6](06-rules-legal-and-publishing.md)

---

<sub>[Spot a mistake? [Edit this page on GitHub](https://github.com/trevaintdead/ai-game-modding-guides/edit/main/guides/00-start-here.md).](https://github.com/trevaintdead/ai-game-modding-guides/edit/main/guides/00-start-here.md) &middot; [Open an issue](https://github.com/trevaintdead/ai-game-modding-guides/issues/new) &middot; Part of [AI Game Modding Guides](https://github.com/trevaintdead/ai-game-modding-guides)</sub>
