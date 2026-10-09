# 2. Passthrough Mods

A passthrough mod links two games that run at the same time. One game (the **host**) draws the world. The other game supplies the gameplay, like movement, blocks, or combat. They swap information constantly so each one sees what the other is doing.

## How it works

Say you want Minecraft inside Skyrim:

1. Skyrim runs normally and draws everything on screen.
2. Minecraft runs with its window hidden and simulates the player, blocks, and combat.
3. A plugin inside Skyrim and a mod inside Minecraft pass information back and forth over shared memory. Minecraft is authoritative for the player; Skyrim provides collision and NPCs.
4. Skyrim draws the Minecraft blocks itself. In SkyCraft 0.1.2, Minecraft exports its world meshes and textures and the Skyrim plugin draws them inside Skyrim's own renderer, so they get Skyrim's depth, lighting and shadows. Only Minecraft's hand, HUD and menus are captured as a picture and laid over the top. Some other projects paste Minecraft's whole picture in instead; [guide 14](14-choosing-a-route.md) explains the difference.

Because both games run together, **every player needs a copy of both**.

In short: two games exchanging state, where neither works without the other running. A normal mod would have one game containing the other's content.

## How the two games talk

This is the decision people get wrong first, so it's worth being concrete. SkyCraft's transport, which every other example copies:

- **Named shared memory.** One block of memory both processes open, called `Local\SkyCraft_v1` on Windows. Both sides map the same physical pages, so a write is visible to the other without a copy through the kernel. It only works because both processes are on the same machine.
- **A header at the top.** Magic number, protocol version, both process IDs, heartbeats. If a header looks wrong, the two sides stop immediately rather than interpret garbage.
- **Latest-value slots for per-frame data.** Player position, camera, frame sync. Written with a seqlock, so the reader can detect a torn read and try again.
- **Two ring buffers for events.** One each way. Things that happen once: a block placed, a hit landed, a save requested. Ring buffers stop you dropping an event under load, which a shared slot would do.
- **Named events for wakeups**, so a waiting process sleeps instead of spinning a core.

Three rules that come out of that layout, and that you should ask the agent for before it writes anything:

**One schema, two languages.** SkyCraft defines the messages once in `protocol/messages.*` and generates a C++ header and a Java class from it. A layout test runs in CI against both. Hand-written structs in two languages drift apart the first time you add a field.

**Everything fixed-size and little-endian.** No serialization library in the hot path. Variable-length data, like a list of collision boxes, goes in the ring buffer as a count followed by fixed-size records.

**Both sides must survive the other dying.** Heartbeats detect a crash. If Minecraft dies, Skyrim hands control back to the player instead of leaving a puppet with no brain. If Skyrim dies, Minecraft pauses. Decide this on day one, because retrofitting a failure path into a working transport is miserable.

> A full step-by-step walkthrough of building one of these is in [guide 9](09-worked-example-passthrough-mod.md).
>
> "Passthrough" covers several different designs: swapping state, pasting the guest's picture into the host, or having the host draw the guest's meshes. [Guide 14](14-choosing-a-route.md) explains the difference, and [guide 15](15-case-studies-what-each-project-actually-did.md) shows how SkyCraft, LibertyCraft, the CrossOver bridges and others did it.

## Examples to study

