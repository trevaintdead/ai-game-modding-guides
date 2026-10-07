# 6. Rules, Legal, and Publishing

This is not legal advice. It's what the Discord requires and what the example projects do. The full posting walkthrough is in [guide 10](10-posting-your-project.md).

## The golden rule: no game files in your repo

Your repository holds **your code only**. Never commit:

- game assets (models, textures, sounds, fonts, maps, shaders)
- game files or folders
- decompiled code or Ghidra databases
- files extracted from the game
- Minecraft assets (they come from the player's own copy at runtime)

Players supply their own copies. The example projects handle this in a few ways:

- **Setup that builds from the player's copies.** GTA San AnSkateas ships a setup script that builds what the mod needs from the player's own installs.
- **Extractor tools.** gang-beasts-rust has Python tools that read the player's install and write extracted data to a folder that Git ignores. Ask your agent to write one of these for you.
- **Reading at runtime.** IW4L and benilla read the game's files in place and never copy them.

### Use a whitelist `.gitignore`

A normal `.gitignore` lists what to leave out. A **whitelist** `.gitignore` ignores everything and lists only what to include. That way an extracted file can never be committed by accident. gang-beasts-rust does this. Ask your agent to set it up on day one.

The template in [`AGENTS-starter.md`](../templates/AGENTS-starter.md) includes this as a hard rule.

## Single-player and offline only

- **Don't inject code into an online client.** Kernel and user-mode anti-cheat are a stop sign: Easy Anti-Cheat, BattlEye, Vanguard, EA Javelin, Ricochet, ACE, nProtect, XIGNCODE and mhyprot. You can get banned. Members report that Claude won't help circumvent anti-cheat, and the universal-modder toolkit also limits itself to single-player or offline games and stays away from anti-cheat.
- If your game has an online mode, work in its single-player or offline mode only.
- Tools running next to a protected game can trip its anti-cheat even when you never touch it. Close the game before a reverse engineering session.

## Don't automate the person's keyboard

This one gets forgotten because it's not about the game's rules, it's about whoever is sitting at the machine.

Automation that drives mouse and keyboard takes over their input. Ask before starting a long automated session while they're at the PC, and check the window is idle first.

Ask before any of these, too: installing a loader into a game folder, changing registry or graphics settings, deleting anything, or publishing on their behalf.

And don't kill processes with a wildcard matcher. `pkill -f` matches your own shell. Kill by exact process ID.

This is the rule people break most often, usually by accident. If someone asks you to add an online game's content to a project, that's the line. See the "Ideas that don't work" table in [guide 2](02-passthrough-mods.md).

## Reverse engineering: what's fine and what's not

Reverse engineering is a normal part of this work, and [guide 3](03-rust-rewrites-and-ports.md) covers the tooling. The law behind these lines is in [guide 13](13-reverse-engineering-and-the-law.md), and what to do if a publisher contacts you is in [LEGAL.md](../LEGAL.md). The lines are:

**Fine:**
- Studying a game you own, on your own machine, for your own use
- Using decompilers and format documentation to understand file formats
- Publishing your *findings* as documentation, which is how OpenMW and OpenRCT2 exist
- Building extractors so other players read their own copies
- Extracting a game from your own disc or your own dump, so a version-matching tool can reach the build it needs

**Not fine:**
- Getting around DRM, activation, or copy protection
- Circumventing anti-cheat
- Redistributing extracted assets, decompiled source, or game data
- Downloading an ISO or dump from a file-sharing site
- Making a tool whose purpose is to bypass access controls

A version downgrader sits on the fine side. It exists so a copy you own reaches the build a mod was written against, and it does nothing to the protection on the disc.

The general principle: when a loader checks that you own the game, satisfy the check the intended way. tModLoader refuses to start unless its free companion app is in your Steam library, and the answer is to add that app, not to patch the check. If a tool only works by disabling DRM or defeating an ownership check, that's the line.

There is a legal reason this rule is drawn where it is, and it is worth knowing because it covers more cases than the tModLoader example. Circumventing a technological protection measure is a **separate** infringement from copying, and it stands on its own. A mod can be entirely lawful and still unlawful to install if getting it in means getting past protection. Using a loader the publisher ships, or that the community maintains openly, is a different act from patching a check out of an executable.

**One more rule that catches people out**, and it is not about DRM at all. Courts treat **identical bugs and identical dead code** as the strongest available evidence that you copied rather than wrote independently. Two people implementing the same specification do not independently arrive at the same broken edge case. If your code and the original share a pointless quirk, you copied it. Rewrite those on purpose and leave a comment saying why.

That one matters more with an agent than with a person, because an agent working from decompiled output reproduces structure faithfully, pointless parts included. [Guide 13](13-reverse-engineering-and-the-law.md#doing-this-with-an-agent) covers how to prompt around it.

Takedowns happen even without shipping assets. Take-Two had GitHub remove re3 and reVC, the reverse-engineered GTA III and Vice City code, and later sued the authors. Activision sent a cease-and-desist to the H2M mod the day before its launch. Garry's Mod removed twenty years of Nintendo-related Workshop content after a takedown request. Keeping game files out of your repo is necessary, not sufficient.

If that happens to you, there is a formal route rather than just deleting and hoping, and it has deadlines and perjury attestations in it. Read it in [LEGAL.md](../LEGAL.md#if-your-repository-gets-a-takedown) before you reply to anyone, and get a lawyer before you file anything.

If you're unsure where a line is, ask. Nobody gets in trouble for asking first.

## Online play and anti-cheat

Single-player and offline, always. Two things worth being precise about, because both come up:

- **Anti-cheat isn't a blanket ban.** Some games with anti-cheat allow modding offline through the game's own option. Rocket League is the common example: Easy Anti-Cheat is required for online play and mods don't run while it's on, but with it off, offline matches, training, LAN, and replays work with mods. Anything online is still out.
- **Never publish anything that helps someone bypass anti-cheat.** Not a tool, not a config, not instructions.

## Credit and licenses

- **Credit every project you build on or learn from.** FalloutCraft and OWCraft both credit SkyCraft by name and keep its license.
- **Keep their licenses.** Add a `THIRD-PARTY-NOTICES.md` listing what you reused and under which license (OWCraft does this).
- **Pick a license for your own code.** MIT is common among these projects. Without a license, others can't legally reuse your code.
- **Say it's an unofficial fan project** and not affiliated with the game's developer or publisher.
- **Say you used AI.** Several example projects have an honest note about it. It helps people judge the project and trust it. Undisclosed AI-generated releases get reacted badly to, and some communities ban them outright.
- **Say what's finished and what isn't.** Test before you claim something works.
- **Don't ship decompiler output.** Names like `FUN_` or `sub_` in a released mod tell a reviewer the code was decompiled rather than written. Rename them.
- **Keep retail offsets out.** A hardcoded address from the original executable does nothing in your own build and advertises where the code came from.

If a rights holder asks you to change or remove something, do it. gang-beasts-rust says this in its README, and it's the right default whether or not another project bothers to.

## Publishing on the Discord

#share-your-projects has rules:

- **A GitHub repo link is recommended** if you want others to use your work, but it isn't required. Don't upload files or link direct downloads or file hosts.
- No ripped assets, leaked code, or links to pirated or leaked material.
- Use the tags and the template from the pinned guidelines post.
- Say what games and versions your project needs.
- Credit what you built on.

[Guide 10](10-posting-your-project.md) has the full posting guide and a pre-flight checklist.

## Other places to publish

- **Steam Workshop and similar:** Make sure nothing in your upload is copyrighted game content. Asking your agent to write an easy asset extractor for players to run beats shipping assets. Check each platform's own rules.
- **Releases on GitHub:** Many projects ship a zip on their Releases page. Check that it doesn't contain game files before you publish it.

## Checklist before you publish

- [ ] My repo has no game files, decompiled code, or extracted assets
- [ ] I use a whitelist `.gitignore`
- [ ] I credited every project I built on and kept their licenses
- [ ] My README says what games and versions it needs
- [ ] My README says what works and what doesn't
- [ ] My README says it's an unofficial fan project and mentions AI use
- [ ] My release zip (if any) contains none of the game's files
- [ ] I tested it on a clean setup

## If you already pushed something you shouldn't have

Git history is public the moment you push. Full recovery steps are in [guide 10](10-posting-your-project.md#if-you-already-committed-game-files). Assume anything pushed was copied. A history rewrite alone doesn't remove it from anyone who already cloned.

---

<sub>[Spot a mistake? [Edit this page on GitHub](https://github.com/trevaintdead/ai-game-modding-guides/edit/main/guides/06-rules-legal-and-publishing.md).](https://github.com/trevaintdead/ai-game-modding-guides/edit/main/guides/06-rules-legal-and-publishing.md) &middot; [Open an issue](https://github.com/trevaintdead/ai-game-modding-guides/issues/new) &middot; Part of [AI Game Modding Guides](https://github.com/trevaintdead/ai-game-modding-guides)</sub>
