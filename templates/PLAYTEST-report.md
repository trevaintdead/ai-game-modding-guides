# Playtest report

Use one of these each time you play a build to check it. It's the evidence behind every "works" in your README. Keep them in `playtests/` or paste them into `MODLOG.md`.

The agent can't watch the game for you ([guide 5](../guides/05-testing-and-troubleshooting.md)). A short report lets other people check your "works" instead of taking it on trust.

## Template

```markdown
# Playtest [date] [build or commit]

## Setup
- Build / commit: [hash]
- Host: [game + exact version], loader [version]
- Guest: [game + exact version], loader [version]
- OS, GPU, translation layer: [...]
- Settings that matter: [resolution, MSAA, frame cap, fullscreen/windowed]
- Save used: [new game / named save / test world]

## What I tested
| # | Scenario | Expected | What happened | Pass / fail / not tested |
|---|----------|----------|---------------|--------------------------|
| 1 | | | | |

## Measurements
- Frame rate host / guest: [numbers, and how measured]
- Anything timed: [what was measured, start and end points]

## Logs and captures
- [file names or pasted lines; screenshots only if they show no other people's names or private chats]

## Not tested this time
- [be specific: multiplayer, other GPUs, other game versions, loading saves...]

## Verdict
[One or two sentences. "Works on my machine for scenarios 1 through 4" is a fine verdict.]
```

## What counts as tested

| Wording | Means |
|---------|-------|
| Tested in game | You played this exact build and saw it work |
| Tested with a fake host or guest | Only the transport or one side was exercised |
| Built | It compiles. Nothing more |
| Creator reports | Someone else says it works; you didn't check |
| Not tested | Say so. It's useful information |

A green test run isn't a playtest. One project's save test passed even though the game data it was meant to load wasn't there. Another project's network test only checks that a reply contains one word.

## Completion check

A playtest report is complete when someone else could repeat it: the build, versions, settings, save and steps are all there, and every scenario says pass, fail or not tested.

## Privacy

Crop or skip screenshots that show other people's usernames, DMs or private servers. Don't paste account names or your home folder path into public logs.
