# 10. Posting Your Project

You've got something that runs. Now you want people to find it, and you don't want your project taken down over a rule you didn't know about.

## What to post, and where

Post to **#share-your-projects** on the Discord so the right people see it. Half-working projects are welcome there, not only finished ones.

Put your project on **GitHub** and link the repo if you want other people to be able to use it. A repo is strongly recommended. A direct download link isn't.

### Never post these

- Ripped assets, textures, models, sounds, or maps
- Leaked or decompiled game code
- Game files of any kind
- Links to pirated or leaked material
- Direct file hosts or download links instead of a repo

If your project needs game content, it reads it from the player's own install. That's the rule, and it's why every example project has an extractor or a setup script instead of a data folder.

## Before you post: the pre-flight

Go through this list. It takes five minutes and prevents the problems that come up most.

- [ ] `git status` is clean and there's no uncommitted game data
- [ ] Repo is scanned for large files: `git ls-files | xargs du -h | sort -rh | head -20`
- [ ] `.gitignore` is a **whitelist**, ignoring everything and including only source
- [ ] `git log --all --stat` shows no assets were ever committed
- [ ] You credited every project you built on, with a link
- [ ] `THIRD-PARTY-NOTICES.md` exists if you reused code
- [ ] README states the games and **exact versions** it needs
- [ ] README states what works and what doesn't
- [ ] README says it's an unofficial fan project
- [ ] README mentions you used AI
- [ ] Any release zip was checked for game files
- [ ] It was tested on a clean machine, not only your own

Two of those catch the most common problems. `git log --all --stat` is the one people skip, and it's the only way to find an asset that got committed three weeks ago and then deleted. `git ls-files | xargs du -h | sort -rh | head -20` catches a 400 MB texture someone added before setting up the whitelist.

### If you already committed game files

Git history is public the moment you push. Do this:

1. Rotate anything sensitive first.
2. `git filter-repo --path path/to/bad --invert-paths`
3. Force-push: `git push --force`
4. Ask GitHub Support to garbage-collect the old objects. They only become unreachable, not gone, until then.
5. Delete and re-upload any release zip that contained them.

Assume anything ever pushed was copied. Don't rely on a history rewrite alone.

## The README is the post

Most people read the README and nothing else. Structure yours like this:

```markdown
# [Project Name]

One sentence: what it does and what makes it different.

![Screenshot or short GIF](docs/screenshot.png)

## What works
- Feature
- Feature
- Feature

## What doesn't work yet
- Feature that's half-done
- Anything untested
- Known bugs

## Requirements
- Game A: version X.Y.Z (Steam / GOG / other)
- Game B: version X.Y.Z
- [Loader](link) for Game A
- Single-player / offline only

## How to install
1. Install both games and the loader.
2. Build: `setup.ps1` or `./gradlew build`
3. Copy the output into [folder].
4. Run Game A.

## How to play
Short, concrete steps.

## How it works
A few paragraphs, or link docs/DESIGN.md.

## Credits
- [SkyCraft](link): the design this is based on
- [Everyone else who helped]

## Legal
Unofficial fan project. Not affiliated with or endorsed by the publisher.
No game assets are included. Players supply their own copies.

Built with AI coding agents.
```

## Make the repo look trustworthy

New projects with no stars and no commits get ignored. These things help:

- **A screenshot or a GIF.** Usually the difference between a click and a scroll. Notepad-draw something if you have to.
- **A commit history.** Twenty small commits reads as "someone who works carefully." One giant commit reads as "paste."
- **A MODLOG.md.** Shows you test what you claim. OWCraft keeps one and links it from its README, which is the cheapest possible signal that you actually test.
- **An honest "what doesn't work" section.** This builds more trust than a list of features, and it saves you the support questions.
- **Tell people to back up their saves.** The early projects in this space all say it, and they're right.

## The forum post

Keep it short. The README does the detail.

```
**Title:** [Game A] + [Game B] passthrough mod

**Games:** [Game A] v[version] + [Game B] v[version]
**Repo:** [link]
**Needs:** both games installed, plus [loader] for [Game A]

**What works:** [2-3 bullets]

**What's broken:** [be honest]

**Tested on:** Windows [version], GPU [model]
**Tested how:** [installed on a clean machine / only on my own PC]
```

The last line matters more than it looks. Saying you only tested on your own machine tells the reader exactly how much to trust the post, and it saves you a support thread where someone discovers a problem you could have warned them about.

### Tags and format

Use the tags and the post template from the pinned guidelines on the Discord. If you skip the template your post is harder to read and gets less help.

### After you post

- Stay in the thread. Most "it doesn't work" reports are a version mismatch or a missing loader, and you can answer in one line.
- Ask for logs, not descriptions: "I need your log from both games, not what it looked like."
- When someone reports it works for them but not for you, that is a bug report worth chasing.

## Where else to publish

- **GitHub Releases:** ship a build as a zip. Check the zip contents for game files first. Many of these projects do this.
- **Nexus Mods / ModDB:** external mod sites with their own rules. Most require that users supply their own game files.
- **Steam Workshop:** if your game supports it. Same rule: no copyrighted game content in your upload. Ask your agent to write an extractor players run themselves rather than shipping assets.
- **A thread on the Discord.** Link the repo. Don't repost the whole thing.

## Anti-cheat and online games

Not negotiable, and not a legal grey area: **single-player and offline games only.** Mods for games with anti-cheat get people banned, and AI agents won't help you circumvent it. The community's own tooling refuses these too.

If your game has both an online and an offline mode, target the offline mode.

## If a rights holder contacts you

Remove it. That's the right call whether or not another project bothers to say so in its README, and gang-beasts-rust does. Credit and links to the original work help, but they aren't a licence to keep shipping someone's assets.

## The post-checklist

- [ ] Passed the pre-flight list
- [ ] Repo link, not a download link
- [ ] Games and exact versions stated
- [ ] Screenshot or GIF
- [ ] "What doesn't work" section is honest
- [ ] Credits in the README
- [ ] Credits on the post itself
- [ ] Used the template and tags
- [ ] Said it's a fan project
- [ ] Mentioned AI use
- [ ] Told people to back up saves
- [ ] Ready to answer questions in the thread

---

<sub>[Spot a mistake? [Edit this page on GitHub](https://github.com/trevaintdead/ai-game-modding-guides/edit/main/guides/10-posting-your-project.md).](https://github.com/trevaintdead/ai-game-modding-guides/edit/main/guides/10-posting-your-project.md) &middot; [Open an issue](https://github.com/trevaintdead/ai-game-modding-guides/issues/new) &middot; Part of [AI Game Modding Guides](https://github.com/trevaintdead/ai-game-modding-guides)</sub>
