# 15. Case Studies: What Each Project Actually Did

Videos make every project look the same. The code doesn't. This guide walks through real projects by the problem they solved, with the versions, the interfaces, who owns what, the unit conversions, the platform limits and the things that went wrong.

Read [guide 14](14-choosing-a-route.md) first if you haven't picked a route.

> **How to read these.** "Reports" means the creator says so. "The code does" means the project's code does it; that doesn't mean it was tested. Check each project's current README before you copy a detail.

## Case 1. Forking a working bridge for a new host: SkyCraft → LibertyCraft

**Question it answers:** how much of SkyCraft can you reuse for a different host game?

| | [SkyCraft](https://github.com/chasmlol/SkyCraft) | [LibertyCraft](https://github.com/mrborghini/libertycraft) |
|---|---|---|
| Host | Skyrim (SKSE, D3D11) | GTA IV (a community scripting SDK, D3D9) |
| Guest | Minecraft, Fabric | The same Fabric mod, forked |
| Platform | Windows | Linux, with GTA IV running under Wine; the transport was split into Windows and POSIX versions |
| How the host's collision is obtained | Reads Skyrim's Havok collision shapes directly | Samples GTA geometry with native line probes and nearby objects |
| What the guest receives | Triangles plus eighth-block voxels | The same representation |

**What was inherited.** LibertyCraft's first commit says plainly that it forks SkyCraft's Fabric mod and its MIT protocol header and keeps protocol layout v11. The guest mod, the shared-memory design and some helper code came across.

**What was new.** Everything about GTA: driving, vehicles and seats, phones, pedestrians, guns, crimes, weather, and the host renderer. The commit history shows it in order: scaffold, guest port with a Linux bridge, GTA IV downgrade and launcher tooling, the native plugin core (marked "untested in game" at that point), vehicles, rendering, collision and combat, then playtest fixes.

**Lessons worth copying:**

- **Reuse the representation, not the method.** Both projects send triangles and voxels to Minecraft. Only the way they get that geometry from the host differs. That's why the guest side could be kept.
- **Keep the fixed protocol layout, but don't assume the meaning stayed the same.** LibertyCraft changed the magic number and added GTA-specific events and flags. Matching layout isn't compatibility with an old SkyCraft build.
- **Keep the history honest.** An early commit "fixed" a Games for Windows Live connection loop; a later commit says the real cause was a different add-on and removes it. Write down the correction, not just the first theory.
- **Credit lineage.** LibertyCraft keeps SkyCraft's attribution. Which exact SkyCraft commit it forked from isn't recorded, because the first commit has no parent. If you fork, write down the upstream commit you started from.

## Case 2. A frame-compositing bridge you can read end to end: Minecraft × GTA V

**Question it answers:** what does a picture-transport bridge actually need?

This is the `examples/minecraft-gta5-passthrough` folder in [universal-modder](https://github.com/rehan-remade/universal-modder). Its README credits Claude Code and says the author tested it in September 2026 on Steam build 3889 of GTA V Legacy (ScriptHookV build 3889.0, ReShade version 6.8.0).

- **Two channels.** Gameplay messages use a localhost WebSocket. Three Minecraft images (world colour, depth, and hand plus HUD) travel through named Windows shared memory. A ReShade add-on uploads them and blends them into GTA's finished frame.
- **Ownership changes with the mode.** On foot, GTA's pose places the Minecraft player. In elytra flight, Minecraft owns movement and GTA supplies look and the chase camera.
- **Collision is rebuilt, not copied.** GTA probes the ground around the player (160 columns per frame within a radius of 40) and Minecraft gets barrier blocks. That's a walkable surface, not walls, overhangs or interiors. Blocks placed in Minecraft become frozen GTA boxes (at most 400). Slabs and stairs are just "solid".
- **Combat crosses as events.** GTA people become invisible villagers in Minecraft; Minecraft fighters become invisible frozen GTA doubles. Damage is sent as events.
- **Known limit, stated by the author:** the demo cuts around a stale Minecraft image left over GTA's pause menu; the fix wasn't built.

**What the included test proves.** `ws_test.cpp` passes if any message contains the word `explosion`. It doesn't check camera accuracy, depth alignment or reconnecting. A passing test here means "the socket talks", nothing more.

## Case 3. Same idea, two different hosts: the CrossOver bridges

**Question it answers:** if you've done one host, how much carries over to the next?

A single creator built Minecraft 1.21.1 (Fabric) bridges for Monster Hunter: World and Elden Ring. Minecraft runs natively on macOS; the host runs in CrossOver; a file-backed shared memory area is visible to both.

| | Monster Hunter: World | Elden Ring |
|---|---|---|
| Host version | 15.23.00 (build 421810) | App 1.17.1 (executable 2.7.1.0) |
| Graphics path | a Direct3D-on-Metal layer, D3D11 | D3DMetal, D3D12 |
| Image layers sent | World, depth, combined hand+HUD | World, depth, HUD, separate hand |
| Lighting the guest | Multiplies the world by blurred host-image brightness | Two downsample passes for ambient and haze, plus a colour tint on the world and hand |
| What "damage amount" means | Native HP | Minecraft damage, converted by the target's maximum HP |

**Lessons worth copying:**

- **Same magic number, different meaning.** The two protocols share a magic and version but differ in header size, regions, units and damage meaning. They live in separate folders for that reason. Version your protocol per host.
- **The matching frame doesn't always arrive.** The host keeps eight recent camera poses and prefers the exact matching frame, then an older one, then the last uploaded picture. It logs how often each happened. Log your fallbacks too.
- **Resolution cap.** Frames above 1920×1200 fall back to a transparent overlay window with no depth occlusion.
- **A sync bug in one host and not the other.** The Monster Hunter frame writer's sequence counter doesn't go odd at the start of a write the way its comment says, so a reader could accept a half-written frame. The Elden Ring writer gets it right. It isn't known to cause a visible glitch. Have the agent re-read your sync code against its own comments.

## Case 4. Debugging alignment without fooling yourself: NewVegasCraft

**Question it answers:** the guest image slides or shakes against the host. What do you do?

[NewVegasCraft](https://github.com/Davozh/new-vegascraft) runs Minecraft beside Fallout: New Vegas (Steam 1.4.0.525) using an xNVSE plugin and a ReShade compositor. Its commit pages credit Claude Opus 5.5 and document a long diagnostic sequence, including the wrong turns.

| Problem | What was tried | Where it ended up |
|---|---|---|
| Crash on the first native collision ray | Stack alignment | A 4-byte-aligned stack met 16-byte SSE loads; aligned storage fixed it |
| Host depth was empty | Copy before clear; an INTZ texture theory | The real scene depth was 4× MSAA. The INTZ patch was removed as a false lead; MSAA had to be off for that setup |
| Shaders failed to compile under Proton | | A native 32-bit `d3dcompiler_47` replaced the built-in one in that setup |
| The image shook | Moved where the host pose is read | The host pose is now read when the frame is presented, not in the main loop, so the pose matches the image actually shown |
| Uploads stalled | Dynamic textures | Creator reports 18.7 ms → about 5 ms per guest upload |
| The world seemed to drift | FOV scaling, projection dumps, motion tools, lag tests | FOV control was removed. The remaining "drift" was a real Minecraft block intersecting a sign |

**Lessons worth copying:**

- **Build measurement tools first.** NewVegasCraft cycles composite, host-depth, guest-depth and difference views on one key, drops a marker pillar where the host crosshair points on another, and dumps the native projection matrix on a third.
- **Remove failed fixes.** Several plausible fixes were wrong. The project removed them instead of stacking every theory as a requirement.
- **Test the transport with a fake host before the real game.** Its fake Linux host compared each exported frame with the camera pose recorded for it, so a misalignment couldn't hide behind comparing different frames. That's a synthetic test, not proof the real game lines up.

## Case 5. Drawing the guest with the host's renderer: Minecraft × Half-Life

**Question it answers:** what does route 3 look like on an old engine?

SawyerTheNerd's [Minecraft × Half-Life](https://github.com/SawyerTheNerd/Minecraft-X-HalfLife) (GoldSrc, not Half-Life 2) credits Claude Opus 5.5. A hidden real Minecraft supplies movement, physics and meshes. Modified Half-Life client and server DLLs draw the Minecraft geometry through Half-Life's own OpenGL pipeline. Only the HUD is a pasted image.

- **Units:** 40 Half-Life units per block; 72 units is about 1.8 m.
- **Collision:** BSP faces and the player-clip hull become Minecraft collision, with the hull's expansion removed first. Moving brushes send updates.
- **Handing back control:** ladders, `use`, noclip and death return movement to Half-Life. Crouch heights don't line up exactly.
- **Health:** Minecraft's 20-point health maps to Half-Life's 100 with a factor of 5.
- **Open items:** the project's own task list keeps arms, hazards, lighting, mob paths and shared death as not yet accepted in game.

## Case 6. Exporting geometry into an emulated game: GalaxyCraft

**Question it answers:** can you do this with a console game in an emulator?

[GalaxyCraft](https://github.com/M0uidev/GalaxyCraft) splits the work across three programs: Minecraft/Fabric for blocks, inventory and generated meshes; Dolphin running the original Super Mario Galaxy 2; and a native module inside the game for rendering, gravity, collision and Mario's adaptation.

- **The guest's geometry is drawn by Mario's game.** Minecraft data is sent as GameCube display lists, textures and KCL collision.
- **Two byte orders.** The host protocol is little-endian (version 10); the emulated mailbox is big-endian (version 5). Much of the model data is already big-endian and must not be swapped twice. A C assertion file in the repository pins the offsets.
- **Two ownership modes.** Native Mario movement, or Minecraft movement with Mario hidden and Steve drawn.
- **Caps:** the README describes broad block and entity coverage; the protocol has finite caps.

## Case 7. A shared match instead of a bridge: Signet

**Question it answers:** can players in different games share one match?

[Signet](https://github.com/kian-cx/signetprotocol) runs one neutral simulation on a server and lets each game act as a viewer. Its SDK predicts movement at a fixed 20 Hz, replays unacknowledged commands when the server corrects it, and logs corrections over 5 cm.

- **The shared rules are simple on purpose.** Movement is 2.5D, so stacked floors collapse into one column. Shooting is horizontal. Original-looking Doom walls don't mean Doom physics.
- **The planned AI translator is a plan.** The "Forge" pages say it isn't implemented and no model has been validated.
- **The code is less strict than the docs.** The docs say the client never blocks; the code writes to TCP while holding a lock. The docs say authority makes cheating impossible; the roadmap still lists command-rate checks as pending. Believe the roadmap.

## Case 8. One mechanic, many hosts: Faith Runner and AC1 movement

**Question it answers:** do you need to run the whole second game?

- **[Faith Runner](https://github.com/tnrjns/faith-runner)** rebuilds selected Mirror's Edge movement in Rust. It needs only box sweeps and overlap queries from its host, so Skyrim's Havok, Minecraft blocks or a Bevy greybox can all provide collision. Skyrim uses 70 units per metre in that port. The Skyrim port guesses ladders and other fixtures from collision shapes, and moving objects stay stale until reread.
- **[AC1 Movement Rewritten](https://github.com/Banned445/AC1-Movement-Rewritten)** rebuilds Altaïr-style movement on a Bevy greybox and imports models and animations from your own PC install. It falls back to a plain capsule when no game data is present, which keeps the movement code testable without the game. Treat this as the creator's description.

## Case 9. A plan versus what shipped: Garry's Redemption

**Question it answers:** how do you tell what's built from what's planned?

The project's first commit contains a 219-line planning document for an agent. It plans in-frame Vulkan/DX12 compositing and depth-aware props. The later release notes describe a separate overlay window above RDR2 that can't do exclusive fullscreen, and call true in-frame drawing unbuilt. Native acceptance is reported on one Windows 11 / NVIDIA / Vulkan PC.

**Lesson:** a design document is a plan. Before you list a feature in your README, check the release notes and the code.

## Case 10. One rebuilt engine, several hosts: Skate 3 in Bully, Garry's Mod and WoW

**Question it answers:** where should a rebuilt guest engine live, in the host's process or beside it?

These projects use the player's own extracted Skate 3 data and a rebuilt Skate engine from the same community lineage. None runs the original Skate 3 executable.

| | [BullySkate](https://github.com/Faiqie/BullySkate) | [SkateGM](https://github.com/the-schwilliam/SkateGM) | [World of Skatecraft](https://github.com/Kimmo3223/world-of-skatecraft) |
|---|---|---|---|
| Host | Bully (native adapter + scripting) | Garry's Mod | [benilla](https://github.com/samwhosung/benilla), a recreated WoW 1.12.1 client |
| Where the engine runs | separate 64-bit physics and sound worker processes | inside Garry's Mod | inside the recreated client, through its extension entry point |
| Host collision | converted Bully collision; physical collision only, not navigation volumes | Garry's Mod world, with an extra collision layer for moving props | not described |

**Details worth copying from BullySkate:**

- **Small, fixed contracts.** Bully and the physics worker share a 6,184-byte block with no pointers (up to 24 actors, 8 vehicles, 8 input samples). Sound gets its own 296-byte block, protected by an odd/even sequence counter.
- **Axes stated once.** Bully is Z-up and the Skate engine is Y-up, so positions go across as (x, z, −y).
- **Its own clock.** The physics worker steps on a fixed period built from queued input durations, not one step per rendered frame. If inputs pile up, it merges them and caps the catch-up, so a long stall can lose a button press.
- **Sound fails quiet.** If no new sound state arrives for 250 ms, the sound worker mutes rather than looping the last sound.
- **Hand back for the host's own content.** You step off the board for doors, shops and missions, then get back on.

**From the others:** SkateGM maps every controller (PlayStation, Switch, generic) onto the Xbox-style input the engine already expects, then picks button labels separately, so the engine's input code never changed. World of Skatecraft adds a skateboarding profession through a patched server; the Windows setup uses a stock server, so it doesn't have that feature.

Other projects load the same engine as a 32-bit DLL inside an older 32-bit host, through a C interface. That's the lowest-latency choice, but a guest fault is a host fault, so the engine has to recover by itself: restore the last good pose when it produces invalid numbers, and stop if errors repeat within a few seconds.

**Evidence level:** BullySkate's contracts are in its code. The other details are creator reports from READMEs and commit pages.

## Case 11. Using the original game as a reference: Diablo II movement in DevilutionX

**Question it answers:** how do you check a rebuilt mechanic against the real thing?

This mod puts Diablo II-style continuous movement into DevilutionX, the rebuilt Diablo I engine. Diablo I's tile occupancy, combat and saves stay underneath.

- **A reference oracle.** It compares its movement tables against a user-supplied Diablo II 1.12 `D2Common.dll` and an MIT-licensed reimplementation. The direction table is computed rather than copied from game data.
- **A way back.** If continuous movement stalls, it re-centres the hero and returns to stock tile walking.
- **Know which version a report describes.** v0.1 used Diablo II speeds everywhere. v0.2 keeps Diablo I walking pace in dungeons. An older report in the repo still describes v0.1 speeds.
- **Read the test results carefully.** Its report claims 65/65 checks, but those checks measure different things (table agreement, travel time, unchanged level hashes, frame rate). The frame-rate check takes the best of several runs, not the average. Checks that need the owned Diablo II files are skipped when the files are missing.

## Case 12. One engine, three different things: Halo CE in IW4L

**Question it answers:** why do two videos of the "same" mashup behave so differently?

The [Halo / MW2 Director](https://github.com/0xburn/halo-mw2-director) project patches the [IW4L](https://github.com/vladtrc/iw4L) Rust engine (a Modern Warfare 2 runtime) on macOS with Metal, and has three separate modes:

1. **An authored cinematic.** Halo CE characters are retargeted to MW2 rigs and act out a timed script. The Warthog follows a fixed route. It looks like gameplay, but nothing reacts.
2. **An imported map.** A local Halo CE map becomes level geometry, textures, collision and spawns, but MW2's movement, weapons and rules still run. Scenery has no collision, and teleporters, pickups and vehicles are missing.
3. **A bot match.** MW2 soldiers fight Halo-style "Spartans" with changed movement and accuracy rules. There are no Halo shields, and the changed movement profile needs a new network protocol version, so older builds and demos don't match.

**Lesson:** say which mode a clip shows. A cinematic, a map import and a reactive match are different claims.

## Case 13. Lessons from a standalone rebuild: HL2-RS

**Question it answers:** what goes wrong when two renderers or input paths meet?

[HL2-RS](https://github.com/kvalls/hl2-rs) rebuilds parts of Half-Life 2 in Rust, loading Source assets. It isn't a mashup, but its fixes apply to any project that shares a renderer or automates input:

- **Depth test and depth write are separate switches.** Transparent surfaces often need testing without writing.
- **GPU state leaks between passes.** A previous pipeline can stop a depth clear from working; set what you need, then restore it.
- **Automated input takes a different path.** Synthetic Windows input lacks raw mouse motion, so the project falls back to ordinary mouse deltas when raw input is absent.
- **A loaded level isn't campaign parity.** Its scene and NPC tests are narrow, and weapon spread and damage are approximations.

## What the cases have in common

1. Every working project chose an owner for the player and wrote down how control comes back.
2. Every one converts units explicitly: 70 host units per block (New Vegas), 40 per block (Half-Life), 1 GTA metre per block ([Wither Storm](https://github.com/VortexisTV/wither-storm-gta5-passthrough) × GTA V, and LibertyCraft in GTA IV), 2 units per block (ULTRAKILL in [Killcraft](https://github.com/goonsn/Killcraft)), 0.01905 metres per Source unit ([Garry's Redemption](https://github.com/codeByAlexff/garrys-redemption)).
3. Collision is rebuilt into a representation the other game understands, and it's never complete. Write down what's missing.
4. The best-documented projects record their wrong turns.
5. "Tested" usually means one machine, one version. Say which.
6. Where the guest runs (same process, a worker, or another machine) decides what a crash or stall does to the host. Decide it on purpose.

---

<sub>[Spot a mistake? [Edit this page on GitHub](https://github.com/trevaintdead/ai-game-modding-guides/edit/main/guides/15-case-studies-what-each-project-actually-did.md).](https://github.com/trevaintdead/ai-game-modding-guides/edit/main/guides/15-case-studies-what-each-project-actually-did.md) &middot; [Open an issue](https://github.com/trevaintdead/ai-game-modding-guides/issues/new) &middot; Part of [AI Game Modding Guides](https://github.com/trevaintdead/ai-game-modding-guides)</sub>
