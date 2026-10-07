# AI Game Modding Guides

Guides for two kinds of projects, both built with an AI coding agent via a harness:

- **Passthrough mods:** two games running at once and linked together, like [SkyCraft](https://github.com/chasmlol/SkyCraft) (Minecraft inside Skyrim).
- **Rust rewrites and ports:** rebuilding a game's engine in Rust so it reads data from your own copy, like [IW4L](https://github.com/vladtrc/iw4L).

These guides answer the questions people asked on the Discord. If something is missing or wrong, [open an issue](CONTRIBUTING.md) or send a pull request.

Worth saying up front: everyone has their own methods, prompting style, and workflow. We can't cover everything, so take the methods in these guides with a grain of salt and build your own from them.

[![Licence: MIT](https://img.shields.io/badge/Licence-MIT-green.svg)](LICENSE)
[![Contributing welcome](https://img.shields.io/badge/contributions-welcome-brightgreen.svg)](CONTRIBUTING.md)
[![Legal notice](https://img.shields.io/badge/Legal-notice%20and%20takedown%20process-blue.svg)](LEGAL.md)
[![Discord](https://img.shields.io/badge/Discord-join%20the%20server-5865F2?logo=discord&logoColor=white)](https://discord.gg/ccFpNC26Ts)

> **Status:** draft. Tools, models, plan limits, and mod loaders change fast. Verify a detail before you rely on it.
>
> **Single-player and offline games you own only.** Nothing here covers anti-cheat, DRM, or online play. See [the rules](guides/06-rules-legal-and-publishing.md).
>
> **Before you publish anything**, read [LEGAL.md](LEGAL.md). It covers what this repository does and does not cover, and what to do if a publisher contacts you or your repo gets taken down. None of it is legal advice.
>
> **Platform.** Most passthrough mods target the Windows build of a game, and they aren't guaranteed to work under Wine or Proton. A few creators report running them that way (LibertyCraft on Linux, the CrossOver bridges on macOS); see [guide 8](guides/08-mod-loaders-and-script-extenders.md#windows-is-the-common-denominator). If you've gotten one running, please tell us.
>
> Rust rewrites are cross-platform: the project just has to be built for your OS. IW4L documents Linux and macOS build steps.

## Start here

Never done this before? Read **[Start here](guides/00-start-here.md)**.

Have a question you haven't seen answered yet? Go to the **[FAQ](guides/07-faq.md)**.

## The short version

1. **Use an AI agent through a harness, not a chat website.** A harness (Claude Code, Codex, OpenCode, and others) runs on your PC (using the API provided by your AI provider), reads your game folders, writes and edits files, and runs builds. The browser versions of AI do not have access to files on your PC, so it is much easier to use the Agent through a harness.
2. **Check whether your game has a mod loader.** This decides whether your idea is realistic. See [guide 8](guides/08-mod-loaders-and-script-extenders.md).
3. **Install the games first.** The agent finds the files itself, so you don't upload anything.
4. **Point it at an example project** ([SkyCraft](https://github.com/chasmlol/SkyCraft) for passthrough, [IW4L](https://github.com/vladtrc/iw4L) for rewrites) and tell it what you want.
5. **Expect many rounds.** The first prompt rarely finishes the job. You playtest, report what happened, and the agent fixes it.
6. **Never commit game files.** Your repo holds your code only. Players use their own copies.
7. **Agents usually leave DRM and anti-cheat alone**, and online-only games are a no-go. We don't condone circumventing agent guardrails, piracy, or DRM. Some games with anti-cheat do allow offline modding through the game's own option; see [the rules](guides/06-rules-legal-and-publishing.md#online-play-and-anti-cheat).

## Guides

| # | Guide | Read it if |
|---|-------|-----------|
| 0 | [Start here](guides/00-start-here.md) | You have never done this before |
| 1 | [Choosing and setting up an AI agent](guides/01-choose-and-set-up-an-ai-agent.md) | You don't know which tool or model to use, or what it costs |
| 2 | [Passthrough mods](guides/02-passthrough-mods.md) | You want to link two games |
| 3 | [Rust rewrites and ports](guides/03-rust-rewrites-and-ports.md) | You want to rebuild a game's engine |
| 4 | [Prompting and workflow](guides/04-prompting-and-workflow.md) | You want to know what to say and how to keep a project on track |
| 5 | [Testing and troubleshooting](guides/05-testing-and-troubleshooting.md) | Something broke, or the AI is stuck |
| 6 | [Rules, legal, and publishing](guides/06-rules-legal-and-publishing.md) | Before you share anything |
| 7 | [FAQ](guides/07-faq.md) | Quick answers to the most common questions |
| 8 | [Mod loaders and script extenders](guides/08-mod-loaders-and-script-extenders.md) | You need to know what you can install, or whether your idea is possible |
| 9 | [Worked example: a passthrough mod, start to finish](guides/09-worked-example-passthrough-mod.md) | You want the whole process with the actual prompts |
| 10 | [Posting your project](guides/10-posting-your-project.md) | You've got something that runs and want people to use it |
| 11 | [Models and what to spend](guides/11-models-and-cost.md) | You're deciding what to pay, or which model to point the agent at |
| 12 | [Worked example: IW4L, an AI-assisted Rust rewrite](guides/12-worked-example-rust-rewrite.md) | You want an honest Rust rewrite case study |
| 13 | [Reverse engineering and the law](guides/13-reverse-engineering-and-the-law.md) | You're decompiling something and want to know where the lines actually are |
| 14 | [Choosing a route](guides/14-choosing-a-route.md) | You're not sure whether your idea is passthrough, compositing, a rewrite or something else |
| 15 | [Case studies: what each project actually did](guides/15-case-studies-what-each-project-actually-did.md) | You want versions, ownership and failures from real projects |
| 16 | [Ownership, sync and rendering](guides/16-ownership-sync-and-rendering.md) | Something slides, flickers, falls through the floor or desyncs |

## Legal

[LEGAL.md](LEGAL.md) covers what this repository is and is not, the behaviours that actually lower your risk, what to do if a publisher contacts you, and the formal DMCA counter-notification process if your repository gets taken down.

Read it before you publish, not after a letter arrives. None of it is legal advice.

## Templates

Drop these into your own project.

- [`templates/AGENTS-starter.md`](templates/AGENTS-starter.md): a rules file that makes the agent follow your rules every session
- [`templates/STATUS-handoff.md`](templates/STATUS-handoff.md): the note you give a fresh chat when the old one gets stuck
- [`templates/MODLOG-template.md`](templates/MODLOG-template.md): a running log of what changed and what was tested
- [`templates/workflow-writeup.md`](templates/workflow-writeup.md): for sharing how you made your project
- [`templates/BRIDGE-CONTRACT.md`](templates/BRIDGE-CONTRACT.md): who owns what, units, messages and lifecycle, written before the bridge code
- [`templates/PLAYTEST-report.md`](templates/PLAYTEST-report.md): what you tested, on which versions, and what you didn't
- [`templates/ATTRIBUTION-and-lineage.md`](templates/ATTRIBUTION-and-lineage.md): what you inherited, from which commit, and what's new

## Examples worth studying

| Project | What it shows |
|---------|---------------|
| [SkyCraft](https://github.com/chasmlol/SkyCraft) | Passthrough: Minecraft inside Skyrim (script extender plugin + Fabric mod) |
| [FalloutCraft](https://github.com/zeyvu/FalloutCraft) | Passthrough: SkyCraft's design reused for Fallout 4 |
| [OWCraft](https://github.com/Yaekai/OWCraft) | Passthrough: SkyCraft's design reused for Outer Wilds, with a development log and design doc |
| [GTA San AnSkateas](https://github.com/ryglizzy/GTA-San-AnSkateas) | Skate 3's engine running inside GTA San Andreas |
| [2010 Rust Rewrite Mashup](https://github.com/chasmlol/2010-rust-rewrite-mashup) | A Rust rewrite combined with other games |
| [IW4L](https://github.com/vladtrc/iw4L) | Rust/Bevy runtime for Modern Warfare 2 (2009), reading your own install. Experimental, and honest about it |
| [gang-beasts-rust](https://github.com/muffinmxn/gang-beasts-rust) | Rust/Bevy rewrite with Python extractors and a whitelist `.gitignore` |
| [benilla](https://github.com/samwhosung/benilla) | A large Rust/Bevy rewrite (a WoW 1.12.1 client) |
| [universal-modder](https://github.com/rehan-remade/universal-modder) | Ten agent skills, a CLI, and a knowledge base of per-game field notes |

Finished open-source engine reimplementations, if you want to see what the long game looks like: [OpenMW](https://github.com/OpenMW/openmw), [OpenRCT2](https://github.com/OpenRCT2/OpenRCT2), [OpenTTD](https://github.com/OpenTTD/OpenTTD).

## Who these guides are for

People who have never written code and want to try something anyway. You don't need to be a programmer to start, and you don't need Rust or reverse engineering either. Do keep in mind, having knowledge and experience will get you a long way.

You do need to be willing to describe problems clearly and to spend most of your time playtesting and reporting back.

## Get help

Guides can only cover so much. For anything specific to your setup, ask in **#support-help** or post your project in the **#share-your-projects** channel on the [chasm server](https://discord.gg/ccFpNC26Ts). That's where people post problems and projects.

Include your games and exact versions, the loaders, the agent and model, what you tried, and the logs. If the chat got stuck, the `STATUS.md` trick in [guide 4](guides/04-prompting-and-workflow.md#the-handoff-trick-for-stuck-chats) writes most of that for you.

Corrections to the guides themselves are better as a pull request. See [CONTRIBUTING.md](CONTRIBUTING.md).

## Open questions

Nobody has settled these. If you know the answer, post it on the Discord:

- Does a detailed prompt or a short, loose one work better? [Both camps are quoted here.](guides/04-prompting-and-workflow.md#the-prompting-debate-as-members-put-it)
- Which free model can finish a project?
- How do you handle Unreal Engine games?

## Contributing

Experienced developers are welcome. Technical write-ups, corrections, dead ends worth documenting, and workflow examples all help.

## Disclaimer

These are unofficial fan projects. They are not affiliated with or endorsed by any game's developer or publisher. Nothing here is legal advice.

[LEGAL.md](LEGAL.md) has the full notice, what to do if a publisher contacts you, and the DMCA counter-notification process.

## Licence and attribution

Guides: [MIT](LICENSE). Linked projects keep their own licences, so check each one before reusing its code.

If you write a guide based on one of these, credit it by name and keep its licence. [FalloutCraft](https://github.com/zeyvu/FalloutCraft) and [OWCraft](https://github.com/Yaekai/OWCraft) both credit [SkyCraft](https://github.com/chasmlol/SkyCraft) that way.
