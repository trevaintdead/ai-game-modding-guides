# 7. FAQ

Quick answers to the questions people ask most. Where nobody has a confirmed answer yet, it says so.

## Getting started

**How do I get started?**
Read [Start here](00-start-here.md). The short version: set up an AI agent, install your games, open the agent in an empty folder, give it an example project, and say what you want.

**Is it really just "tell the AI to do it"?**
For the core idea, yes. Members who have made these projects say they link an example repo and ask for the same thing with their games. Expect to hit problems anyway, and expect to spend most of your time fixing them with the agent. See [guide 4](04-prompting-and-workflow.md).

**Do I need to know how to code?**
No, but it helps. You need to be clear about what you want and what's wrong, and you can ask the agent to explain anything. Members with no coding background have gotten projects working. A little knowledge helps you check what the AI is doing.

**I have never written code. Am I going to be stuck?**
Less than you think, if you accept the split: the agent writes it, you playtest it and describe what happened. That's the whole job. It's also why [guide 4](04-prompting-and-workflow.md) and [guide 5](05-testing-and-troubleshooting.md) are mostly about *communicating problems* rather than programming.

## Picking your games

**What do I put in the agent? Do I give it my whole game folder?**
You tell the agent where the games are installed and it finds what it needs. Members pointed their agents at their game installs, including a Minecraft install with Fabric. You don't upload anything.

**How do I know if my idea is possible?**
Check whether the host game has a mod loader or script extender. That's the main thing. See [guide 8](08-mod-loaders-and-script-extenders.md).

**Can I merge game X with game Y?**
Maybe. It depends mostly on whether the host game can run your code (script extender, mod loader, plugin system) and whether it's single-player. Projects exist in the shape of Elden Ring plus Spider-Man mechanics, and an Octane-style car in Minecraft, but nothing is guaranteed. Search for existing projects and tools for your games first.

**What games are easiest to start with?**
Terraria, Stardew Valley, Skyrim, Fallout 4, and Minecraft, by a wide margin. They have the best-documented loaders in gaming: tModLoader and SMAPI for the XNA games, SKSE and F4SE for the Bethesda ones, Fabric for Minecraft. That's also the direction most existing projects went, so there's code to read. Terraria and Stardew are worth knowing about if you assumed you needed Bethesda or Unity, because their games are .NET underneath and decompile to plain C#.

**What are the hardest?**
Games with no mod loader and no source. If the host game has nothing, you're reverse engineering an engine before you can start. See [guide 8](08-mod-loaders-and-script-extenders.md).

**Can I do this on Linux or macOS?**
Depends on the kind of project. A passthrough mod needs the host game running, and every example in these guides is Windows-only because the mod loaders are Windows tools. You'd be running the game under Wine or Proton and debugging it yourself. A Rust rewrite is different: the engine is your own code, so it builds for your OS, and IW4L documents Linux and macOS steps. Your own copy of the game still has to be readable from that OS though. See [guide 8](08-mod-loaders-and-script-extenders.md#windows-is-the-common-denominator).

