# 3. Rust Rewrites and Ports

A rewrite or port rebuilds a game's engine from scratch, so it runs on its own instead of inside the original. The new engine reads models, textures, maps, and sounds from **your own installed copy** of the game at runtime. The repo contains only your code.

## Examples to study

| Project | What it shows |
|---------|---------------|
| [IW4L](https://github.com/vladtrc/iw4L) | A Call of Duty: Modern Warfare 2 (2009) runtime in Rust and Bevy. Experimental: gameplay is incomplete, and it says so. Reads your own install in place and ships no assets |
| [gang-beasts-rust](https://github.com/muffinmxn/gang-beasts-rust) | Python tools extract your game's data into formats a Rust/Bevy engine loads. A whitelist `.gitignore` keeps extracted files out of the repo |
| [benilla](https://github.com/samwhosung/benilla) | A WoW 1.12.1 client in Rust and Bevy. A big project with hundreds of commits, readers for the game's file formats, and a generated map of the code |
| [2010 Rust Rewrite Mashup](https://github.com/chasmlol/2010-rust-rewrite-mashup) | A rewrite combined with other games |

The good ones share some habits: a clear "what works / what's missing" list, no game files, credits, and an `AGENTS.md` or development log so the AI's work can be followed.

## Why Rust and Bevy?

You don't have to use them. C and C++ work fine. People pick Rust and Bevy because:

- Rust catches memory mistakes before the game runs, so you get fewer random crashes
- it's easy to set up
- Bevy is a free engine that's all code, with no editor to learn
- AI is good at fixing Rust, because the compiler's error messages say what's wrong

Not every project uses Bevy. It is the most common choice in this space, but you'd pick a different one if you wanted to. IW4L also uses wgpu for rendering on top of Bevy, and translates the original game's Direct3D 9 shader bytecode to WGSL.

**Bevy's performance problems are the main reason to consider leaving it.** Projects built on it often hit frame rate trouble that is genuinely hard to fix, and chasing it can eat the project. The usual cause is putting too much in the ECS: if you shove every entity and per-frame query through Bevy's system graph, you pay for it every tick.

The escape is that Bevy is really two things, an ECS and a renderer. Projects that outgrow it keep `bevy_ecs`, which is where your game logic lives, and swap the rendering and app shell for something else. That is a contained refactor rather than a rewrite, because the entity code does not change. Worth knowing before you commit to it, so you find out which situation you are in.

Rust is also common in the loaders themselves. **[me3](https://github.com/garyttierney/me3)**, the successor to Mod Engine 2, is a Rust framework covering Elden Ring, Dark Souls III, Sekiro, Armored Core VI and Elden Ring Nightreign, and its workspace is a readable set of small crates: a launcher, an IPC layer, a mod host and a mod protocol. It is not a rewrite of the game, but the way it is put together is a good model for the kind of tooling this guide keeps pointing at. [Guide 8](08-mod-loaders-and-script-extenders.md) covers it as a loader, along with one restriction you need to know about before you plan around it.

## Be realistic about size

A rewrite is a big job. IW4L is around 160 commits in and still describes itself as experimental, with missing behaviour, bugs and desyncs. Benilla is described as complete, with hundreds of commits behind it. Start with a goal that fits in a sentence, like "load and show the first level and walk around in it." Grow from there.

A full game with real depth is a multi-month project for one person working with an agent, not a weekend. Scope it as though you are building a small game that happens to reuse an original's assets, because that is what it is.

If you are starting from a decompilation, think hard about whether you want one. The point of matching the original exactly is to recover its behaviour, but if your actual goal is a better version of the game rather than a faithful copy, most projects are better off rewriting from the start and keeping only the data files. Faithful decompilation is a multi-month effort that needs a group. A rewrite from your own format readers is a multi-month effort you can finish alone.

## How these projects are usually built

1. **Extract.** Tools (often Python) read the player's own install and convert models, textures, and maps into formats the engine can load. Some projects read the original formats directly at runtime instead.
2. **Engine.** A Rust engine (Bevy is common) plus a physics library draws and simulates the world.
3. **Rebuild the rules.** Movement first, then maps, then weapons and interactions, then everything else.
4. **Compare with the real game.** Play the original next to your build and note differences.
5. **Write down what works and what's missing.**

If you have a working game and want to link a second one to it, you want [guide 2](02-passthrough-mods.md). Much smaller job.

## Do I need to decompile?

**Usually not, and check before you assume you do.** In order of preference:

1. **The file formats are already documented.** Lots of games have community specs, and some have open-source readers already written. If one exists, use it. This is the free path.
2. **The game has a source release.** Some studios shipped their engines or games as source, legally and publicly. Check.
3. **You need the executable's logic.** Only then is decompiling on the table.

People routinely skip steps 1 and 2 and lose whole evenings to it. The rule worth following is simple: *look for an existing decomp or format project before you start.* Ask the agent to search first. It's good at finding community projects.

Two places worth searching by hand, because the agent will not know about either:

- **[GameDecompLibrary](https://github.com/solarfren69420/GameDecompLibrary)** is a catalog of 304 decompilation projects, tools and source releases, each entry tagged with its method and each figure sourced. Check it before you plan anything.
- **[Guide 17](17-decompile-system-map.md)** covers everything a decompile turns out to involve, which is worth reading before you estimate the work.

Step 1 is worth preferring even when step 3 would work, for a reason that has nothing to do with effort. **Studying a game through its interface involves no copying at all**, so there is no copyright question to answer. The moment you run a decompiler, a copy of protected expression exists on your disk, and you have moved from a clean position into one that depends on a fair use argument. Prefer observation wherever the question allows it. [Guide 13](13-reverse-engineering-and-the-law.md#black-box-grey-box-white-box) covers the distinction and the cases behind it.

### When you do need it

Tools, in the order worth trying:

| Tool | Cost | Notes |
|------|------|-------|
| **Ghidra** | Free, open source | The one people use. Needs a Java runtime, and a processor module for some older consoles. Has an [MCP server](https://github.com/bethington/ghidra-mcp) |
| **IDA Pro** | Commercial, expensive | The industry standard, with an [official MCP server](https://github.com/HexRaysSA/ida-mcp) from Hex-Rays. Free tier is limited |
| **Binary Ninja** | Commercial, cheaper than IDA | Worth knowing about, though there is no worked example here using it |

**The bigger question is what the code is.** Picking the wrong tool for the language wastes days, so check this before installing anything:

| What the game uses | What to reach for |
|---|---|
| Managed .NET (Terraria, Stardew, Celeste, most Unity on Mono) | [ILSpy](https://github.com/icsharpcode/ilspy). Decompiles to readable C#, and `ilspycmd` emits a whole project you can search |
| Unity IL2CPP | [Cpp2IL](https://github.com/SamboyCoding/Cpp2IL) against `GameAssembly.dll` plus `global-metadata.dat`, then Ghidra for the method bodies, which are native |
| Java | Vineflower, CFR or Recaf. For Minecraft, use Loom's `genSources` with Mojang mappings |
| Native C/C++ | Ghidra or IDA, driven through an MCP server so the agent can decompile and rename functions itself |
| Retail ROMs and disc images | [N64Recomp](https://github.com/N64Recomp/N64Recomp) recompiles N64 games to native executables instead of emulating them. Other consoles have similar tools |
| Live memory | Cheat Engine for a value scan, x64dbg for breakpoints, Frida to hook functions |
| Rendering | RenderDoc to capture a frame and see every draw call and render target |
| Data and asset files | The community tool first. Unity: UABEA or AssetRipper. Unreal: FModel or UAssetGUI. Bethesda: xEdit. GameMaker: UndertaleModTool |

Two limits on Cpp2IL worth knowing before you commit an afternoon: its analysis does not work for games targeting Unity 2020.2 or later, and it produces pseudocode and textual analysis rather than real IL.

**Read the real thing, don't guess.** Decompiler output, the actual data file, a memory read or a GPU capture is the specification. Write down what you learn as you go, with the names, IDs, offsets and formats, because you will need it again in an hour.

Say this to the agent explicitly. Models will cheerfully invent a plausible struct layout or a made-up offset when they are unsure, and the result compiles, runs, and is wrong in a way that takes hours to find. A prompt that says *match the original, and read the real files rather than guessing* cuts down on that noticeably.

Driving a decompiler through an MCP server is what changes the workflow. Without one you paste disassembly into a chat and paste it back. With one the agent reads the decompiler directly, so ask it to find a function or rename everything it understands.

Asking the agent to decompile a folder "using the correct tools" is a realistic request: it identified the platform and format, installed Ghidra with the right processor module, and ran the process. That is roughly the workflow you are aiming for.

### A realistic prompt

```
I want to understand how [Game] stores its [maps / models / animation data].

Before installing anything: research whether there is existing community
documentation, an open-source library, or a decomp project for this game's
file formats. Tell me what you found first.

If nothing exists and the executable is the only option, use Ghidra.
Work in [gitignored folder]. Do not write anything into this repo except
a notes file describing what you learned.
```

### Rules for the output

- **Everything stays on your machine.** Never commit decompiled code, Ghidra databases, or extracted assets. See [guide 6](06-rules-legal-and-publishing.md).
- **Use a whitelist `.gitignore` from day one** so an extracted file can never be committed by accident.
- **Write down what you learned as documentation**, not as code. That documentation is the shareable part. This is how open-source engine reimplementations like [OpenMW](https://github.com/OpenMW/openmw) and [OpenRCT2](https://github.com/OpenRCT2/OpenRCT2) exist.
- **Single-player, offline games you own only.** Leave DRM alone. Leave anti-cheat alone. Don't target anything to get around access controls. See [guide 6](06-rules-legal-and-publishing.md).
- **Don't redistribute the output.** Personal study of a game you own is the scope. Publishing extracted assets or decompiled source is not.
- **Keep your research local.** IW4L used Ghidra to inspect the original binaries and records what it learned in `docs/provenance/`, with the dumps and databases themselves kept out of the repo.

Read [guide 6](06-rules-legal-and-publishing.md) before going down this path. It's not legal advice, but it lists what the community's own tooling refuses to do.

## Step by step

1. Pick a game and a one-sentence goal.
2. Look for existing research, tools, and decomp projects.
3. Open your agent in a new project folder with Rust installed (rustup.rs). On Windows you'll also need the Visual Studio C++ build tools.
4. Send the starter prompt.
5. Build in small steps, playtesting each one.
6. Keep a log of what works and what doesn't. Update the README's "what works / what's missing" list.
7. Share it as a GitHub repo with no game files. See [guide 10](10-posting-your-project.md).

## Starter prompt

```
I want to build a Rust rewrite of [Game] that reads its data from my own installed copy at [path] at runtime. Match the original's actual behaviour, and where you are unsure, read the real files and decompiled output rather than guessing. Use [IW4L / gang-beasts-rust / benilla] as a reference for structure: [links].

Rules: never copy game assets or decompiled code into the repo. Use a whitelist .gitignore. Credit anything you learn from and keep licenses.

First, look for existing documentation, file format specs, and decomp projects for this game, and tell me what's out there. Then start on the smallest thing that proves the data loads. Don't ask me again before that works.
```

## Passthrough or rewrite?

| | Passthrough | Rewrite |
|--|-------------|---------|
| Goal | Mix two games' gameplay | A standalone engine you control |
| Needs | Both games running together | Only your game files |
| Size | Often smaller | Often much bigger |
| Good first project? | Usually | Only with a small goal |

---

<sub>[Spot a mistake? [Edit this page on GitHub](https://github.com/trevaintdead/ai-game-modding-guides/edit/main/guides/03-rust-rewrites-and-ports.md).](https://github.com/trevaintdead/ai-game-modding-guides/edit/main/guides/03-rust-rewrites-and-ports.md) &middot; [Open an issue](https://github.com/trevaintdead/ai-game-modding-guides/issues/new) &middot; Part of [AI Game Modding Guides](https://github.com/trevaintdead/ai-game-modding-guides)</sub>
