# 5. Testing and Troubleshooting

## You do the playtesting

The agent can't watch a game in real time, so it's poor at judging how things look and feel. It may not notice a missing particle effect, a wrongly rotated model, or janky movement. Experienced members handle this two ways:

- **Ask for telemetry.** Have the agent log as much as possible (positions, damage numbers, events, frame times) so it can check its own work without looking at the screen. A useful pattern is having it read damage numbers from the logs while you hit a training dummy.
- **Playtest yourself.** If the agent tries to test visually, stop it and say you'll playtest. You verify faster than it can, either way.

Sorting out UI and menus early also helps, because it makes testing simpler later.

## How to report a problem

"It doesn't work" can't be fixed. This can:

```
I did: [what you did, step by step]
I expected: [what you wanted to happen]
What happened: [what actually happened]
Logs: [paste the relevant log lines, from both games if it's a passthrough mod]

Find the cause before changing any code.
```

## When the agent is stuck in a loop

1. Stop. Repeating the same prompt rarely helps, because the agent takes the same reading of it again.
2. Say what you actually want instead of restating the request. If it kept trying to rewrite everything from scratch when you wanted it to reuse the original's motion graphs, name that. A vague complaint produces a vague fix.
3. Ask it to write a status file: the project state and the exact problem. See [`templates/STATUS-handoff.md`](../templates/STATUS-handoff.md).
4. Start a fresh chat and give it the file. This is the one case where a new chat earns its cost, because you are escaping a loop rather than tidying up.
5. Ask for different approaches, and explain which ones already failed.
6. If it still can't, a smaller goal may help.

Read the status file yourself before sending it. It is the agent's account of what it thinks is happening, and the wrong assumption is usually visible in writing even when it survived twenty turns of conversation.

## Common problems

**"The file is too large" or you're copying code by hand.**
You're probably in a chat website or a non-agent mode. Switch to an agent. See [guide 1](01-choose-and-set-up-an-ai-agent.md).

**The agent can't access my files / Codex keeps failing to get access.**
Check that tool's documentation for permission or sandbox settings. Give it access to your project and game folders only.

**I ran out of usage.**
Plans have a roughly 5-hour reset window and a weekly limit. Stay in one chat rather than starting new ones, since prompt caching is what keeps a long session affordable, and stay on the model you started with. Check your provider's current plans for the actual limits.

**The AI refuses.**
Read the reason. If it's about anti-cheat or online games, the answer is no, and those aren't supported. If it's a single-player mod with your own copy of the game, say that plainly and describe your goal honestly. Don't try to disguise what you're doing, and don't try to get around DRM or anti-cheat.

**The game crashes or my save broke.**
Back up your saves before testing. Several projects warn that they're early and experimental. Roll back using Git. Check the project's known limitations.

**The version doesn't match.**
Mods and extractors are tied to specific game versions. Some games need a downgrade tool to reach the supported version. GTA San Andreas, for example, needs version 1.0 for GTA San AnSkateas. Check the README of the project you're following.

**Antivirus flagged a download.**
Unsigned tools and bundled launchers are sometimes flagged. In one SkyCraft issue, a user reported a Malwarebytes flag on a release zip that didn't show up on a re-scan, and an online scanner showed no detections. It's still worth checking where a file came from and opening an issue on the project.

**The guest world slides, flickers, shows through walls, or the player falls through the floor.**
These have known usual causes: a camera pose from the wrong frame, unreadable or already-cleared depth, collision that only goes one way, or a stale image left on screen. [Guide 16](16-ownership-sync-and-rendering.md) lists them with what each project did.

**It works but it's slow or stutters.**
Expected, and usually fixable. Ask for frame-time logs from both processes and tell it to profile before changing anything. OWCraft's notes say skipping the presentation of the hidden window took Minecraft from 25 to 60 fps.

The gameplay game still has to run its client. It's hidden, not headless: depending on the design it builds the meshes the host draws, or renders the picture the host pastes in, plus the hand and HUD.

Other wins are sending deltas instead of full state, and fixing your update rate. See [guide 9](09-worked-example-passthrough-mod.md#step-8-make-it-not-stutter).

**The mod works for me but not for a friend.**
Check that they have both games, the right versions, and the same loaders installed. Ask them for logs.

**A mod built for one game won't run on my computer's setup.**
Write down your OS, game versions, and tool versions when you ask for help, and include logs. Some projects are only tested on one setup (OWCraft says it was tested on one Windows PC).

## Asking for help

Post on the Discord, or open a GitHub issue. Include:

- the games and their exact versions
- the loaders and their versions
- the agent and model you're using
- what you tried
- the error message or logs

A `STATUS.md` written by the handoff trick in [`templates/STATUS-handoff.md`](../templates/STATUS-handoff.md) already contains most of that. Answers come back quicker, and it works in a Discord thread or a GitHub issue just as well as in a fresh chat.

For anything you tested, a [`PLAYTEST-report.md`](../templates/PLAYTEST-report.md) says which versions and settings you used and what you didn't test.

---

<sub>[Spot a mistake? [Edit this page on GitHub](https://github.com/trevaintdead/ai-game-modding-guides/edit/main/guides/05-testing-and-troubleshooting.md).](https://github.com/trevaintdead/ai-game-modding-guides/edit/main/guides/05-testing-and-troubleshooting.md) &middot; [Open an issue](https://github.com/trevaintdead/ai-game-modding-guides/issues/new) &middot; Part of [AI Game Modding Guides](https://github.com/trevaintdead/ai-game-modding-guides)</sub>