**SkyCraft or universal-modder, which do I use?**
They do different jobs. SkyCraft is a working passthrough mod you read and adapt; [universal-modder](https://github.com/rehan-remade/universal-modder) is eleven agent skills plus a CLI that walks an agent through modding any game, including recon and reverse engineering. If you want Minecraft in Skyrim, use SkyCraft. If you're starting from a game nobody has touched, universal-modder is the better starting point.

**What if my game doesn't have a loader?**
You have more options than "give up" or "reverse engineer the whole thing". Work down this list and take the first one that reaches your idea: edit the game's data files directly, patch managed code with Harmony or Mixin, use native hooks on a C/C++ engine, or only then reimplement. Most ideas that look like they need a native hook turn out to be a data edit. [Guide 8](08-mod-loaders-and-script-extenders.md#which-route-is-cheapest) has the table.

**How do I move a character or asset from one game into the other?**
The usual answer is an extractor plus a converter, and you write your own code for it. Have the agent write an asset extractor so players can pull what they need from their own copies. Don't extract assets into your repo. See [guide 6](06-rules-legal-and-publishing.md).

**What about games I can't install from Steam, or console titles?**
Some projects need a disc-based copy extracted yourself, like Skate 3 for Xbox 360. That works for a game you own. What doesn't work is taking an ISO from a download site. See [guide 8](08-mod-loaders-and-script-extenders.md#disc-based-and-console-games).

**Are there games where this is impossible?**
Sometimes, for reasons other than loaders. Some games ship under publishing restrictions that limit what a port can use: Microsoft's Halo MCC has position and collision restrictions, while Xbox decomp projects exist under their own terms. Ask your agent to check the specific title's terms before you plan around it.

## Tools and cost

**What AI should I use?**
The most common pairing is Claude Code with a top Claude model, and Codex is a close second. See [guide 1](01-choose-and-set-up-an-ai-agent.md).

**Are ChatGPT or OpenAI models any good?**
They're very good and come with a decent allowance. Not recommended right now, because they give less capability and less usable usage per subscription than Claude does. If you already pay for one, there's no reason to cancel.

**Why does the AI say my game files are too large?**
You're probably using a chat website. You need an agent that runs on your PC and reads your files directly. You don't upload anything.

**Does the game have to be running while the AI works?**
The agent doesn't need the game running to read your files or write code. You do need the games running to playtest. For passthrough mods, both games run together when you test.

**What MCP servers do I need?**
None to start. Claude Code and Codex are agents, and MCP (Model Context Protocol) is a standard for plugging extra tools into one. None of the example projects list a server as a requirement.

Worth adding if you go on to reverse engineer: both [Ghidra](https://github.com/bethington/ghidra-mcp) and [IDA](https://github.com/HexRaysSA/ida-mcp) ship MCP servers, so the agent can decompile and rename functions itself instead of you pasting disassembly into a chat. The IDA one is official from Hex-Rays and installs with one command.

**Can I use a free plan?**
Not confirmed for a real project. Members expect to hit limits quickly. Free models inside OpenCode work for learning the workflow but get cut off and rate-limited. Pay-per-use API keys are another route. See [guide 11](11-models-and-cost.md).

**What's the best value?**
[OpenCode Go](https://opencode.ai/go) at $10, pointed at DeepSeek V4.1 Flash. OpenCode estimates roughly 26,000 requests per five-hour window on that model, which is their figure rather than something anyone's measured here. If you're buying one subscription instead of paying per token, Claude Pro at $20 beats anything cheaper.

**Will the $20 plan be enough?**
Yes for a project of this size. Expect roughly 3-4 hours of heavy use in each 5-hour window on the top model, so a weekend build fits comfortably. It depends on how much you do.

**Should I go straight to the $200 plan?**
No. Upgrade in order: $20, max it out, then $100, then $200. Two things worth knowing before you do: the 5x and 20x multiples apply to the five-hour session window rather than your weekly allowance, and a weekly cap sits on top either way. See [guide 11](11-models-and-cost.md).

**Do long chats burn my usage faster?**
Yes. The whole conversation is carried along on every turn, so a 300-turn chat costs more per turn than a fresh one. Start a fresh chat with a [`STATUS-handoff.md`](../templates/STATUS-handoff.md) file when things get long. See [guide 4](04-prompting-and-workflow.md).

**Can I run a local model on my own GPU?**
They don't work well for this, and 12 GB of VRAM isn't enough for a good local coding model. Worth trying if you're curious, but expect to fight it. [Guide 11](11-models-and-cost.md) has the current thinking.

**My GPU doesn't matter then, right?**
Correct, for cloud models. A 5090 changes nothing if you're using Claude or GPT, because the work happens on the provider's servers. It only matters if you're running a local model, which is the case above.

**How do I give Codex full access? It keeps failing.**
Check Codex's documentation for its permission and sandbox settings. Give it access to your project and game folders only. Full access to your whole PC is risky.

**Is giving an agent full PC access safe?**
Many members do it, but it's a real risk. Use a separate folder, Git, and backups. Consider a separate user account, a VM, or a container. See [guide 1](01-choose-and-set-up-an-ai-agent.md).

**Do I even need an IDE?**
No, but VS Code is worth installing so you get proper versioning and file views. The agent will create your files and run your builds either way. You don't copy code into folders by hand.

## Technical

**Do I need to decompile the games?**
Usually not. For a SkyCraft-style passthrough mod you need a way to run code inside the host game, not decompilation. For rewrites, check for existing format documentation and open-source readers first; decompiling is a last resort. [Guide 3](03-rust-rewrites-and-ports.md) covers when and how, including which tool to use, and [GameDecompLibrary](https://github.com/solarfren69420/GameDecompLibrary) is 304 existing projects to check before you start one. [Guide 17](17-decompile-system-map.md) is a reference for everything a decompile turns out to involve.

**Which tool is best for decompiling: IDA Pro, Ghidra, or Binary Ninja?**
Depends what the code is, which matters more than which decompiler you pick:

- **Managed .NET** (Terraria, Stardew, Celeste, most Unity games built on Mono): [ILSpy](https://github.com/icsharpcode/ilspy) decompiles to readable C#. `ilspycmd` gives you a whole project you can search.
- **Unity IL2CPP**: [Cpp2IL](https://github.com/SamboyCoding/Cpp2IL) on `GameAssembly.dll` plus `global-metadata.dat`. It recovers types, signatures and dummy DLLs for ILSpy. Two limits worth knowing: its analysis doesn't work for Unity 2020.2 or later, and it generates pseudocode rather than real IL.
- **Native C/C++**: Ghidra is free and the usual choice. IDA Pro is the commercial standard. Binary Ninja sits in between.

You can also ask your agent which tool fits your game's format and let it set it up.

**Can Unreal Engine games be decompiled?**
Usually, yes, and UE4SS exists as a modding and introspection tool, which gets you a long way without decompiling. Ask your agent to check for existing community tooling for your specific title, since support varies by title.

**Why Rust and Bevy? Why not C or C++?**
You don't have to use them. People reach for them because Rust catches memory mistakes before the game runs, setup is easy, Bevy is a free all-code engine, and AI is good at fixing Rust. C and C++ work fine too. The plugins that go inside Skyrim, Fallout 4, and GTA are still C++.

**Do I just download Rust and write my own stuff?**
For a rewrite you'd install Rust, but the agent writes the code. You also need the game installed and an agent set up. IW4L is the one to read: a Rust and Bevy runtime for Modern Warfare 2 that reads your own install. [Guide 12](12-worked-example-rust-rewrite.md) walks through how it was built, and [guide 3](03-rust-rewrites-and-ports.md) covers the rest.

**Can I make a completely new game instead of modding one?**
Yes, and it's a smaller job than a rewrite. Building a basketball game from scratch with open-source assets and animations is the same agent workflow, minus the part where it has to reverse-engineer anything.

**How do I stop the AI from testing visually?**
Tell it you'll playtest, and have it log numbers and events instead. See [guide 5](05-testing-and-troubleshooting.md).

**How do I make it run smoother?**
Give the agent frame-time logs from both processes and ask it to profile before changing anything. OWCraft's notes say skipping the presentation of the hidden window took Minecraft from 25 to 60 fps.

The gameplay game still has to run its client, though. It's hidden, not headless: it builds the meshes the host draws, or the picture the host pastes in, plus the hand and HUD (see [guide 9](09-worked-example-passthrough-mod.md#step-8-make-it-not-stutter)). Making it render nothing would break the visual premise. For other causes of stutter, see [guide 16](16-ownership-sync-and-rendering.md).

Other wins are sending deltas instead of full state, and fixing your update rate. See [guide 9](09-worked-example-passthrough-mod.md#step-8-make-it-not-stutter).

**The mod works but it's janky. Is that normal?**
Yes, at first. Members describe their projects as "jank as hell but working." Performance and polish come after it functions.

**How long does a passthrough mod take?**
The only figure worth having is about 3-4 hours of back-and-forth for an Elden Ring + Spider-Man mashup, described as jank but working. Treat that as one data point, not a typical runtime. A rewrite is a completely different scale.

## Rules and sharing

**Can I mod games with anti-cheat?**
The line is online versus offline, not whether anti-cheat is installed. Rocket League is the clearest case: Easy Anti-Cheat is required for online play and mods don't run while it's on, but turn it off through the game's own option and offline matches, training, LAN, and replays all work with mods.

Anything online is still out, you can get banned, and agents won't help you bypass it. They leave anti-cheat alone by default anyway. See [guide 6](06-rules-legal-and-publishing.md#online-play-and-anti-cheat).

**Can I put game files in my repo?**
No. See [guide 6](06-rules-legal-and-publishing.md).

**How do I publish on Steam Workshop?**
Make sure your upload has no copyrighted game content. Have the agent write an extractor that players run themselves. Check the platform's rules.

**Where can I see finished projects?**
In #share-your-projects on the Discord. A website to collect them is also in the works.

**How do I post my project?**
See [guide 10](10-posting-your-project.md), which has the pre-flight checklist and a post template.

**What do I write so people bother looking at it?**
A screenshot or GIF, a real commit history, an honest "what doesn't work" section, and a MODLOG. See [guide 10](10-posting-your-project.md#the-readme-is-the-post).

## Still unanswered

These are open, with no confirmed answer yet. If you know, post it on the Discord, or open a pull request and add it here.

- Which free model works best with OpenCode, and whether any of them can finish a real project
- Whether free plans from the big providers can complete a real project (see [guide 11](11-models-and-cost.md))
- How to decompile Unreal Engine games. UE4SS gets you a long way without decompiling, but that's not the same answer
- Whether detailed prompts or short loose prompts are more efficient. People disagree; see the debate section in [guide 4](04-prompting-and-workflow.md)
- Making games run better on original hardware (such as PS3), and whether emulator research applies
- Whether local models can handle a real project on 12 GB of VRAM. Probably not, but nobody has written up a proper attempt
- Whether a passthrough mod can be made to work on Linux or macOS at all
- Which games have publishing terms that block a port outright, beyond the Halo MCC restrictions
- Whether a passthrough mod works when the gameplay game only ships a console version, or only as a disc

---

<sub>[Spot a mistake? [Edit this page on GitHub](https://github.com/trevaintdead/ai-game-modding-guides/edit/main/guides/07-faq.md).](https://github.com/trevaintdead/ai-game-modding-guides/edit/main/guides/07-faq.md) &middot; [Open an issue](https://github.com/trevaintdead/ai-game-modding-guides/issues/new) &middot; Part of [AI Game Modding Guides](https://github.com/trevaintdead/ai-game-modding-guides)</sub>
