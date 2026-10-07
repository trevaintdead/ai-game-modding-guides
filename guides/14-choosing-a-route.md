# 14. Choosing a Route: What Kind of "Game in a Game" Is It?

"Minecraft in Skyrim" or "Skate 3 in GTA" can mean at least seven different things. They look alike in a video and are built completely differently. Picking the wrong one is the most expensive mistake you can make, because it decides what both games have to support, what every player needs to own, and what can never work.

This guide sorts the routes by the question you're asking, then gives a real example of each, with what was checked and what wasn't.

> **Reading the examples.** When a page says something "works", that's the creator's report. Versions move fast, so check each project's current README.

## The seven routes at a glance

| Route | What actually runs | What crosses between them | Every player needs | Example |
|-------|-------------------|---------------------------|--------------------|---------|
| 1. Live passthrough | Both real games, at the same time | State: positions, collision, hits, events | Both games | [SkyCraft](https://github.com/chasmlol/SkyCraft) |
| 2. Frame compositing | Both real games; the guest's *picture* is pasted into the host's | Colour, depth and HUD images, plus a camera pose | Both games | Minecraft × GTA V in [universal-modder](https://github.com/rehan-remade/universal-modder/tree/main/examples/minecraft-gta5-passthrough) |
| 3. Native geometry transfer | Both real games; the host draws the guest's meshes itself | Meshes, textures, collision shapes | Both games | SkyCraft's renderer, [LibertyCraft](https://github.com/mrborghini/libertycraft), [GalaxyCraft](https://github.com/M0uidev/GalaxyCraft) |
| 4. Shared neutral simulation | A separate server; each game is only a viewer | Neutral positions, inputs and events | Whichever viewer they use | [Signet](https://github.com/kian-cx/signetprotocol) |
| 5. Engine recreation | One rebuilt engine reading your own game data | Nothing crosses; it's one program | The original game's files | [IW4L](https://github.com/vladtrc/iw4L), [benilla](https://github.com/samwhosung/benilla), [HL2-RS](https://github.com/kvalls/hl2-rs), [CS:Craft](https://github.com/FrosttysBots/CS-Craft) |
| 6. Asset or map conversion | Only the host game | Converted files, made once, offline | The host game (and their own copy of the source game) | Doom maps rebuilt as Hytale blocks, a Halo CE map imported into IW4L |
| 7. Rebuilt guest engine or mechanic inside a real host | The real host game, plus a rebuilt engine or one rebuilt mechanic from the guest | The rebuilt part's state, through a DLL or a worker process | The host game, plus their own copy of the guest's data | Skate 3 in Bully and Garry's Mod; [Faith Runner](https://github.com/tnrjns/faith-runner); Diablo II movement in DevilutionX |

Routes 1 to 3 are the ones people call "passthrough". A few rarer ideas also get called mashups but are different again: linking progression between separate games (multiworld randomizers), translating one game's network protocol into another's server, compiling programs into Minecraft commands, and running an emulator inside a game. They're out of scope here. Guides [2](02-passthrough-mods.md) and [9](09-worked-example-passthrough-mod.md) cover the common SkyCraft style. Route 5 is [guide 3](03-rust-rewrites-and-ports.md).

## Start with these four questions

### 1. Do you want the guest game's real behaviour, or just its look?

- **Real behaviour** (Minecraft's own physics, inventory, crafting): you need the real guest game running. That's routes 1 to 3.
- **Just the look, or a few rules**: convert assets (route 6) or rebuild one mechanic (route 7). These are far smaller jobs.

Original-looking assets don't mean original physics. Signet's Doom viewer shows Doom-looking walls, but the movement rules are a simplified shared mode, not Doom's engine.

### 2. Who owns the player?

Write this down before any code. It decides the whole transport.

| Project | Player movement owned by | Host still owns |
|---------|--------------------------|-----------------|
| SkyCraft | Minecraft | The world, NPCs, rendering. Furniture, mounts and kill-moves hand control back to Skyrim temporarily |
| Minecraft × GTA V (on foot) | GTA V. GTA's pose places the Minecraft player every frame | Everything except the Minecraft world layer |
| Minecraft × GTA V (elytra flight) | Minecraft | Look direction and the chase camera |
| [CrossOver bridges](https://github.com/justbustin/minecraft-crossover-bridge) (Elden Ring, Monster Hunter: World) | Minecraft drives movement and camera while active | Enemies, health and display |
| [Minecraft × Half-Life](https://github.com/SawyerTheNerd/Minecraft-X-HalfLife) (GoldSrc) | Minecraft | Ladders, `use`, noclip and death hand movement back to Half-Life |
| [Garry's Redemption](https://github.com/codeByAlexff/garrys-redemption) (design doc) | Hidden Garry's Mod (sandbox and player physics) | RDR2 rendering, AI, law, quests and saves |

There's no single right answer. Pick one, and write down how control **comes back** (cutscenes, vehicles, furniture, death).

### 3. Can the host draw the guest's world itself?

- **Yes, through the host's renderer** (route 3): best looking, because the guest gets the host's lighting and shadows. Hardest, because you're working inside the host's render pipeline.
- **No, paste pictures on top** (route 2): quicker to get working and portable between hosts, but lighting and shadows are approximated from the host's image, and stale images are a real problem (see [guide 16](16-ownership-sync-and-rendering.md)).

### 4. What does each player have to own and install?

If both games run, every player needs both games, both loaders, and matching versions. Some projects need a specific old build, such as a downgraded 1.0 US executable. [NewVegasCraft](https://github.com/Davozh/new-vegascraft) targets Steam New Vegas 1.4.0.525. The Minecraft × GTA V example was tested by its author on GTA V Legacy Steam build 3889. Write the versions in your README from day one.

## Route by route

### Route 1. Live passthrough (state exchange)

Both games run. A plugin in the host and a mod in the guest exchange state through shared memory or a local socket.

- **What it's good for:** keeping the guest's real gameplay (SkyCraft keeps Minecraft's inventory, combat processing and block interactions).
- **What it costs:** you need a loader on both sides, and you have to bridge every system you want: collision, combat, water, menus, death.
- **Real detail worth copying:** SkyCraft uses separate shared-memory areas for different jobs: snapshots for state that's constantly replaced, bounded queues for one-off events, and a triple-buffered overlay for the HUD. LibertyCraft, which forked SkyCraft, kept that design. A dropped event and a late snapshot have different consequences, so they get different channels.

### Route 2. Frame compositing (picture transport)

The guest renders its own world. Its colour, depth and HUD images are copied into shared memory, and a host-side hook (often a ReShade add-on) blends them into the host's frame using depth.

- **Examples:** Minecraft × GTA V (universal-modder example), the CrossOver bridges for Elden Ring and Monster Hunter: World, NewVegasCraft, [Wither Storm](https://github.com/VortexisTV/wither-storm-gta5-passthrough) × GTA V.
- **What it's good for:** getting something on screen quickly, and reusing one guest across several hosts.
- **What it costs:** pixels go GPU → CPU → shared memory → GPU every frame. That round trip through the CPU costs upload time. NewVegasCraft's creator reports getting per-frame guest upload from 18.7 ms down to about 5 ms by switching to dynamic textures.
- **What it can't do on its own:** make guest blocks receive real host shadows, or stop the host's NPCs walking through guest blocks. Those need state exchange as well (route 1). Most real projects are a mix.

### Route 3. Native geometry and collision transfer

The guest exports meshes, textures and collision; the host draws and collides with them natively.

- **Examples:** SkyCraft inserts Minecraft geometry into Skyrim's D3D11 world render, before Skyrim's post-processing. LibertyCraft does the same for GTA IV's D3D9 renderer. GalaxyCraft sends Minecraft geometry as GameCube display lists and KCL collision so Super Mario Galaxy 2, running in Dolphin, draws it.
- **What it's good for:** the guest looks like it belongs: native lighting, shadows, fog, depth of field.
- **What it costs:** you're working with the host renderer's timing and state. Translating API calls isn't enough. LibertyCraft has to snapshot GTA state on the game thread, draw on the render thread, and save an opaque-depth copy before glass is drawn.

### Route 4. Shared neutral simulation

Neither original game runs the shared match. A separate server runs one simplified simulation, and each game (or a recreation of it) is a viewer that translates it into its own look.

- **Example:** Signet (a Rust SDK and a dedicated server). The SDK predicts movement at a fixed 20 Hz, and its Doom and OpenArena viewers are reimplementations. Its Minecraft support is a gateway to an official server.
- **What it's good for:** players in different games sharing one match with consistent rules.
- **What it costs:** everyone plays by the shared rules, not their own game's physics. In its code, movement is 2.5D (one floor per column, so overpasses collapse) and shooting is horizontal. Its planned "Forge" AI translation tool is explicitly unimplemented.

### Route 5. Engine recreation

Rebuild the engine and load the original game's data from your own install. Covered in [guide 3](03-rust-rewrites-and-ports.md) and [guide 12](12-worked-example-rust-rewrite.md). Once you have a recreation, combining it with another one is ordinary programming: the [2010 Rust Rewrite Mashup](https://github.com/chasmlol/2010-rust-rewrite-mashup) maps COD bullets to block damage and explosions to TNT-style destruction inside a recreated runtime.

- **Other examples:** HL2-RS rebuilds parts of Half-Life 2 in Rust from Source assets. CS:Craft rebuilds CS:GO in Rust/Bevy and adds a Minecraft mode, and its "stitching" plan with IW4L (shared content IDs, per-game weapon behaviour, a common trace interface) is still a design. [World of Skatecraft](https://github.com/Kimmo3223/world-of-skatecraft) adds a rebuilt Skate engine to benilla, a recreated WoW 1.12.1 client, through an extension entry point.
- **Watch for:** being written in the same language doesn't make two engines composable. Coordinates, entity IDs, physics, input, animation and saves still have to be reconciled.

### Route 6. Asset or map conversion

Convert the guest's maps or models once, offline, into the host's formats. Doom maps rebuilt as Hytale blocks, one open-world RPG's world converted into another's, and [PipeLink](https://github.com/Sm1jjj/PipeLinkLauncher)'s conversion of owner-extracted models into RenderWare are all examples.

- **Into a recreated engine:** a Halo CE map can become level geometry, textures, BSP collision and spawns inside IW4L, while IW4L keeps MW2's movement and weapons. In that project, scenery is visual only, and teleporters, pickups and vehicles are missing.
- **Collision-only conversions:** PipeLink turns the owner's GTA San Andreas collision files into a collision-only map for the Skate engine while GTA keeps drawing the world. All surfaces become one material, so per-surface sounds are lost.
- **Watch for:** a converted map isn't the original game. One Doom-in-Hytale conversion left a hidden door that blocks completing the first level.

### Route 7. Rebuilt guest engine or mechanic inside a real host

Run the real host game, and plug in a rebuilt version of the guest's engine, or just one of its mechanics. Nothing from the original guest executable runs.

- **A whole rebuilt engine, three ways (the Skate 3 family):**
  - **In-process:** some projects load a 32-bit Rust Skate engine DLL through a C interface inside an older 32-bit host.
  - **[BullySkate](https://github.com/Faiqie/BullySkate)** keeps Bully in its own process and runs the rebuilt Skate physics and sound in separate 64-bit worker processes, talking through small fixed shared-memory blocks. You leave the board for doors, shops and missions, then get back on.
  - **[SkateGM](https://github.com/the-schwilliam/SkateGM)** runs the same engine lineage inside Garry's Mod, and converts every controller into the Xbox-style input the engine already understands.

  All of them load the player's own extracted Skate 3 data. Where the process boundary sits decides what happens when the guest crashes or stalls: BullySkate's host watches its workers' health, but errors there generally still need Bully reopened; the in-process DLL has to recover by itself (restore the last good pose, and stop if errors repeat).
- **One mechanic:** Faith Runner rebuilds selected Mirror's Edge movement in Rust and plugs it into Skyrim and Minecraft through a C interface. [AC1 Movement Rewritten](https://github.com/Banned445/AC1-Movement-Rewritten) rebuilds Altaïr-style movement on a Bevy greybox, with a capsule fallback when no game data is present. The DevilutionX Diablo II movement mod keeps Diablo I's tile occupancy and combat underneath.
- **What it's good for:** the smallest project that still feels like "Game B inside Game A". The Diablo II movement mod keeps a way back: if continuous movement stalls, it re-centres the hero and returns to stock tile walking.
- **Watch for:** matching extracted numbers isn't proof of matching feel. Faith Runner's doubled gravity is inferred from a developer comment; the actual source of the doubling is unresolved.

## Picking for your idea

| You want | Start with | Avoid |
|----------|-----------|-------|
| "Play Minecraft in [world game]" with building and Minecraft combat | Route 1 + 3, forked from SkyCraft | Starting from nothing |
| Something on screen this weekend | Route 2, from the universal-modder GTA V example | Promising native lighting |
| A friend in a different game joining your match | Route 4 | Assuming either game's own netcode will help |
| Another game's movement in your favourite game | Route 7 | Running the whole second game for one mechanic |
| Skateboarding (or another full mechanic set) in an open-world host | Route 7 with a rebuilt engine, in a worker process | Hooking the original guest executable |
| A level from one game playable in another | Route 6 | Calling the result "running" the original |
| Your own engine for an old game, then mashups | Route 5 | Committing game data to the repo |

## Before you commit to a route

- [ ] You know which route (or mix of routes) you're building.
- [ ] You've written down who owns the player, and how control returns.
- [ ] Both games' exact versions and loaders are listed.
- [ ] You know what every player has to own and install.
- [ ] You've found the closest existing project and read its design doc and its known limitations.
- [ ] You've copied [`templates/BRIDGE-CONTRACT.md`](../templates/BRIDGE-CONTRACT.md) into your project.

Next: [guide 15](15-case-studies-what-each-project-actually-did.md) for worked case studies, and [guide 16](16-ownership-sync-and-rendering.md) for the symptoms you'll hit and what usually causes them.

---

<sub>[Spot a mistake? [Edit this page on GitHub](https://github.com/trevaintdead/ai-game-modding-guides/edit/main/guides/14-choosing-a-route.md).](https://github.com/trevaintdead/ai-game-modding-guides/edit/main/guides/14-choosing-a-route.md) &middot; [Open an issue](https://github.com/trevaintdead/ai-game-modding-guides/issues/new) &middot; Part of [AI Game Modding Guides](https://github.com/trevaintdead/ai-game-modding-guides)</sub>
