# 17. The Decompile System Map

**This is a reference document, not a guide.** It is longer than the rest of this repo on purpose. It exists to be looked up rather than read through, so if you are wondering where to start, read [guide 3](03-rust-rewrites-and-ports.md) instead and come back here when you hit a wall.

Forty-six areas, each listing what has to be worked out and why. Sections keep their original numbering, so you can point someone at "number 33" and they will find the same thing you did.

Star counts quoted in this page are as of October 2026. They are there to show which projects have traction, not to be precise, and they move.

Where a claim could be checked against a public project, it has been, and the project is named. Where it could not, the page says so.

## What a decompile actually is

The goal is to reproduce the original game's behaviour closely enough that your replacement loads the original's legally obtained data and behaves like the original.

Two different jobs get called "decompile", and confusing them costs months:

**Matching decompilation** reconstructs source that compiles to a byte-identical copy of the original binary. [zeldaret/oot](https://github.com/zeldaret/oot) (5,562 stars) does this for Ocarina of Time, and the project states plainly that it "is not producing a PC port." Matching demands the exact compiler, the exact optimisation level, and the original build timestamp, because all three end up baked into the output.

**Reimplementation** builds a new engine that reads the original's data files and behaves similarly, without reproducing the binary. [OpenRCT2](https://github.com/OpenRCT2/OpenRCT2) (16,387 stars) and [OpenMW](https://github.com/OpenMW/openmw) (6,606 stars) are both this, and so is [IW4L](12-worked-example-rust-rewrite.md).

Matching is the harder discipline and the one with more legal exposure, since you are reproducing the actual code rather than its behaviour. [Guide 13](13-reverse-engineering-and-the-law.md) covers that difference.

The four-stage summary this map is built on:

> **Reverse engineering finds the pieces.** **Decompilation reconstructs the code.** **Reimplementation rebuilds the machine.** **Verification proves you actually got it right.**

## 1. Executable and program code

The obvious part, and still the largest.

Functions, classes and structs, globals, constants, pointers, virtual tables, RTTI, callbacks, state machines, memory allocation, threading, timers, error handling, initialisation, shutdown, and the main loop.

The chain you are trying to climb:

```
Machine code -> Assembly -> Decompiler output -> Named systems -> Readable source -> Verified behaviour
```

Tools for that, roughly in the order people reach for them: **Ghidra**, **IDA Pro**, Binary Ninja, radare2 with Cutter, Capstone for disassembly libraries, Frida for instrumentation, x64dbg and WinDbg on Windows, gdb elsewhere, and rr for recording execution so a crash can be replayed.

[ghidra-mcp](https://github.com/bethington/ghidra-mcp) (4,708 stars, Apache-2.0) and [ida-mcp](https://github.com/HexRaysSA/ida-mcp) expose these through an MCP server, so an agent can drive the decompiler directly instead of you pasting disassembly into a chat window. Both are covered in [guide 3](03-rust-rewrites-and-ports.md#when-you-do-need-it).

## 2. Engine core

The machinery the rest sits on.

```
Main loop
├── Timing
├── Jobs and threads
├── Memory
├── Events
├── Object system
├── Resource manager
├── Scene manager
├── File system
└── Platform layer
```

And the per-frame order:

```
Input -> Simulation -> Physics -> AI -> Animation -> Rendering -> Audio -> Present frame
```

Getting this order right tells you where to look for a bug. A camera problem is not in the input code.

## 3. Asset system

A decompile that cannot read the game's files is not much use.

Work out the archives, package files, compression, serialisation, resource IDs, asset lookup, dependency tables, streaming and caching.

```
game.pak
├── textures
├── meshes
├── animations
├── maps
├── sounds
├── scripts
└── configuration
```

The distinction worth memorising, because people get this wrong at the start:

```
Decompiler  = understands code
Asset parser = understands data
```

You almost always need both, and they are separate tools. [Guide 3](03-rust-rewrites-and-ports.md) lists the asset tools by engine.

## 4. Meshes and models

How geometry is stored: vertices, indices, normals, tangents, UVs, vertex colours, submeshes, material slots, LODs, skeleton bindings, and morph targets.

```
Mesh
├── Vertex buffer
├── Index buffer
├── Materials
├── Skeleton
├── Collision mesh
└── LOD 0/1/2/3
```

That the collision mesh is separate from the visible mesh is the point, and section 8 comes back to it.

## 5. Textures

Texture formats, mipmaps, compression, texture arrays, cubemaps, and the normal, diffuse, roughness, metallic, emissive and mask maps a modern material expects.

Old games store their own formats. Those usually bottom out in a small number of known ones once decoded:

| Format | What it is |
|---|---|
| BC1, BC2, BC3 | The S3TC block formats, Direct3D's names for DXT1, DXT3 and DXT5 |
| BC4, BC5 | One and two channel variants, the usual choice for normal maps |
| BC7 | Higher quality RGBA, the late addition to the family |
| DDS | The container those normally arrive in |
| PNG, TGA | Uncompressed or plainly compressed, used by later and 2D-heavy games |

S3TC, also written DXTn, DXTC or BCn, is a group of lossy block compression schemes developed at S3 Graphics and built on block truncation coding from the 1970s. It shipped in DirectX 6.0 and OpenGL 1.3, which is why almost everything from that era uses it. It compresses in fixed-size blocks so the GPU can read one without decoding its neighbours, which is the whole reason it was adopted.

## 6. Materials

Materials say how a mesh uses its shaders and textures.

```
CarPaint.material
├── Shader = VehiclePaint
├── Albedo
├── Normal
├── Metallic
├── Roughness
├── Reflection
└── Parameters
```

To reproduce them you need the material definitions, shader references, texture bindings, render states, blending, transparency, culling and depth rules. Get the depth or culling state wrong and the mesh renders inside out or through walls, which looks like a geometry bug.

## 7. Shaders and rendering

One of the biggest pieces, and the one where matching decompilation projects spend the most time.

Shader stages: vertex, fragment or pixel, geometry, compute, and tessellation.

The renderer around them:

```
Renderer
├── Camera
├── Visibility
├── Lighting
├── Shadows
├── Materials
├── Post processing
├── Reflections
├── Particles
├── Decals
└── UI
```

What has to be worked out: draw calls, render queues, render passes, framebuffer layout, shader uniforms, GPU buffers, texture bindings, lighting models, fog, shadows, bloom, motion blur, tone mapping and anti-aliasing.

Then the API underneath usually has to change, because the original targeted something that no longer exists:

```
DirectX 8, DirectX 9, OpenGL, proprietary API  ->  Vulkan, DX12, modern OpenGL, WebGPU
```

This is a solved problem with mature tooling, and it is worth knowing it is solved. [DXVK](https://github.com/doitsujin/dxvk) (18,245 stars, Zlib) implements D3D8 through D3D11 on Vulkan, and [vkd3d-proton](https://github.com/HansKristian-Work/vkd3d-proton) (2,995 stars) does D3D12. Both run existing Windows games on Linux under Wine, which is exactly the translation problem, already dealt with.

For compiling shaders from source, the DirectX Shader Compiler ([DXC](https://github.com/microsoft/DirectXShaderCompiler), 3,659 stars) is the maintained option for anything Direct3D-era tooling cannot handle.

## 8. Collision

Collision is usually not the visible geometry:

```
Visible mesh != Collision mesh
```

Primitive types: boxes, spheres, capsules, convex hulls, triangle meshes, heightfields, trigger volumes, raycasts.

And the pipeline that turns them into contact: collision layers, collision masks, broad phase, narrow phase, contact generation, triggers, queries. A decompile that renders correctly and lets you walk through walls has usually got the layers or the masks wrong rather than the shapes.

## 9. Physics

Past collision detection: gravity, velocity, acceleration, friction, restitution, rigid bodies, impulses, constraints, joints, ragdolls, vehicles, suspension, buoyancy, and character controllers.

```
Input -> Character controller -> Collision detection -> Physics response -> Animation
```

Game-specific physics matters more than it sounds. A fighting game, a racer and a platformer each have entirely different amounts of code here.

## 10. Skeletons and animation

Bone hierarchy, bind poses, animation tracks, interpolation, animation events, animation blending, inverse kinematics, root motion, morphs, facial animation.

```
Idle -> Walk -> Run -> Jump -> Fall -> Land
```

Which is usually driven by a state machine rather than by "play animation X". Reproducing the state machine is reproducing the gameplay; playing the right clip at the wrong time is visibly wrong.

## 11. Maps and world format

How levels are represented:

```
World
├── Terrain
├── Static geometry
├── Entities
├── Spawn points
├── Lights
├── Triggers
├── Doors
├── Navigation
├── Audio zones
├── Weather
└── Scripts
```

Plus world streaming, chunks, sectors, portals, occlusion, LOD and instancing. Section 28 covers streaming on its own, because it is usually a separate system with its own failure modes.

## 12. Gameplay code

The part players actually perceive: player, weapons, vehicles, NPCs, items, inventory, health, damage, quests, missions, progression, economy, abilities, combat, interaction.

Every odd little behaviour may matter. One action is usually a chain rather than a function:

```
Fire weapon
├── Check ammo
├── Play animation
├── Spawn projectile
├── Raycast
├── Apply damage
├── Spawn effect
├── Play sound
└── Alert AI
```

Get one link out of order and the bug looks like a physics problem or an audio problem.

## 13. AI

Perception, navigation, decision making, combat, cover, pathfinding, behaviour trees, state machines. Specifically: aggro, sight, hearing, patrols, reactions, pathfinding, tactical logic, and scripted behaviour.

Note the boundary with section 14, which is easy to overlook and expensive to get wrong.

## 14. Navigation

Often its own massive subsystem: navmesh, waypoints, path graphs, A*, obstacle avoidance, jump links, doors, ladders, and vehicles as navigable units.

The sentence that matters most in this section:

> AI can be reconstructed perfectly and still look broken if navigation is wrong.

Nothing in this repo covers navigation, which is a gap worth naming.

## 15. Audio

Sound archives, codecs, music, dialogue, sound effects, positional audio, attenuation, environmental zones, reverb, mixing, priorities.

One sound is a graph, not a sample:

```
Gunshot
├── Dry sample
├── Distance falloff
├── Indoor reverb
├── Occlusion
└── AI hearing event
```

The last item is why audio reverse engineering overlaps with AI. Hearing events are how the game knows something happened.

## 16. Input

More than key codes.

```
Keyboard, Mouse, Controller, Touch, VR, Force feedback
```

Bindings, dead zones, sensitivity, analog curves, action mapping, context-sensitive controls.

```
Button A -> "Jump" -> PlayerController::Jump()
```

The step from physical button to named action is a lookup table, and reproducing it correctly is what makes a recreated game feel responsive rather than merely working. [Guide 16](16-ownership-sync-and-rendering.md) covers who owns the player, which is the next question.

## 17. UI and HUD

Menus, HUD, fonts, sprites, widgets, layouts, inventory screens, pause menu, map, subtitles, notifications, plus the code connecting the UI to game state. That last part is where the work is; the layout is the easy half.

## 18. Scripting system

Many games hide a great deal of gameplay outside the native executable. Lua, Python, AngelScript, JavaScript, UnrealScript, custom bytecode, or a proprietary VM.

```
Script loader
VM
Opcodes
Native bindings
Events
Serialisation
Debugging
```

A game's mission logic may live almost entirely here. If a decompile seems to be missing entire features, check whether the original ever had them in the executable at all before spending a week looking for code that is not there.

## 19. Save games

Save format, serialisation, object IDs, versions, checkpoints, progression, configuration. The goal:

```
Original save -> New engine -> Same game state
```

Versioning is the part that bites. A format that carries its own version number tells you how the original handled fields appearing and disappearing.

## 20. Networking

For a multiplayer game this can practically become another project: sockets, packets, replication, prediction, interpolation, authentication, lobby, matchmaking, server browser, dedicated server, voice chat. Plus packet structures, message IDs, state synchronisation, tick rates, authoritative logic, and latency compensation.

This section is here for completeness and it is **out of scope for anything you build from this repo.** [Guide 6](06-rules-legal-and-publishing.md) rules out online play entirely, and nothing here should be read as an exception. Reconstructing a game's protocol for your own offline use is a different act from shipping a multiplayer modification, and if that is the project you are contemplating, stop and read guide 6 first.

## 21. Original development tools

The shipped game is only part of what its developers built around it. Studios had level editors, model converters, texture converters, animation exporters, shader compilers, script compilers, packagers, localisation tools, build systems, and debug consoles.

Recreating these can massively improve a source port. If the game shipped with an official toolkit, that is a better starting point than a decompiler, and [guide 3](03-rust-rewrites-and-ports.md) lists a few.

## 22. Build pipeline

Work out how raw assets became game-ready data.

```
Blender or Maya -> Exporter -> Mesh compiler -> Game mesh
Photoshop        -> Texture compiler -> Game texture
```

and the modern version of the same pipeline:

```
Blender -> Open formats -> Conversion tools -> Game runtime
```

Matching decompilation makes this concrete in a way most people do not expect. [zeldaret/oot](https://github.com/zeldaret/oot) ships a `spec/` directory, `linker_scripts/`, and a `docs/libu64.md` describing the build, plus `docs/compilers.md` recording the exact compiler required, and a table of every regional retail build with its build timestamp and MD5. Reproducing a 1998 binary in 2026 means matching the original toolchain, not a current one.

## 23. Platform layer

Old games talk directly to APIs that no longer exist. Win32, DirectInput, DirectSound, Direct3D, OpenGL, and old console SDK APIs all need replacing.

```
Original engine -> Platform abstraction -> Windows / Linux / macOS
```

This is the layer that makes a port cross-platform, and [guide 3](03-rust-rewrites-and-ports.md) explains why Rust rewrites in this repo tend to be cross-platform while passthrough mods are not.

## 24. Threading and job system

Later engines contain a render thread, physics thread, streaming thread, audio thread, a worker pool, and async IO.

The warning here is worth repeating: incorrect timing in the job system creates bugs that look completely unrelated to it.

## 25. Math

Yes, math behaviour can matter.

Vectors, matrices, quaternions, transforms, fixed point, floating point behaviour, random number generation, interpolation.

And where tiny differences show up:

```
Physics
AI
Replays
Networking
Speedruns
```

Floating point is the one that catches people. Old code built for x87 or for a different optimisation level will not produce identical results in a modern compiler, and the divergence compounds over time. Matching projects solve it by pinning the compiler precisely.

## 26. Random number generation

Games frequently rely on deterministic RNG. You need the algorithm, the seed, the update order, and the call order.

Otherwise:

```
Same save != Same behaviour
```

Anything procedural, and anything the original computes from a random value, will diverge. Section 42 explains why this is the hardest thing to notice.

## 27. Timing

A huge source of compatibility problems, and the area most likely to bite a reimplementation.

```
Game tick
Physics tick
Render tick
Animation timing
Frame limiter
Delta time
Fixed timestep
```

The classic failure:

```
Game designed for 30 FPS -> run at 240 FPS -> physics enters another dimension
```

Matching decompilation makes this concrete. [zeldaret/oot](https://github.com/zeldaret/oot) lists the original build timestamp for every regional release, from 98-10-21 for NTSC 1.0 through to the GameCube revisions, because the timestamp is compiled into the binary and changing it changes the output.

The two-game version of this, where two processes drift against each other's frame clocks, is in [guide 9](09-worked-example-passthrough-mod.md#step-5-send-one-value-across).

## 28. Streaming

Large games continuously stream meshes, textures, audio, terrain, NPCs, scripts and animations.

To reproduce it you need the load radius, priority, memory budget, async IO and unload rules. Get the unload rules wrong and memory climbs until something falls over an hour into play, which is a miserable bug to trace.

## 29. Lighting

Directional, point and spot lights, baked lighting, lightmaps, probes, reflections, shadow maps, ambient lighting, and HDR.

Lighting data may be stored inside the level files rather than separately, which is worth knowing before you go looking for a dedicated asset file that does not exist.

## 30. Effects

Particles, smoke, fire, water, rain, snow, dust, explosions, trails, decals, blood, screen effects.

Many games have custom effect scripting systems, so the particles are usually data driven and the system itself is smaller than the effect list suggests.

## 31. Water, terrain and environment

Sometimes these are mini-engines in their own right.

```
Terrain            Water
├── Heightmap      ├── Waves
├── Splat maps     ├── Reflection
├── Vegetation     ├── Refraction
└── LOD            ├── Buoyancy
                   └── Underwater rendering
```

Terrain with splat maps and vegetation is often a system in its own right rather than part of the world format.

## 32. Vehicles

A vehicle system can include engine, transmission, torque, suspension, tires, steering, damage, collision, audio, camera and AI drivers.

Racing games can spend an enormous amount of code here. If your game has drivable vehicles, budget for it.

## 33. Camera

Often overlooked, and the reason a recreation can feel wrong while everything measures correct.

Field of view, smoothing, collision, follow behaviour, shake, zoom, cutscenes, transitions, and first versus third person.

Bad camera behaviour makes a recreation feel wrong immediately, even to someone who cannot say why. [Guide 16](16-ownership-sync-and-rendering.md) covers the related question of which game owns the camera.

## 34. Cutscenes

Timelines, camera tracks, animation, dialogue, music, triggers, script events, subtitles.

This may have its own proprietary format, in which case it is an asset problem from section 3 rather than a code problem.

## 35. Localization

String tables, fonts, Unicode, languages, subtitle timing, regional assets, and pluralization.

Pluralization is the one that catches reimplementations, because English has two forms and most other languages have more, so a lookup table that works in English is wrong everywhere else. Regional assets can also mean different textures and models per region, which affects section 4 and section 5.

## 36. Configuration

```
.ini, .cfg, .xml, .json, binary config, registry values, console variables
```

And within them, graphics settings, gameplay flags, debug flags, hidden features, and engine variables.

Console variables and hidden flags are where undocumented behaviour hides. If a game does something no setting exposes, it is usually behind a flag nobody documented.

## 37. Debug systems

Extremely valuable if remnants survive, and one of the best places to start a project nobody has attempted.

```
Debug console
Developer menu
Assertions
Logging
Profilers
Cheats
Debug draw
Symbol names
Error strings
```

These can expose the original architecture. Error strings in particular are often the fastest way to find the name of an internal system, because developers wrote them to be read. Debug menus reveal which subsystems exist and how they are toggled.

One caution. This section is about using surviving debug material as documentation. It is not about cheats, and nothing here should be read as a route to gaining an advantage in anything online. [Guide 6](06-rules-legal-and-publishing.md) rules out anti-cheat and online play entirely.

## 38. Object and entity system

Work out what a "thing" in the world actually is.

Either components:

```
Entity
├── Transform
├── Renderer
├── Physics
├── AI
├── Script
├── Audio
└── Gameplay components
```

or an inheritance hierarchy:

```
Object -> Actor -> Pawn -> Enemy
```

Which one the original used matters, because it decides how you add a new thing to it. Getting this wrong means fighting the architecture for the rest of the project. Understanding it can open up the whole engine, which is why this section is early in most people's order of work despite the numbering.

## 39. Resource dependencies

One asset may reference dozens of others.

```
Enemy
├── Mesh
├── Skeleton
├── Animations
├── Material
├── Textures
├── Sounds
├── AI definition
└── Script
```

So you need a working resource dependency graph. Without one, loading a single asset means guessing what else to load, and you find the missing dependencies by crashing.

## 40. Behavioural verification

The decompiler saying something **does not make it correct.**

```
Original game vs reimplementation
```

Test positions, physics, timing, damage, AI, animation, rendering, RNG, inputs, and saves. Automate as much as you can.

Matching projects turn this into a build-breaking check. [zeldaret/oot](https://github.com/zeldaret/oot) ships `diff.py` and `diff_settings.py`, which build the ROM from source and compare it byte for byte against the retail image, with a per-version configuration naming the expected output and the base ROM to compare against. It is not a suggestion. A mismatch fails the build.

## 41. Render comparison

Capture the same frame from the original and from your build, then diff the images.

Catches wrong field of view, lighting errors, misplaced meshes, animation mistakes, and shader differences. It is faster than reading code for all five, because those are visual problems and this tests them visually.

## 42. Regression test suite

Every behaviour you discover should become a test.

```
Player jump height = 2.84 m
Pistol damage = 20
Door opens after trigger 16
NPC detects player at 14.5 m
```

Then a later change cannot silently break it.

One warning about tests that pass. [Guide 15](15-case-studies-what-each-project-actually-did.md) documents a project reporting 65 of 65 checks where the checks measured table agreement and unchanged file hashes rather than whether the game feels right, and where the frame rate figure was the best of several runs rather than an average. A suite that measures the wrong thing is worse than none, because it stops you looking.

## 43. Symbol database

One of the most valuable things you accumulate, because losing it means paying twice for the same work.

```
0x00453120 -> Player_Update
0x00454380 -> Player_Jump
0x00581210 -> Physics_Raycast
0x00621280 -> RenderWorld
```

Record the address, the function name, the subsystem, your confidence in it, notes, references, pseudocode, and any test result that confirms it. The confidence field is the one people skip and later wish they had.

In practice this is a file, not a database. [zeldaret/oot](https://github.com/zeldaret/oot) keeps `undefined_syms.txt` mapping addresses to names as they are discovered, alongside `sym_info.py`, and its documentation guide asks contributors to name functions in the original's own style, so the code reads like the game it came from.

**Where this repo's rules differ.** Those addresses are offsets into a retail binary. [Guide 6](06-rules-legal-and-publishing.md) says to keep them out of released code, because a hardcoded retail address does nothing in your own build and advertises where the code came from, and [IW4L](12-worked-example-rust-rewrite.md) enforces it with an automated check that greps its tracked tree. Keep the symbol database locally and keep it out of the repository. The reasoning is identical to keeping game files out: it is research material, not source.

## 44. Knowledge base

Do not make every person rediscover the same thing.

```
/wiki  /functions  /formats  /assets  /shaders  /maps  /physics  /network  /tests
```

Agents can query this too, which is the practical argument for keeping it in files rather than in someone's head.

[zeldaret/oot](https://github.com/zeldaret/oot) is a good model: a `docs/` directory with a decompilation tutorial, a documentation style guide, compiler notes, retail version tables, and a `Doxyfile` generating reference documentation from the source comments. Its progress is published to a public site, so the state of the project is visible without reading the repository.

## 45. AI-assisted pipeline

The sequential version:

```
Binary
  -> Ghidra or Binary Ninja
  -> Function discovery
  -> Symbol database
  -> AI analysis
  -> Source reconstruction
  -> Compile
  -> Tests
  -> Compare against the original
  -> Fix
  -> Repeat
```

Running several agents in parallel, one per subsystem, all feeding a shared knowledge base, is a natural idea and this repo has no evidence it works. Finished projects report one agent working through many rounds instead. Take the sequential pipeline as the useful part. [Guide 13](13-reverse-engineering-and-the-law.md#doing-this-with-an-agent) explains why one agent that has read decompiled output is structurally a dirty room, and what splitting specification from implementation across two sessions does about that.

## 46. A sensible project layout

What a reconstructed project tends to grow into:

```
/game
├── core/       ├── engine/     ├── renderer/  ├── physics/
├── collision/  ├── audio/      ├── animation/ ├── ai/
├── gameplay/   ├── networking/ ├── scripting/ ├── platform/
├── ui/         ├── world/      ├── assets/    ├── formats/
├── tools/      ├── tests/      └── docs/
```

Nothing here is a rule. It is the shape that keeps assets and formats separate from engine code, which is the separation that stops the project turning into one large tangled crate.

## What "done" actually means

Not "the decompiler produced output."

More like:

```
Executable behaviour understood
Functions identified
Engine architecture reconstructed
Asset formats understood
Meshes load correctly
Textures load correctly
Materials work
Shaders recreated
Collision matches
Physics matches
Animation matches
Audio works
Maps load
Scripts execute
AI behaves correctly
Gameplay systems match
Saves work
Networking works
UI works
Timing matches
RNG matches
Platform APIs replaced
Tools rebuilt
Regression tests pass
Behaviour verified against the original
```

That is a long list, and reading it is the point. It is why scoping down beats working faster: a rewrite that loads the original's assets and runs one level properly is worth more than one that claims everything and desyncs.

## Credits and sources

**Written by [solarfren69420](https://github.com/solarfren69420)**, a member of the Discord who has taken a game apart to this level and wrote this map so other people could see the shape of the job. Forty-six areas, kept in the original numbering so sections can be referred to by number.

Most of what is here is theirs. The additions made when adapting it for this repo were the research cited below, the notes on where this repo's rules differ, and the plain-English rewriting.

They also maintain **[GameDecompLibrary](https://github.com/solarfren69420/GameDecompLibrary)**, a catalog of 304 decompilation projects, tools and source releases, with a published accuracy report and a source link behind every figure it quotes. If you are about to start something, that is the place to check whether it already exists.

Claims checked against public projects, with links in the sections where they appear:

- [zeldaret/oot](https://github.com/zeldaret/oot), [botw](https://github.com/zeldaret/botw), [tp](https://github.com/zeldaret/tp) and [mm](https://github.com/zeldaret/mm), the matching decompilation projects
- [DXVK](https://github.com/doitsujin/dxvk) and [vkd3d-proton](https://github.com/HansKristian-Work/vkd3d-proton) for API translation
- [DirectX Shader Compiler](https://github.com/microsoft/DirectXShaderCompiler) for shader compilation
- [OpenRCT2](https://github.com/OpenRCT2/OpenRCT2) and [OpenMW](https://github.com/OpenMW/openmw) as reimplementations
- [ghidra-mcp](https://github.com/bethington/ghidra-mcp) and [ida-mcp](https://github.com/HexRaysSA/ida-mcp) for agent-driven decompilation
- S3TC and BCn, from the S3 Graphics block compression family included in DirectX 6.0 and OpenGL 1.3

Two things this page cannot tell you. Sections 12 through 17, 21, 22, 28 through 36 and 38 are listed from practice rather than from a specific project, so treat them as a map of what tends to be involved rather than a specification. And the numbers in section 42 are illustrative, not measured from any game.

If you have done a decompile and want to correct or extend this, [open an issue](https://github.com/trevaintdead/ai-game-modding-guides/issues/new). Navigation, audio, cutscenes and localization are the areas least represented in this repo, and a write-up from someone who has been through them would be welcome.

---

<sub>[Spot a mistake? [Edit this page on GitHub](https://github.com/trevaintdead/ai-game-modding-guides/edit/main/guides/17-decompile-system-map.md).](https://github.com/trevaintdead/ai-game-modding-guides/edit/main/guides/17-decompile-system-map.md) &middot; [Open an issue](https://github.com/trevaintdead/ai-game-modding-guides/issues/new) &middot; Part of [AI Game Modding Guides](https://github.com/trevaintdead/ai-game-modding-guides)</sub>
