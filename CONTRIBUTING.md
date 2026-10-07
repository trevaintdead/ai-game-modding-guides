# Contributing

These guides exist because people kept asking the same questions and getting the same answers. If you know something that isn't here, adding it is the most useful thing you can do.

Questions, half-written ideas, and "is this even possible?" are all welcome on the Discord. You don't need a finished write-up to start a conversation there.

You do not need to be a professional developer. This repo is young and has no contributors yet, which makes a first one valuable.

## What's wanted

**Experienced developers writing proper technical guides.** This is the biggest gap. `toast` said it directly on the Discord: *pretty daunting, will it be technically thorough for those who would want to learn?* If that question is aimed at you, this repo is the place to answer it.

Specifically useful:

- A real walkthrough of a project you finished, including the dead ends
- Corrections to anything here that's wrong or outdated
- Answers to the open questions in [the FAQ](guides/07-faq.md#still-unanswered)
- New games, loaders, or engines to add to [guide 8](guides/08-mod-loaders-and-script-extenders.md)
- Testing and performance write-ups, which is the least covered topic

**Workflow write-ups from beginners too.** If you got something working recently and you remember being confused, you are the person who can write it down. See [`templates/workflow-writeup.md`](templates/workflow-writeup.md).

## What's not wanted

- **Anything about anti-cheat, DRM, or online play.** Not as a how-to, not as a "how I got around it." This is a hard line, not a preference.
- **Game assets, ripped or extracted, in any form.** Including in screenshots beyond fair use, and including in issues or pull requests.
- **Decompiled code.**
- **Guides that only say "tell the AI to do it."** That is partly true, and it is not a guide. Write what happened around it.
- **Vague or unverified claims.** "X works great" with no version numbers is worse than nothing, because people will follow it.

## How to contribute

1. **Open an issue first** for anything substantial. Two minutes of talking it through saves everyone a wasted pull request. Even if you'd rather just write it, an issue means people who had the same problem get an answer even if your PR stalls.
2. **Fork and edit.** Markdown only, no build step.
3. **Keep the voice.** Short sentences, plain words, no jargon without an explanation. The audience has never written code.
4. **Use real numbers.** Version numbers, timings, error messages. "It worked after about 20 minutes on my machine" beats "it was fast."
5. **Mark uncertainty honestly.** If you don't know, say so. The FAQ has a section for that.

### Style

- Third person or "you". Avoid "we".
- British or American spelling, consistent within a file.
- Sentence case for headings.
- Backticks for file names, commands, and log output.
- Tables for anything you'd otherwise compare in a list.

Match the tone of the existing guides. They're deliberately plain, and they say "members report" rather than asserting facts they can't back up. That keeps them useful when things change.

## Where things go

| File | Put it here |
|------|-------------|
| Getting started questions | [00-start-here.md](guides/00-start-here.md) |
| Agents, models, cost, limits | [01](guides/01-choose-and-set-up-an-ai-agent.md) |
| Linking two games | [02-passthrough-mods.md](guides/02-passthrough-mods.md) |
| Rebuilding an engine | [03-rust-rewrites-and-ports.md](guides/03-rust-rewrites-and-ports.md) |
| Prompts and keeping a project on track | [04-prompting-and-workflow.md](guides/04-prompting-and-workflow.md) |
| Debugging and playtesting | [05](guides/05-testing-and-troubleshooting.md) |
| Legal, ethics, licensing | [06](guides/06-rules-legal-and-publishing.md) |
| Quick Q&A | [07-faq.md](guides/07-faq.md) |
| Loaders, script extenders, engine families | [08](guides/08-mod-loaders-and-script-extenders.md) |
| Full walkthroughs | [09](guides/09-worked-example-passthrough-mod.md) |
| Getting a project seen | [10-posting-your-project.md](guides/10-posting-your-project.md) |
| Which kind of project an idea is | [14-choosing-a-route.md](guides/14-choosing-a-route.md) |
| Real project case studies, with versions and limits | [15](guides/15-case-studies-what-each-project-actually-did.md) |
| Symptom → cause notes for sync, rendering and collision | [16](guides/16-ownership-sync-and-rendering.md) |
| Project files people copy | `templates/` |

## Open debates

[Guide 4](guides/04-prompting-and-workflow.md#the-prompting-debate-as-members-put-it) has a long section on whether detailed prompts or short loose prompts work better. Members disagree sharply. If you run a controlled comparison, that would be a useful contribution and it would be welcome. Post the results on the Discord or open a pull request.

[Guide 8](guides/08-mod-loaders-and-script-extenders.md) is missing plenty. If you know that a game has a good modding setup that isn't listed, add it. Include the loader, its language, and a link.

Two more known gaps:

- **Non-Windows.** The main example projects are Windows-only. A few creators report Linux (Wine/Proton) and macOS (CrossOver) setups, listed in [guide 8](guides/08-mod-loaders-and-script-extenders.md#windows-is-the-common-denominator). A step-by-step write-up of one would fill a real hole.
- **Games with publishing restrictions.** Halo MCC and the Xbox decomp projects have terms that limit what a port can use, and nobody has written that up.

## Licence

By contributing you agree your work is published under the repo's [MIT licence](LICENSE).

## Code of conduct

Be useful and be decent. No gatekeeping, no "just tell the AI to do it" as a dismissal, and no sneering at beginners. A lot of people here are beginners, and being the one who helps is the whole point.
