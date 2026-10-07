# Attribution and lineage

Most projects here build on someone else's: [FalloutCraft](https://github.com/zeyvu/FalloutCraft), [OWCraft](https://github.com/Yaekai/OWCraft) and [LibertyCraft](https://github.com/mrborghini/libertycraft) all start from SkyCraft, and many rewrites start from earlier reverse-engineering work. Saying exactly what you inherited and what you added is good manners, and the licence often requires it.

Copy this to `CREDITS.md`, or add it as a section of your README.

## Template

```markdown
# Credits and lineage

## Built on
| Project | Author(s) | Licence | Upstream commit or version we started from | What we use |
|---------|-----------|---------|--------------------------------------------|-------------|
| [SkyCraft](https://github.com/chasmlol/SkyCraft) | chasmlol | [check] | [commit hash or release] | [e.g. Fabric mod, protocol header] |

## What's new in this project
- [The host adapter, the renderer integration, the combat bridge...]

## What we changed in inherited code
- [file or area]: [what and why]

## Inherited text we haven't checked yet
- [e.g. "docs/DESIGN.md still describes the upstream game in places"]

## Tools and AI
- Agent and model: [e.g. Claude Code, Opus x.y], used for [which parts]
- Other tools: [Ghidra, ReShade, ...]
- Human work: [what you did yourself]

## Game content
No game files are included. Players use their own copies of [games].
```

## Completion check

- Every upstream project is linked, with its licence and the exact commit or release you started from.
- New work and inherited work are listed separately.
- AI use is described for this project only. Don't claim upstream work was AI-made, or human-made, unless its own author says so.

## Common mistakes

- **No starting commit.** LibertyCraft's history says it forked SkyCraft, but its first commit has no parent, so the exact upstream version it started from can't be recovered. Write it down on day one.
- **Inherited docs left as if they were yours.** A forked design document can still describe the original game. Mark it or update it.
- **Treating a new file name as new code.** Moving upstream helpers into a new file doesn't make them yours.
- **Removing someone's licence notice because it mentions another game.** Keep it.
- **Borrowing someone else's credits.** A fork's history contains the upstream's commits. FalloutCraft's history includes SkyCraft commits co-authored with an AI model; those describe SkyCraft work, not the later Fallout-specific changes. Credit tools for your own commits only.
- **One licence for everything.** Your code, inherited code, generated tables, downloaded tools and the player's game files can each have different terms. [BullySkate](https://github.com/Faiqie/BullySkate)'s notices, for example, list the Skate rewrite, the loader SDK and an audio port separately.

## Privacy

Credit people by the name they publish under. Don't add real names, Discord handles from private servers, or screenshots of private chats unless the person has agreed.
