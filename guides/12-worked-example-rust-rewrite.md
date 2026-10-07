# 12. Worked example: IW4L, an AI-assisted Rust rewrite

This case study is about [IW4L](https://github.com/vladtrc/iw4L), a standalone Rust runtime for Call of Duty: Modern Warfare 2 (2009). Around 160 commits at the time of writing, roughly 800 stars, Apache-2.0, and actively developed.

It is unfinished, and says so: *"Gameplay remains incomplete; expect missing behavior, bugs and desyncs."* It ships no game assets. You point it at a copy of MW2 that you already own and it reads that installation's maps, models, textures and weapons into its own engine.

What makes it worth reading is not the engine. It is how a project at this scale handles provenance, licensing, and the line between what an agent wrote and what a person decided.

## Why IW4L and not something else

This page used to cover [mw2-rust-rust-rewrite](https://github.com/Dj-Shortcut/mw2-rust-rust-rewrite), a standalone Rust/Bevy project by Dj-Shortcut that builds on IW4L. That was a fair case study and most of what it got right is still true here, but IW4L is a better example for beginners for three reasons.

It is finished enough to read. Its own `docs/` covers rendering, simulation, map loading, the GSC runtime, bot AI and performance, plus a reproducible test suite. You can follow what a real project does instead of inferring it.

It documents its own provenance. The movement note linked below exists because a contributor raised a licensing question and the maintainer wrote it down. Most projects at this scale never do that.

It states its limits plainly. *"Gameplay remains incomplete; expect missing behavior, bugs and desyncs."* A case study should teach you what a real project looks like mid-flight, not what one looks like in a promo screenshot.

If you came here for the old page: the parts about reverse engineering being part of the history, about checking a dependency's licence rather than assuming it, and about players supplying their own game files all still apply. Nothing in it was wrong.

## The stack, and what each piece does

| Area | Implementation |
|---|---|
| Language | Rust, on [Bevy](https://bevy.org/) with wgpu for rendering |
| Assets | Native FastFile readers convert game data into a shared intermediate representation |
| Shaders | Retail Direct3D 9 Shader Model 3 bytecode is translated to WGSL |
| Simulation | Server authority, client prediction and replay share one simulation step over explicit Bevy ECS state |
| Networking | Custom UDP traffic, with a QUIC master for browsing and relaying |

The shader translation and the single shared simulation step are the two pieces worth studying. The second one means replay, prediction and the live game cannot disagree with each other, because they run the same code.

Asset readers also cover MW3 and Black Ops, though MW2 is the one it expects.

## What you need to supply

Your own installed MW2 multiplayer data. Nothing else, and no assets are redistributed.

**Windows** is the easy path: download `iw4l-windows.zip` from the releases, extract it into an empty writable folder, and run `iw4l.exe`. It finds MW2 in your Steam libraries and creates a shortcut. MW2 is required; BO1 and MW3 are optional.

**Linux and macOS** need a real build. Rust through rustup, plus a C toolchain and the Bevy system libraries. `docs/BUILD.md` lists packages per distro, and Linux needs X11, ALSA, udev, Wayland and xkbcommon headers. macOS needs only the Xcode command line tools. Game data comes from the Windows depot of a Steam copy via `steamcmd`, pointed at by the `IW4L_GAMES` environment variable.

This is the shape most rewrites take: the engine is portable, the game files are not.

## How much of it is AI-written

The README's last line before the acknowledgements is *"This whole project is written by an LLM."*

The human work is still substantial and mostly invisible in the commit log. Someone decided the architecture, wrote `AGENT.md`, wrote the documentation set in `docs/`, set the licensing, and wrote `CONTRIBUTING.md` and `SECURITY.md`. Those are judgment calls an agent does not make for you.

IW4L is also rebrand-renamed. Its `NOTICE` says it *"was originally developed by vladtrc"* under an earlier name, which is a reminder that projects in this space get iterated on heavily.

## The provenance problem, and the honest answer

Rewrites that study an existing engine run into a real problem: some of what you write will resemble something you read. That is a licensing question, not a style question, and IW4L handles it in a way worth copying.

`docs/provenance/movement-iw4.md` records that the movement solver in `movement_iw4/src/slide.rs` had *"uncertain provenance because of reported similarities to GPL movement implementations."* The note goes further than most projects would:

- It names the first tracked commit of the old module and the exact blob hash of what was replaced.
- It states plainly that *"This record does not establish that copying or adaptation occurred."*
- It records that the replacement was written by an isolated agent with conversation history excluded, working from a freshly written behavioral contract rather than from the old code.
- It then says the isolation was *"procedural isolation on a shared host, not an enforced filesystem sandbox or a guarantee about model training data."*

That last sentence is the most useful thing on this page. It describes a real control without pretending it is a guarantee. If you build something similar, write that sentence.

`AGENT.md` adds two standing rules with an automated check. Retail offsets stay out of the code, because a standalone runtime resolves nothing against a retail image, and decompiler placeholder names (`FUN_...`, `DAT_...`) are dead weight. `make publish-check` greps the tracked tree for both shapes. The file is careful about what that proves: *"it proves no claim about origin or licensing, and passing it is not an argument for anything beyond the absence of those shapes."*

## Licensing, spelled out

Apache-2.0 for the project's own code, with a `NOTICE` that separates its own material from what it bundles. Two fonts are compiled into the binary with `include_bytes!`, so they ship with every build:

- **Oxanium**, SIL Open Font License 1.1
- **Fira Mono**, SIL Open Font License 1.1, from an unmodified Mozilla revision

`iw4l.exe licenses` prints the licence texts compiled into the executable. That is a small detail that shows the project expects people to redistribute it.

The README also credits what informed the work: [OpenAssetTools](https://github.com/Laupetin/OpenAssetTools) and its `iw4x-x64` fork for asset layouts, [IW4x](https://github.com/iw4x/iw4x-client) for asset and protocol behavior, [KisakCOD](https://github.com/SwagSoftware/KisakCOD) for engine structure, and Ghidra for inspecting the original binaries.

Read that list as a provenance record rather than a formality. Naming what you studied is how a reader can judge the result.

## What works, and what doesn't

The README's claim is deliberately narrow: explore maps, fight bots, record and replay demos. Gameplay is incomplete.

`docs/` is organized around that honesty. There are separate files for rendering, simulation, map loading, the GSC runtime, bot AI, and performance, and a page documenting what commands let you poke the live process. `docs/PERF.md` insists on native `.pftrace` traces as *"the only runtime truth"* and gives you SQL for querying them. `docs/BENCH.md` covers map-load timing. There is an approved-scenarios suite for repeatable end-to-end checks, including two clients through a dev master.

Two things that will surprise you if you don't read them:

**Cheats are on by default.** The host accepts `move`, `look`, `tp`, `nudge`, `god`, `kill`, `force_spawn` and a `give` supply list. `--no-cheats` turns them off. This is a research runtime, not a competitive client.

**APIs, caches, config and the wire protocol change between commits.** Multiplayer peers must run the same build. Anyone planning to test with a friend needs to agree on a commit first.

## The rules the project set for itself

Worth reading as templates, whether or not you touch a project like this.

`AGENT.md` is a single short file, not a sprawling document. It says what the agent must not touch, what must stay out of the published tree, and how to read the files in order.

`CONTRIBUTING.md` draws a line most projects don't:

> Small and self-contained: a fix, a crash, a wrong constant, a doc correction. Open it directly.
> Architectural: a new crate, a new subsystem, a change to how data flows. Open an issue first. A large branch that arrives unannounced is likely to be turned down.

It also says what a useful bug report contains and what it does not: *"Do not attach your `.env`, a memory dump, or an archive of the game."* And it prioritises honestly, with *"a failure in a scenario the project says works outweighs a missing feature it never promised."*

`CONTEXT.md` describes the maintainer's own workflow: working memory kept outside git in a `context/` folder, one artifact per task with its evidence and verdict, and the rule that *"knowledge that is not in an artifact does not exist"* for the next agent. It is explicitly not the contribution path.

`SECURITY.md` scopes reports to problems that reach past arranged playtests between people who agreed to play, and says upfront that it is a channel rather than a program: no bounty, no SLA.

## What to take from this

- **Read the provenance notes before you admire the code.** They tell you which parts you can safely build on.
- **An agent can write most of the code and none of the licensing.** Budget your own time accordingly.
- **A build system is a real deliverable.** The distro package lists, the cross-build notes, the Windows portable folder, and the licensed-font bookkeeping took human decisions.
- **Narrow claims survive contact with users.** IW4L says what works in two lines and links the rest. Projects that promise everything get judged on the missing parts.

## Credits

- [IW4L](https://github.com/vladtrc/iw4L) by vladtrc: the runtime this case study is about, and the provenance work worth copying.
- [mw2-rust-rust-rewrite](https://github.com/Dj-Shortcut/mw2-rust-rust-rewrite) by Dj-Shortcut: the subject of this page before it was replaced. Its original write-up is in this repo's history, and the reverse-engineering, licensing and asset-supply notes in it still apply to projects like this.
- [OpenAssetTools](https://github.com/Laupetin/OpenAssetTools), [IW4x](https://github.com/iw4x/iw4x-client), [KisakCOD](https://github.com/SwagSoftware/KisakCOD) and Ghidra, credited by IW4L as the research that informed it.

IW4L is unofficial and unaffiliated with the owners of MW2 or its trademarks.

---

<sub>[Spot a mistake? [Edit this page on GitHub](https://github.com/trevaintdead/ai-game-modding-guides/edit/main/guides/12-worked-example-rust-rewrite.md).](https://github.com/trevaintdead/ai-game-modding-guides/edit/main/guides/12-worked-example-rust-rewrite.md) &middot; [Open an issue](https://github.com/trevaintdead/ai-game-modding-guides/issues/new) &middot; Part of [AI Game Modding Guides](https://github.com/trevaintdead/ai-game-modding-guides)</sub>
