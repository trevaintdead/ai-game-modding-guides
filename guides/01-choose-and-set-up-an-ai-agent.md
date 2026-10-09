# 1. Choosing and Setting Up an AI Agent

## Agent vs. chat website

Most people get stuck on this one.

- **A chat website** (claude.ai in the browser, ChatGPT in the browser) only sees what you paste or upload. It can't open your game folders, so you hit "file too large" errors and end up copy-pasting code by hand.
- **An agent** runs on your PC. It reads your game folders, creates and edits files, runs builds, and reads logs. Nothing to upload.

You want an agent. If you're using the Claude desktop app, look for the Code mode. It is in the top left and easy to miss. Check the provider's current docs for exact steps.

## Which agent

Members mostly use these. Pick one and stick with it while you learn.

| Tool | Notes from the community |
|------|--------------------------|
| **Claude Code** | The most mentioned. Use the desktop app; the terminal and VS Code versions work fine but are worse to live in |
| **Codex** | Also widely used, including to run Ghidra-based decompiling |
| **OpenCode** | Works with many models, including free ones. Which free model is best is still an open question |
| **VS Code + Roo Code + OpenRouter** | A pay-per-use route, commonly paired with a DeepSeek model. Only worth it if a subscription is not available to you, since it costs more for the same work |

### A note on "MCP"

Several people confused these. **Claude Code and Codex are agents.** MCP (Model Context Protocol) is a standard for plugging extra tools into an agent.

You do **not** need any MCP server to start. None of the example projects list one as a requirement, and no MCP server is needed to read your game files, write code, or run a build. That is all the agent does on its own.

The general rule is that a command line beats an MCP server. If a tool has a usable CLI, the agent can drive it directly and you have one fewer moving part. Reach for MCP when there is no CLI worth using, which is mostly the case with GUI-only desktop tools.

If you go on to reverse engineer anything, that's when one is worth having, because a decompiler is a GUI tool with no command line worth scripting. Ghidra and IDA both ship MCP servers, which means the agent can decompile and rename functions itself instead of you pasting disassembly into a chat. The official [Hex-Rays IDA MCP](https://github.com/HexRaysSA/ida-mcp) installs with one command, and [ghidra-mcp](https://github.com/bethington/ghidra-mcp) does the same for the free option.

### What the agent is

Each of these has a desktop app and a terminal version, and you want the app. The terminal versions are what the developers test, which makes them the most current, but they are also unpleasant: a chat window you cannot resize, no inline images, and status you cannot see without typing a slash command. The apps cost nothing extra and are easier to live in for hours.

The agent runs on your machine with your user account's file access, so what protects you depends on how you've configured it.

Claude Code asks before it acts, shows file edits as diffs for you to approve, and has a built-in sandbox you switch on with `/sandbox`. Codex has its own permission and sandbox settings. Protection drops when people switch to full-access modes, which members here describe doing. That is why the safety section at the bottom of this guide matters: the defaults help, and the failure mode is turning them off.

The sandbox is worth understanding before you rely on it, because it has real limits. It covers shell commands only, so Claude's own file tools, hooks and local MCP servers still run with your full access. It runs on macOS, Linux and WSL2, so on native Windows you need Claude Code inside WSL2 to get it. It is off by default. And if it cannot start, because a dependency is missing or the platform is unsupported, Claude Code warns you and carries on running commands unsandboxed rather than stopping. Setting `failIfUnavailable` makes it exit instead, which is the stricter choice if you want the sandbox to be a real gate.

One more thing worth knowing: when a command fails under the sandbox, Claude may retry it with `dangerouslyDisableSandbox`, and that retry runs outside the boundary. In the default modes you get a prompt unless a matching allow rule covers it, so decline it when you did not expect it. In `bypassPermissions` mode the retry runs with no prompt at all, which is one more reason not to run that way. `/sandbox` has an Overrides tab that turns the retry off, called strict sandbox mode.

Sandboxed commands can still read most of the machine by default, including credential files such as `~/.ssh` and `~/.aws/credentials`, unless you deny those paths in the sandbox settings.

None of that makes it safe. It narrows what a shell command can reach, which is worth having, and the rest still depends on the habits at the bottom of this guide.

None of the agents will touch DRM or anti-cheat on their own, and online-only games are a no-go for all of them. If an agent refuses something, read the reason before assuming it's a limitation on what it can do.

