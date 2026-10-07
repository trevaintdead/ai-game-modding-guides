# 16. Ownership, Sync and Rendering: Symptoms and Their Usual Causes

When two games share a screen, most bugs fall into a few families: who owns something, whose clock a value came from, which units it's in, and what the renderer already did before you drew. This guide lists symptoms people actually hit, the cause each project found, and what they did.

Use it with [guide 5](05-testing-and-troubleshooting.md). Paste the matching section into your agent when you report a bug; it gives the agent a cause to check instead of a guess.

> **Evidence level.** Each "found in" points to a real project's code, commits or notes. Where something is a likely cause rather than a confirmed one, it says **hypothesis**.

## Before anything else: write the contract

Most of the bugs below come from something nobody wrote down. Fill in [`templates/BRIDGE-CONTRACT.md`](../templates/BRIDGE-CONTRACT.md) at the start: who owns what, units and axes, tick rates, message layout, version, and what happens on pause, load, death and restart. Then ask the agent to check every change against it.

## Rendering and depth

**The guest world slides or lags behind the host when you turn.**
- *Usual cause:* the host camera pose was read at a different moment from the image that was shown.
- *Found in:* [NewVegasCraft](https://github.com/Davozh/new-vegascraft) moved its pose read to when the frame is presented. The [CrossOver bridges](https://github.com/justbustin/minecraft-crossover-bridge) keep eight recent poses and match each guest image to the pose it was rendered with.
- *Check:* log a frame counter or timestamp with every pose on both sides and compare them. Don't eyeball it.

**It still looks like drift after the timing is fixed.**
- *Possible cause:* it isn't drift. NewVegasCraft's last "drift" was a real Minecraft block intersecting a sign.
- *Check:* add a debug key that drops a marker at the host crosshair and a view that shows the difference between host and guest depth.

**Guest blocks show through walls, or host depth reads as empty.**
- *Usual causes:* the depth buffer is multisampled (MSAA), so it can't be read the simple way; or it was cleared before you copied it.
- *Found in:* NewVegasCraft's real scene depth was 4× MSAA; MSAA had to be off for that setup. It also had to copy depth before the host cleared it. The Monster Hunter: World bridge watches for full-size depth clears and picks between two candidates.
- *Check:* add a debug view that shows host depth on its own.

**Guest blocks disappear behind glass or water.**
- *Usual cause:* you used host depth after transparent objects were drawn into it.
- *Found in:* [LibertyCraft](https://github.com/mrborghini/libertycraft) saves an opaque-only depth copy before GTA IV draws glass, then draws Minecraft against that copy. Native glass then isn't drawn over the guest; that's a known trade-off.

**Guest blocks look flat or don't match the host's lighting.**
- *Cause:* frame compositing can only estimate lighting from the host's image.
- *Found in:* the Monster Hunter bridge multiplies the guest world by blurred host brightness; the Elden Ring bridge adds ambient, haze and a colour tint. [SkyCraft](https://github.com/chasmlol/SkyCraft), which draws inside Skyrim's renderer, samples native fog, ambient and point lights and captured sun-shadow cascades.
- *Choice:* if you need real host shadows on guest blocks, you need geometry transfer (route 3 in [guide 14](14-choosing-a-route.md)), not a pasted picture.

**A stale guest image stays on screen over the pause menu or a loading screen.**
- *Cause:* the host keeps drawing the last uploaded picture when the guest stops sending.
- *Found in:* the Minecraft × GTA V example's author cut around this in the demo; the fix wasn't built. NewVegasCraft hides the guest layer while host menus are open.
- *Fix to ask for:* detect host menus and stop compositing, and drop images older than a set age.

**Torn or mixed frames, flickering layers.**
- *Usual cause:* the reader accepted a frame while the writer was halfway through it.
- *Found in:* the Monster Hunter bridge's frame sequence counter doesn't go odd when a write starts, unlike its comment. The Elden Ring version does. (**Hypothesis** about any visible glitch: no one has shown it causes one.)
- *Check:* ask the agent to check that your "seqlock" really is odd while writing, even when done, and that the reader checks it before and after copying.

**Stutter that gets worse with resolution.**
- *Usual cause:* uploading guest images through a staging texture each frame.
- *Found in:* NewVegasCraft reports 18.7 ms → about 5 ms per guest upload after switching to dynamic textures. [OWCraft](https://github.com/Yaekai/OWCraft) reports Minecraft going from 25 to 60 fps after skipping presentation of the hidden window.

## Movement, clocks and authority

**Movement looks like it steps or jitters.**
- *Usual cause:* sending the raw 20-tick position instead of the interpolated render position.
- *Found in:* SkyCraft sends previous and current physics positions with a timestamp and lets Skyrim interpolate on its own clock. LibertyCraft interpolates a short tick history with an adaptive delay and holds when ticks are missing.

**The player is flung or teleported when entering a new area.**
- *Usual cause:* the guest arrived before host collision for that area did.
- *Found in:* early [ValCraft](https://github.com/LoAlCo/ValCraft) prioritises host regions in the direction of travel, holds the guest at its last position while keeping its momentum, and makes the host puppet kinematic so host physics can't fling it. A two-second timeout then lets movement continue anyway, so this reduces the problem rather than removing it.

**Cutscenes, vehicles or scripted openings get stuck.**
- *Usual cause:* the guest owns movement when the host's script expects to.
- *Found in:* SkyCraft hands control back to Skyrim for furniture, mounts and kill-moves. LibertyCraft hands control to GTA for missions, cutscenes and cars, then re-syncs the guest with a teleport handshake. SkyCraft's notes warn that the Helgen cart opening can get stuck and suggests Alternate Start or a save after Helgen.
- *Rule:* for every host system you don't bridge, decide how control goes back and how it returns.

**Two people or programs fight over the same setting.**
- *Found in:* NewVegasCraft only restores the host's "fighting disabled" flag if it was the one that set it. That still doesn't handle another mod changing the same flag later.

## Collision

**Host NPCs walk through guest blocks, but you can't walk through host walls.**
- *Cause:* collision has a direction. Sending host geometry to the guest doesn't send guest blocks to the host.
- *Found in:* NewVegasCraft samples host terrain into guest barriers, but host actors pass through guest blocks. [FalloutCraft](https://github.com/zeyvu/FalloutCraft) turns guest solid blocks into native Havok boxes for builds, and its NPCs collide with them but don't path around them. Physical blocking and AI navigation are separate jobs.

**Barriers stay behind after a door or vehicle moves.**
- *Usual cause:* cached collision columns aren't refreshed for moving geometry.
- *Found in:* NewVegasCraft's cached columns stay until reset. The Half-Life bridge marks the old and new regions of a moving brush as dirty, but only flushes updates when its worker is idle.

**Blocks get placed inside rocks or signs.**
- *Found in:* NewVegasCraft first built a two-layer barrier skin, then switched to solid columns from terrain to the highest surface (capped at about 24 blocks), which also fills arches and overhangs.

**Collision rebuilds stall the game.**
- *Seen in some projects:* watch nearby cars, rebuild only when they've moved enough, limit rebuilds to once a second, and skip identical triangle batches by hash.

**"Exact collision" isn't exact.**
- SkyCraft keeps original triangles for the local player but turns capsules and spheres into boxes, unsupported shapes into bounding boxes, and skips some unknown shapes. Name the representation when you write "exact".

## Combat, damage and entities

**Damage numbers are wildly off.**
- *Usual cause:* the two games use different health scales and the bridge doesn't say which.
- *Found in:* the Half-Life bridge maps Minecraft's 20 points to Half-Life's 100 (×5). The Monster Hunter bridge sends native HP; the Elden Ring bridge sends Minecraft damage and converts by the target's max HP. Put the unit in the message.

**Explosions or hits count twice.**
- *Found in:* the Minecraft × GTA V example remembers recent explosion positions for half a second to avoid double counting, and stops player projectiles hitting the stand-in proxies.

**A hit lands on the wrong enemy after one dies.**
- *Possible cause:* the bridge refers to entities by an index that gets reused.
- *Found in:* the Half-Life bridge's entity indices have no generation tag (**hypothesis**: a delayed event could hit a recycled entity). The Monster Hunter bridge adds a serial number to hazard IDs for this reason.

## Loading, timing and other mods

**Your plugin never loads, or loads after the code it needs to hook.**
- *Usual cause:* the loader DLL is loaded too late in startup.
- *Found in:* [PipeLink](https://github.com/Sm1jjj/PipeLinkLauncher) found `dinput8.dll` loaded too late for GTA San Andreas's mod loader and its own plugin. It installed the same loader under the name of a DLL the game imports at startup (keeping the original under a new name), which fixed the order.

**The game on disk passes your checks, but the code in memory is different.**
- *Usual cause:* Steam-wrapped executables are decrypted in memory, so the file on disk doesn't show the real code.
- *Found in:* [BullySkate](https://github.com/Faiqie/BullySkate) launches Bully through Steam, then checks 203 code fingerprints and 19 data locations in the running game before installing any hooks. [Touhou HFR](https://github.com/vittorioromeo/th12_hfr) also requires launching Steam versions through Steam.

**Two mods fight over the same frame or hook.**
- *Found in:* Touhou HFR and a popular rotation wrapper split the job: the wrapper owns render targets, rotation and presentation; HFR owns timing, input and replay, and turns off its own competing scaling. HFR also re-takes its graphics hooks after a translation patch loads, then chains them, because the other patch had been silently replacing them.
- *Rule:* write down which mod owns the picture, the clock and the input, and test the combination you document.

**Raising the update rate changes the game.**
- *Found in:* Touhou HFR runs player movement, bullets and collision in smaller steps than the original 60 per second. Its own docs say this can change hits, grazes and scores, so runs aren't comparable to stock leaderboards. Replays need the extra inputs recorded, and one version wrongly claimed its options switched off during replays.
- *Rule:* if you change timing, say what is no longer comparable with the original.

**A rebuilt guest stalls and you lose button presses.**
- *Found in:* BullySkate's physics worker merges queued input and caps how much time it catches up, so a long stall can drop a button transition. Its sound worker mutes after 250 ms without new state instead of looping.

**Controllers work in one game and not the other.**
- *Found in:* [SkateGM](https://github.com/the-schwilliam/SkateGM) maps PlayStation, Switch and generic pads onto the Xbox-style input the rebuilt Skate engine already expects, then chooses button labels separately. Any connected Xbox pad still takes priority, so "last-used controller" only applies within each kind.

**Automated tests move the camera wildly, or not at all.**
- *Found in:* [HL2-RS](https://github.com/kvalls/hl2-rs) found synthetic Windows input has no raw mouse motion, and absolute mouse coordinates can look like huge relative moves. Real play and automated play can take different input paths; test both.

**Graphics break after another renderer has drawn.**
- *Found in:* HL2-RS sets depth testing and depth writing separately and re-enables depth writes just for a clear, because a previous pipeline had left them off. Save and restore any state you touch when you share a renderer.

## Transport and lifecycle

**The game hitches when the other side is slow.**
- *Usual cause:* a blocking send on the game thread.
- *Found in:* NewVegasCraft's first WebSocket client sent synchronously from the game loop. [Signet](https://github.com/kian-cx/signetprotocol)'s client writes to TCP while holding a lock, and its server broadcasts while holding shared state, so a slow reader can stall it.

**Events go missing.**
- *Found in:* Signet's C interface consumes events even when you call it just to ask how big the buffer should be. SkyCraft drops input events when its ring is full. Decide which events may be dropped and log every drop.

**Everything breaks after one side restarts.**
- *Found in:* SkyCraft and the Half-Life bridge use heartbeats and process IDs to notice restarts; LibertyCraft resends after a restart generation changes. Some rebuilt-engine projects can reload the guest engine without restarting the host, restore the last good pose when the engine produces invalid numbers, and stop if errors repeat within a few seconds.

**It works for you and not after an update.**
- *Usual cause:* producer and consumer file formats drifted apart.
- *Found in:* BullySkate's collision format went from segments to polylines with a schema bump, and the launcher checks the receipt version. Version every file your tools generate.

**The installer broke someone's setup.**
- *Found in:* several installers check only file names, overwrite configs, or delete a config on uninstall that they didn't create. NewVegasCraft's early installer kept existing settings when installing but deleted those same paths on removal. BullySkate's installer is a stronger example: it checks exact asset hashes, keeps timestamped backups, and refuses an unknown existing loader. Even so, none of these copy steps can be rolled back halfway. Ask the agent to back up and restore exactly what it changes, and to check versions or hashes rather than names.

## The debugging habits these projects share

1. Build the debug views and logging before chasing a visual bug.
2. Test the transport with a fake host or a fake guest first. LibertyCraft has stand-ins for both ends. Then test with both real games, because passing a fake test doesn't prove the real one.
3. Write down each theory and remove the ones that turned out wrong.
4. Say which machine, OS, GPU and game version a result came from.
5. Keep the playtest notes in a log. See [`templates/PLAYTEST-report.md`](../templates/PLAYTEST-report.md).

---

<sub>[Spot a mistake? [Edit this page on GitHub](https://github.com/trevaintdead/ai-game-modding-guides/edit/main/guides/16-ownership-sync-and-rendering.md).](https://github.com/trevaintdead/ai-game-modding-guides/edit/main/guides/16-ownership-sync-and-rendering.md) &middot; [Open an issue](https://github.com/trevaintdead/ai-game-modding-guides/issues/new) &middot; Part of [AI Game Modding Guides](https://github.com/trevaintdead/ai-game-modding-guides)</sub>
