# MODLOG

A running log of what changed and what was tested. Ask your agent to add an entry after every change. It helps you, helps a fresh chat catch up, and helps anyone reading your project see what's real.

Newest entries go at the top.

## Entry format

```markdown
## [Date] [short title]

**Changed:** what was changed, and in which files
**Why:** the problem or goal
**Tested how:** what you did to check it (played the game, read logs, ran a test)
**Result:** what happened, with numbers or log lines where possible
**Still broken / not tested:** be honest
**Next:** what to do next
```

## Example

```markdown
## 2026-10-03 Player position sync

**Changed:** Added position messages from the gameplay game to the host plugin (`mod/LinkReader.java`, `plugin/link.cpp`)
**Why:** Step 2 of the plan: send one piece of data between the games
**Tested how:** Started both games, walked around, compared the position each side logged
**Result:** Positions match within about one frame at normal walking speed
**Still broken / not tested:** Fast travel and loading screens not tested; no rotation yet
**Next:** Send collision shapes the other way
```

That example sends position from the gameplay game to the host. SkyCraft works the same way, where Minecraft is authoritative for the player and the host supplies collision. Decide which side owns the player in step one, then stay consistent.

## Tips

- "Tested" means you or the agent actually ran it. If it wasn't run, write "not tested."
- Link to the log file or paste the key lines.
- Keep it short. A line or two per field is plenty.
- **Log the failures too.** "Tried file-based transport, Windows locks the file, switched to shared memory" is the most useful kind of entry, because it's the one that stops someone else repeating it.
- Ask the agent to add the entry itself: `Add a MODLOG entry for what you just did.`

## Why bother

- **For you:** you stop guessing what you already tried.
- **For a stuck chat:** it beats handing over the whole chat. See [`STATUS-handoff.md`](STATUS-handoff.md).
- **For readers:** it's the closest thing to proof that the project is real and tested. Members notice this, and it's what separates a serious project from a vibe-coded one.
- **For you later:** when you come back in six months, you'll want to know why you made a decision.

OWCraft keeps one and links it from its README. Keeping yours visible is the whole point.

Keep it in your repo. It's cheap and it compounds.