You do not need an IDE, though VS Code is worth installing so you get proper file views and diffs. The agent creates your files and runs your builds either way, so you never copy-paste code into folders by hand.

## Which model

Models change quickly, so check what's current before choosing.

- Most people doing this pair Claude Code with a mid-tier Claude model. That is the best value on this kind of work. Several use the top tier, which is worth it for long unattended runs rather than everyday iteration.
- Others use OpenAI's models through Codex.
- Cheaper models work for smaller tasks, but they need more babysitting (see the handoff trick in [guide 4](04-prompting-and-workflow.md#when-a-chat-really-is-stuck)).
- **Local models** (running on your own GPU) don't work well for this. A 12 GB GPU is not enough for a good local coding model, which answers that question for most people. If you've made it work regardless, please add a write-up.
- Your hardware (a 5090, for example) doesn't matter for cloud models. The AI runs on the provider's servers.

## Cost and usage limits

Plans change often, so check the provider's current page. **[Guide 11](11-models-and-cost.md) has specific recommendations**, including which model to use at each budget. The short version: OpenCode Go at about $10 is the best value, Claude Pro at $20 is the best single subscription, and upgrading goes $20 then $100 then $200.

What to expect:

- Paid plans have a short reset window (about 5 hours) and a weekly limit.
- Roughly 3-4 hours of constant use out of each 5-hour window on a top Claude model is a typical budget. A mid-tier model stretches further.
- The $20 tier has been enough to build a from-scratch basketball game.
- **Free tiers:** not confirmed for a real project, and people expect to hit limits fast. When you outgrow free, buy a subscription rather than paying per token.
- Long sessions stay affordable because of prompt caching, which bills repeated context at a fraction of the normal rate. That only works if you stay in one chat and keep the same model, so don't start a new chat to tidy up. See [guide 4](04-prompting-and-workflow.md#stick-to-one-chat).

## Setting up safely

Agents can read and delete files, and many members run them with broad access. That's convenient and risky at the same time. A few habits cut the risk:

1. **Make one folder for the project** and run the agent inside it.
2. **Use Git from day one.** Commit after each working step so you can undo mistakes.
3. **Back up your game saves** before testing.
4. **Be careful with "full access" modes.** Running an agent with full PC access is more convenient, but a mistake can hit files you care about. A separate Windows user account, a virtual machine, or a container such as Podman limits the damage.
5. **Keep passwords and API keys out of files the agent can read**, and out of your repo.
6. **If the agent keeps failing to get access** (a common Codex complaint), read that tool's docs on permission and sandbox settings and grant it access to your project folder and game folders only.
7. **Write down the rules instead of trusting your memory.** A rules file in your project means the agent follows them in every session, including the ones you forget. See [`templates/AGENTS-starter.md`](../templates/AGENTS-starter.md).

## Helpful extras (optional)

- **VS Code** (or another editor) with your agent's extension, so you get proper versioning and file views. It isn't required.
- **Git and a GitHub account.** You'll need these to share your project.
- **[universal-modder](https://github.com/rehan-remade/universal-modder):** an open-source toolkit that wraps an agent you already have. Clone it and start your agent inside it. It works with Claude Code, Codex, Cursor, Gemini CLI, Copilot and OpenCode. Its art tools need a separate fal API key, and it needs Python 3.10+ and ffmpeg. It limits itself to single-player or offline games you own and won't touch anti-cheat.

Worth being clear about what the "skills" in projects like this are, since the marketing leans on them. A skill is a prompt file that gets attached to your conversation every turn. It is not a plugin, it does not add capability the model lacks, and installing eleven of them does not make your agent better at anything. A skill that lists the tools and paths for your particular game saves you a search. That is the whole value, and it is real, but it is a convenience. The same knowledge typed into the chat once does the same job.

You do not need any of this to start. Tell your agent what you want and let it work.

---

<sub>[Spot a mistake? [Edit this page on GitHub](https://github.com/trevaintdead/ai-game-modding-guides/edit/main/guides/01-choose-and-set-up-an-ai-agent.md).](https://github.com/trevaintdead/ai-game-modding-guides/edit/main/guides/01-choose-and-set-up-an-ai-agent.md) &middot; [Open an issue](https://github.com/trevaintdead/ai-game-modding-guides/issues/new) &middot; Part of [AI Game Modding Guides](https://github.com/trevaintdead/ai-game-modding-guides)</sub>
