# 9. Worked Example: A Passthrough Mod, Start to Finish

This walks through building a passthrough mod from nothing, in a fixed order. The names are written as **Game A** (the host, which draws the world) and **Game B** (the gameplay game, which supplies the mechanics). Swap in your own.

The architecture notes come from [SkyCraft's DESIGN.md](https://github.com/chasmlol/SkyCraft/blob/main/docs/DESIGN.md), so you can check them against the source. SkyCraft is Skyrim plus Minecraft, which maps to Game A and Game B below.

**These examples target Windows.** The main reference projects for passthrough mods do, because the loaders are Windows tools. A few creators report Linux and macOS setups through Wine, Proton or CrossOver. See [guide 8](08-mod-loaders-and-script-extenders.md#windows-is-the-common-denominator).

## Step 0: Pick a pair that can work

Most projects fail here rather than in the code.

Game A (host) needs:
- a loader or script extender; see the reference in [guide 8](08-mod-loaders-and-script-extenders.md)
- to be single-player or offline
- a world the player moves through, first or third person

Game B (gameplay) needs:
- a mod API or SDK, or a headless server mode
- gameplay that runs unseen: physics, inventory, combat
- ideally a way to run without showing a window

### The pairs that go well

| Game A (host) | Game B | Why it works |
|---------------|--------|--------------|
| Skyrim (SKSE) | Minecraft (Fabric) | This is SkyCraft. Both have excellent modding support, and Minecraft has an integrated server. |
| Fallout 4 (F4SE) | Minecraft (Fabric) | FalloutCraft did this as a port of SkyCraft. Its Fabric mod came from SkyCraft with FalloutCraft changes, so several features were never ported. |
| Outer Wilds (OWML) | Minecraft (Fabric) | OWCraft added a patch to make it work. |
| GTA San Andreas (plugin-sdk) | Skate 3 (custom engine layer) | GTA San AnSkateas loads a Rust rebuild of Skate 3's engine rather than running Skate 3 itself. Needs Skate 3 for Xbox 360 extracted from your own disc. |

### The pairs that will waste your time

| Idea | Why not |
|------|---------|
| Any online or multiplayer game as Game B | Out of scope. See [guide 6](06-rules-legal-and-publishing.md). |
| A game with no loader and no source | You'd be reverse engineering the whole thing first. That's the "rewrite" path, not passthrough. |

## Step 1: Install and verify

Install both games. Start each one. Confirm they run, and confirm you know where they're installed:

```
Game A: C:\Games\GameA
Game B:  C:\Users\you\AppData\Roaming\.minecraft
```

Also install Game A's loader and test it with an existing mod. If the loader won't load someone else's known-good mod, stop and fix that first. You want a boring, working baseline before you add anything.

## Step 2: Create the project

```bash
mkdir my-passthrough && cd my-passthrough
git init
```

Create three empty files and ask the agent to keep them updated. These are your memory across sessions:

- `AGENTS.md`: rules the agent must always follow
- `MODLOG.md`: what changed and how it was tested
- `docs/DESIGN.md`: how it works, in plain language

Copy [`templates/AGENTS-starter.md`](../templates/AGENTS-starter.md) and
[`templates/MODLOG-template.md`](../templates/MODLOG-template.md) to get started.

## Step 3: The first prompt

Keep it plain. You're pointing at a working example and stating the substitution.

```
I want to make a passthrough mod like SkyCraft (https://github.com/chasmlol/SkyCraft),
but for Game A and Game B.

Clone SkyCraft locally and read its README and docs/DESIGN.md so you understand
the architecture. I want the same approach as SkyCraft.

Game A is installed at [path]. Game B is installed at [path].

Before you build anything, tell me:
- does Game A have a mod loader or script extender we can use?
- does either game have online play or anti-cheat? (We don't touch those.)

Start there, then build it. Don't ask me again before the first one works.
```

A read-only answer first costs one turn and saves you from a confident plan built on a wrong assumption. After that, build rather than ask.

**What you should get back:** a list of what exists for each game, which loader you'd use, and any blockers. If it says "Game A has no modding support," you have your answer for free. Go pick a different host game.

## Step 4: Get one line into a log

Build first, then write down the shape:

```
Start on step 1. Once it works, write docs/DESIGN.md: the two halves of the mod,
what data crosses between them, and the order we are building it in.
Then keep going.
```

The design doc needs four things, and asking for them by name saves a round trip:

- **The two halves.** Which process hosts which code, and in what language.
- **What data crosses.** The message list. SkyCraft's catalog in its DESIGN.md section 10 is the model: player position per frame, collision sections streamed, NPC positions at 20 Hz, and events like block changes and hits.
- **Which game is authoritative for what.** See step 5.
- **The build order.** Each phase ends in something playable.

Step 1 is always the same: **your code loads inside Game A and writes one line to a log file.**

```
Run it. I want to see one line in the log that says your plugin loaded.
Don't do anything else yet.
```

Two games, one log line. Get that, and the rest is iteration.

Once that works, the next step is a handshake, not more features. Both sides open the shared memory, agree on a protocol version, and log it. That catches the failure you'd otherwise debug much later: the two halves open mismatched structs and interpret each other's bytes as nonsense.

```
Next: both sides open the shared memory block and check the header.
Magic number, protocol version, both process IDs. Log what each side read.
If the header doesn't match, both sides stop and say so rather than continuing.
```

## Step 5: Send one value across

**Which game is authoritative matters, and the obvious answer is usually wrong.**

The intuition is that the host game owns the player, because it's the one you look at. SkyCraft does the opposite: **Minecraft is authoritative for player position and physics.** Skyrim draws the world and provides collision, but the player puppet is moved to wherever Minecraft says.

Decide this before you write the transport, because reversing it later means rewriting both halves. Write it into [`templates/BRIDGE-CONTRACT.md`](../templates/BRIDGE-CONTRACT.md), together with how control goes back to the host for cutscenes, vehicles and menus.

Player position first, because it's easy to see and easy to verify. Send it every render frame:

```
Next: Minecraft is authoritative for player position. Each render frame,
send its interpolated position (the partial-tick render position, not the raw
20 TPS tick position) to Skyrim, which moves the player puppet to match.

Log the value you send and the value Skyrim receives, so I can compare them.
```

Then run both games and walk around. Check the log. Positions should match.

**Verification trick:** log the value on both sides with a timestamp or frame counter. You should never have to eyeball whether two numbers match.

That's the architecture proven. Once one float crosses the boundary, the hard part is done.

Two details worth copying from SkyCraft:

- **Interpolated, not raw.** Minecraft ticks at 20 TPS but renders at your display rate. Send the render position, or movement looks like it steps.
- **Frame lockstep.** Both sides disable their own frame caps and vsync, then sync on an explicit "begin frame N" signal. Without this the two games drift and you get stutter.

## Step 6: Send something back

Now close the loop. Skyrim tells Minecraft what the world is shaped like and where the NPCs are:

```
Next: send collision shapes from Skyrim near the player, plus NPC positions.
Inject them into Minecraft's collision queries so Minecraft's own physics
runs unchanged against Skyrim's geometry. Log both directions.
```

Two games talking. Everything after this is features.

**Before you build any of it, answer the crash question.** If Game B dies mid-frame, the player puppet in Game A has no position and no brain. Heartbeats in the shared memory header detect this, and each side needs a defined safe state: Game A hands control back to the player, Game B pauses rather than simulating against nothing. Decide it now. Retrofitting a failure path into a working transport is a bad afternoon.

**Keep one schema, in one place.** Define the messages once and generate both sides' code from them, or write the structs by hand in both languages and accept that they'll drift the first time you add a field. SkyCraft keeps the definition in `protocol/messages.*` and generates a C++ header and a Java class from it, with a layout test in CI on both sides. Everything is fixed-size and little-endian, so there's no serialization library in the hot path. Variable-length data, like a list of collision boxes, travels as a count followed by fixed-size records.

## Step 7: Add one feature at a time

A sensible order, roughly smallest to largest:

1. Position Game A → Game B
2. Input or state Game B → Game A
3. Game A's player moves and it shows up in Game B
4. Spawn Game B's objects (blocks, enemies) in Game A's world
5. Combat
6. Inventory
7. UI, if either game needs it

After each one: **playtest, then commit.** If a step breaks, `git revert` is instant.

## Step 8: Make it not stutter

Passthrough mods run two games and a message channel at once, so performance is the real enemy.

**The gameplay game still runs its client.** It does not run headless. In SkyCraft 0.1.2, Minecraft's client builds the block meshes and textures that Skyrim then draws in its own renderer (that's how Skyrim's walls hide your blocks), and it renders the hand, HUD and menus offscreen as a picture laid over Skyrim's frame. Projects that paste in Minecraft's whole picture instead ("frame compositing", see [guide 14](14-choosing-a-route.md)) need its renderer even more. "Run it with no rendering" breaks either design.

That makes the hidden window's *presentation* the first thing to look at. OWCraft skips presenting Minecraft's hidden window while linked, which took Minecraft from 25 to 60 fps. Other things worth asking about:

- Send deltas rather than full state, if the values are large
- Log frame times from both processes and compare. Numbers beat guessing

```
Game A is dropping to 40fps. Frame times are in [log path]. Find the bottleneck
before changing anything. Tell me what the profile says first.
```

**Two games means roughly twice the RAM, and the gameplay game's appetite is the surprise.** Minecraft's memory is the part you can control: SkyCraft keeps the JVM heap near 3 GB and renders almost nothing, because the world it draws is a void with no terrain. All the geometry comes from Game A. If your gameplay game is loading its own full world, you are paying for two worlds.

Say it once at the start rather than optimising at the end:

```
Game B is only supplying physics and inventory. It doesn't need to load its
own terrain. Cap its memory and tell me what to set.
```

For the specific symptoms (sliding images, missing depth, stuck cutscenes, NPCs walking through blocks) and what other projects found was causing them, see [guide 16](16-ownership-sync-and-rendering.md).

## Step 9: Make saving and loading work

Skip this and you find out the bad way. Two games with two save systems means loading a Game A save and getting an inconsistent Game B state: your blocks and inventory are gone, or they're in the world but not your inventory.

SkyCraft's answer is a shared save id. The plugin stores an id inside the Game A save. On save, Game B flushes its mirror world and snapshots its own region and player data under that id. On load, it reads the id back and restores its side too, so the two always rewind together.

```
Saving Game A must also save Game B. Put a save id in Game A's save file,
have Game B flush and snapshot its state under that id, and restore it when
Game A loads. Test it by building something, saving, quitting both games,
relaunching and loading.
```

Test that last sentence specifically. Saving and loading is where this class of mod quietly breaks, because both games run fine until you relaunch.

## Step 10: Publish

See [guide 10](10-posting-your-project.md) for posting it, and [guide 6](06-rules-legal-and-publishing.md) for the rules you must not break. The short version: your repo is code only, no game files, ever.

## What actually happened, honestly

- An Elden Ring + Spider-Man mashup took about 3-4 hours of back-and-forth before it worked, and the result was jank but playable. One data point, not a typical runtime.
- The wall people hit tends to be the first time something crosses the boundary and lands in the wrong coordinate space, or the two games' frame clocks drifting apart. Both are normal.
- When you hit one: stop repeating prompts. Write a `STATUS.md`, open a fresh chat, hand it over. This is the one case where starting over pays for itself. See [guide 5](05-testing-and-troubleshooting.md).

## The four things that eat the most time

Worth knowing in advance, because each one costs a day the first time it happens:

**A gameplay game whose mod internals move between versions.** SkyCraft pins Minecraft to one version and keeps all its Mixins in one package with a target list, so a game update breaks one known place instead of the whole mod. Pin your version and write it in the README.

**Coordinate space.** Game A and Game B use different units, different up axes, and different origins. Write the mapping down and unit test it before anything else renders. A units mistake here reads like an agent bug, because the agent is confidently building on a wrong number.

**Two save systems.** Covered in step 9. Test by quitting both games and relaunching, not by saving and reloading inside one session.

**Two processes fighting over the GPU and the frame clock.** Covered in step 8.

## The checklist

- [ ] Game A has a loader and it loads someone else's known-good mod
- [ ] Both games are single-player or offline, and you own them
- [ ] Both are pinned to exact versions, and the README says which
- [ ] You've decided which game is authoritative for the player
- [ ] The design doc names the messages that cross between the games
- [ ] Each side has a defined safe state if the other dies
- [ ] `git init` done, and you committed
- [ ] `AGENTS.md` and `MODLOG.md` exist
- [ ] Agent gave you a recon report before writing code
- [ ] Your code loads and logs one line
- [ ] Both sides handshake and agree on a protocol version
- [ ] One value crosses, logged on both sides
- [ ] Save and load tested across a full quit and relaunch
- [ ] Features added one at a time, each tested and committed
- [ ] README says what works and what doesn't
- [ ] No game files in the repo

---

<sub>[Spot a mistake? [Edit this page on GitHub](https://github.com/trevaintdead/ai-game-modding-guides/edit/main/guides/09-worked-example-passthrough-mod.md).](https://github.com/trevaintdead/ai-game-modding-guides/edit/main/guides/09-worked-example-passthrough-mod.md) &middot; [Open an issue](https://github.com/trevaintdead/ai-game-modding-guides/issues/new) &middot; Part of [AI Game Modding Guides](https://github.com/trevaintdead/ai-game-modding-guides)</sub>