| Project | Games | Notes |
|---------|-------|-------|
| [SkyCraft](https://github.com/chasmlol/SkyCraft) | Skyrim + Minecraft | A script-extender plugin (C++) plus a Fabric mod (Java) |
| [FalloutCraft](https://github.com/zeyvu/FalloutCraft) | Fallout 4 + Minecraft | A port of SkyCraft. Keeps its Fabric mod with FalloutCraft changes, replaces the game-side plugin. Several SkyCraft features aren't ported yet |
| [OWCraft](https://github.com/Yaekai/OWCraft) | Outer Wilds + Minecraft | Built on SkyCraft with a patch. Includes a design doc and development log |
| [GTA San AnSkateas](https://github.com/ryglizzy/GTA-San-AnSkateas) | GTA San Andreas + Skate 3 | A variation: a plugin loads a Rust rebuild of Skate 3's engine instead of running the whole second game |

Most of these are built on SkyCraft's design, so SkyCraft is the usual starting reference. Read its `docs/DESIGN.md` before you prompt anything. It tells you which game is authoritative for what, and getting that backwards is expensive to undo. It also has the message catalog in section 10, which is the answer to the hardest question in a passthrough mod: what data actually crosses between the games.

## The one thing that decides if it's possible

**Does the host game have a mod loader or script extender?**

With one, this is a realistic weekend project. Without one, the agent has to reverse engineer the game first, and that belongs in [guide 3](03-rust-rewrites-and-ports.md).

Check the table in [guide 8](08-mod-loaders-and-script-extenders.md) before you commit. Skyrim and Fallout 4 have SKSE and F4SE, Minecraft has Fabric, most Unity games have BepInEx or MelonLoader, and most Unreal games have UE4SS. That list is why SkyCraft-style projects are as common as they are.

## Do I need to decompile anything?

Usually not. For a SkyCraft-style mod, you point the agent at the SkyCraft project and say you want the same thing for your games. The agent works out the rest.

What you do need is a **way to run your own code inside the host game**: a script extender (SKSE for Skyrim, F4SE for Fallout 4), a mod loader (Outer Wilds Mod Loader), or a plugin SDK (plugin-sdk for GTA San Andreas). With one of those, the job is much easier.

Without one, the agent may have to reverse engineer the game. Members do use the agent to decompile with a tool like Ghidra when it's needed, and [guide 3](03-rust-rewrites-and-ports.md#do-i-need-to-decompile) covers how that works. Ask the agent to check what your game supports before it starts.

## Step by step

1. **Pick the host game and the gameplay game.** Check that both are single-player or offline, and that the host has a loader.
2. **Look for existing work.** Search for a mod loader, script extender, or existing mods for the host game. Skipping this is the most common way to lose an evening here.
3. **Install both games** and confirm they run normally. Install the host game's mod tools if it has them, and test the loader with an existing mod before you write anything.
4. **Open your agent in a new, empty project folder.**
5. **Send the starter prompt** (below).
6. **Let the agent make a plan and build.** Ask it to tell you what to run and what you should see.
7. **Playtest.** Describe exactly what happened. Paste logs from both games when something breaks.
8. **Repeat** until it works. Keep notes: see the handoff and log templates.
9. **Share it** as a GitHub repo with no game files. See [guide 6](06-rules-legal-and-publishing.md) and [guide 10](10-posting-your-project.md).

## Starter prompt

You don't need a perfect prompt. Keep it plain. Something like:

```
I want to make a passthrough mod like SkyCraft (https://github.com/chasmlol/SkyCraft), but for [Game A] and [Game B].

Clone SkyCraft locally and read its README and docs/DESIGN.md so you understand the architecture. I want the same approach as SkyCraft.

[Game A] is installed at [path]. [Game B] is installed at [path].

Before you build anything, tell me:
- what loaders, APIs or SDKs exist for these games
- does either have online play or anti-cheat? (We don't touch those.)
- what's the smallest thing I can build first to prove this works

Start there, then build it. Don't ask me again before the first one works.
```

Asking about loaders first costs one turn and saves you from a confident plan built on a wrong assumption, which is the common way beginners lose a whole evening. That is the only reason to hold off. Once you know the route, get on with it, and do not keep checking in.

You can add more later, like what features you want first.

## If it gets stuck

Not a rule, but useful: when the agent goes in circles, aim for a smaller goal. A common order:

1. Get your code loading inside the host game and writing a line to a log.
2. Get both sides to open the shared memory and agree on a version.
3. Send one piece of data from one game to the other (like the player's position).
4. Send something back.
5. Make the player move in one game and show up in the other.
6. Add features one at a time (blocks, combat, vehicles, UI).

Steps 1 to 3 are the whole trick. Once one value crosses between the games, the architecture works and the rest is features.

## What to expect

- **It starts rough.** Several early projects describe themselves as experimental. Back up your saves.
- **Versions matter.** Mods are tied to game versions. Write down which versions you tested. GTA San AnSkateas needs GTA San Andreas at version 1.0 US, not the current Steam release or the Definitive Edition.
- **Performance needs tuning.** You're running two games and a message channel at once. OWCraft's notes say skipping the presentation of the hidden window took Minecraft from 25 to 60 fps. Give the agent frame-time logs and it can find things like that.
- **Multiplayer is limited, and mostly not coming.** SkyCraft ships Minecraft-side multiplayer, where guests who also run SkyCraft join your Minecraft world over LAN while each keeps their own Skyrim. FalloutCraft lists multiplayer as not ported yet. Don't plan on it beyond that.

## Ideas that don't work, and why

These come up constantly:

| Idea | Why not |
|------|---------|
| Any online or multiplayer game as the gameplay side | Out of scope entirely. See [guide 6](06-rules-legal-and-publishing.md) |
| A host game with no mod loader and no source | You'd be reverse engineering the whole engine first |
| Two games in different engines on different runtimes, as your first attempt | Every reference project pairs a native host with one Java or .NET gameplay game. A mismatched pair doubles the problem: the host side needs a loader, and the gameplay side needs a mod API, and now you have to find both at once |
| Running the gameplay game truly headless | The visuals are the whole point. SkyCraft hides the window but its client still builds the block meshes Skyrim draws, and renders the hand and HUD offscreen. Without the client there's no mod |
| A mod that needs to work on Linux or macOS | The loaders are Windows tools. See [guide 8](08-mod-loaders-and-script-extenders.md#windows-is-the-common-denominator) |
| Shipping the second game's assets in the release | You ship code and a setup script. The player supplies the game. See [guide 10](10-posting-your-project.md) |

## What decides how hard yours will be

Two answers, both asked before you write anything:

**Does the host game's loader let you hook its render and player code?** A plugin that can only read a config file is no use. You need to move the player puppet, and you need to composite another game's pixels into the frame. If the loader can't do that, the project gets much harder than it looks.

**Does the gameplay game expose a way to inject its physics?** You want Game B's own physics to run unchanged against Game A's geometry. If Game B only runs as a normal game with its own world, you're back to the "rewrite the engine" path from [guide 3](03-rust-rewrites-and-ports.md).

Minecraft passes both because Fabric gives you a mod inside it and it has an integrated server. A game that ships one executable with a fixed world does not.

---

<sub>[Spot a mistake? [Edit this page on GitHub](https://github.com/trevaintdead/ai-game-modding-guides/edit/main/guides/02-passthrough-mods.md).](https://github.com/trevaintdead/ai-game-modding-guides/edit/main/guides/02-passthrough-mods.md) &middot; [Open an issue](https://github.com/trevaintdead/ai-game-modding-guides/issues/new) &middot; Part of [AI Game Modding Guides](https://github.com/trevaintdead/ai-game-modding-guides)</sub>
