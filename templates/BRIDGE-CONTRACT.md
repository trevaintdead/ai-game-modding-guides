# Bridge contract

For any project where two games (or a game and a rebuilt engine) share state. Copy it to `docs/CONTRACT.md` before the first line of bridge code, and ask the agent to check every change against it.

Most of the hard bugs in [guide 16](../guides/16-ownership-sync-and-rendering.md) come from something that was never written down: who owns the player, which units a number is in, which frame a camera pose belongs to, or what happens when one game pauses.

## Prompt to get the agent to fill it in

```
Fill in docs/CONTRACT.md from the template. Use only what you can confirm from the
code and the games' docs. Where something isn't decided yet, write "UNDECIDED" and
list it at the bottom. Don't change any code.
```

## Template

```markdown
# Bridge contract

## Route
[Live passthrough / frame compositing / geometry transfer / shared simulation /
engine recreation / asset conversion / subsystem recreation, or a mix. See guide 14.]

## Programs and exact versions
- Host: [game, store, exact version or build], loader [name + version]
- Guest: [game or engine, exact version], loader [name + version]
- OS / translation layer: [Windows 11 / Proton x.y / CrossOver x.y]

## Ownership
| Thing | Owner | How control is handed back, and how it returns |
|-------|-------|------------------------------------------------|
| Player position and physics | | |
| Camera | | |
| World geometry | | |
| Collision (host → guest) | | |
| Collision (guest → host) | | |
| NPCs / enemies | | |
| Damage and health | | |
| Inventory | | |
| Saves | | |
| Menus, pause, loading screens | | |
| Cutscenes, vehicles, furniture, scripted events | | |

## Units and axes
- Distance: [e.g. 70 host units = 1 guest block]
- Axes: [e.g. host x-east, y-north, z-up → guest x, z, -y]
- Angles: [degrees/radians, handedness]
- Health/damage: [e.g. guest 20 points = host 100]
- Time: [host frame rate, guest tick rate]

## Messages
| Channel | Kind (snapshot / queue / image) | Direction | Rate | What happens if full or late |
|---------|---------------------------------|-----------|------|------------------------------|
| | | | | |

- Protocol name, magic and version: [...]
- Byte order: [...]
- How each side detects the other restarting: [heartbeat, process ID, generation]

## Frames (only if images cross)
- Layers sent: [colour, depth, HUD, hand]
- How a camera pose is matched to its image: [...]
- What the host shows if no matching image arrives: [...]
- Maximum resolution and fallback: [...]

## Lifecycle
- Start order: [...]
- On pause / menu: [...]
- On loading a save or changing area: [...]
- On death: [...]
- On disconnect or crash of either side: [...]

## Not covered (be honest)
- [Systems this bridge doesn't touch, and what the player will see]

## UNDECIDED
- [...]
```

## Completion check

The contract is done when:

- every row in the ownership table has an owner and a hand-back rule, or says "not bridged";
- every number that crosses has a unit;
- every channel says what happens when it's full or late;
- someone other than the agent has read it.

## Examples of filled-in rows

These are real choices from projects described in [guide 15](../guides/15-case-studies-what-each-project-actually-did.md):

| Thing | Example owner and rule |
|-------|------------------------|
| Player position | Minecraft owns it; Skyrim takes over for furniture, mounts and kill-moves, then hands back ([SkyCraft](https://github.com/chasmlol/SkyCraft)) |
| Player position | GTA owns it on foot; Minecraft owns it in elytra flight (Minecraft × GTA V example) |
| Distance | 40 Half-Life units per block ([Minecraft × Half-Life](https://github.com/SawyerTheNerd/Minecraft-X-HalfLife)) |
| Damage | Protocol amount is native HP (Monster Hunter bridge); protocol amount is Minecraft damage, converted by target max HP (Elden Ring bridge) |
| Late frame | Prefer the exact matching image, then an older one, then the last uploaded image; count each case ([CrossOver bridges](https://github.com/justbustin/minecraft-crossover-bridge)) |
