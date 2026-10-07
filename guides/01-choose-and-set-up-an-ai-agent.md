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
| **Claude Code** | The most mentioned. Works from the desktop app, a terminal, or a VS Code extension |
| **Codex** | Also widely used, including to run Ghidra-based decompiling |
| **OpenCode** | Works with many models, including free ones. Which free model is best is still an open question |
| **VS Code + Roo Code + OpenRouter** | A pay-per-use route, commonly paired with a DeepSeek model when a Claude plan is out of reach |

### A note on "MCP"

Several people confused these. **Claude Code and Codex are agents.** MCP (Model Context Protocol) is a standard for plugging extra tools into an agent.

You do **not** need any MCP server to start. None of the example projects list one as a requirement.

If you go on to reverse engineer anything, that's when one becomes worth having. Ghidra and IDA both ship MCP servers, which means the agent can decompile and rename functions itself instead of you pasting disassembly into a chat. The official [Hex-Rays IDA MCP](https://github.com/HexRaysSA/ida-mcp) installs with one command, and [ghidra-mcp](https://github.com/bethington/ghidra-mcp) does the same for the free option.

### What the agent is

Almost all of these are a terminal program or a VS Code extension. The agent runs on your machine with your user account's file access, so what protects you depends on how you've configured it.

Claude Code asks before it acts, shows file edits as diffs for you to approve, and has a built-in sandbox you switch on with `/sandbox`. Codex has its own permission and sandbox settings. Protection drops when people switch to full-access modes, which members here describe doing. That is why the safety section at the bottom of this guide matters: the defaults help, and the failure mode is turning them off.

The sandbox is worth understanding before you rely on it, because it has real limits. It covers shell commands only, so Claude's own file tools, hooks and local MCP servers still run with your full access. It runs on macOS, Linux and WSL2, so on native Windows you need Claude Code inside WSL2 to get it. It is off by default. And if it cannot start, because a dependency is missing or the platform is unsupported, Claude Code warns you and carries on running commands unsandboxed rather than stopping. Setting `failIfUnavailable` makes it exit instead, which is the stricter choice if you want the sandbox to be a real gate.

One more thing worth knowing: when a command fails under the sandbox, Claude may retry it with `dangerouslyDisableSandbox`, and that retry runs outside the boundary. In the default modes you get a prompt unless a matching allow rule covers it, so decline it when you did not expect it. In `bypassPermissions` mode the retry runs with no prompt at all, which is one more reason not to run that way. `/sandbox` has an Overrides tab that turns the retry off, called strict sandbox mode.

Sandboxed commands can still read most of the machine by default, including credential files such as `~/.ssh` and `~/.aws/credentials`, unless you deny those paths in the sandbox settings.

None of that makes it safe. It narrows what a shell command can reach, which is worth having, and the rest still depends on the habits at the bottom of this guide.

None of the agents will touch DRM or anti-cheat on their own, and online-only games are a no-go for all of them. If an agent refuses something, read the reason before assuming it's a limitation on what it can do.

You do not need an IDE, though VS Code is worth installing so you get proper file views and diffs. The agent creates your files and runs your builds either way, so you never copy-paste code into folders by hand.

## Which model

Models change quickly, so check what's current before choosing.

- Most people doing this pair a top-tier Claude model with Claude Code. Several mention Claude Opus 5.5.
- Others use OpenAI's models through Codex.
- Cheaper models work for smaller tasks, but they need more babysitting (see the handoff trick in [guide 4](04-prompting-and-workflow.md)).
- **Local models** (running on your own GPU) don't work well for this. A 12 GB GPU is not enough for a good local coding model, which answers that question for most people. If you've made it work regardless, please add a write-up.
- Your hardware (a 5090, for example) doesn't matter for cloud models. The AI runs on the provider's servers.

## Cost and usage limits

Plans change often, so check the provider's current page. **[Guide 11](11-models-and-cost.md) has specific recommendations**, including which model to use at each budget. The short version: OpenCode Go at about $10 is the best value, Claude Pro at $20 is the best single subscription, and upgrading goes $20 then $100 then $200.

What to expect:

- Paid plans have a short reset window (about 5 hours) and a weekly limit.
- Roughly 3-4 hours of constant use on the top Claude model out of each 5-hour window is a typical budget.
- The $20 tier has been enough to build a from-scratch basketball game.
- **Free tiers:** nobody has confirmed whether you can do a real project on one. Expect to hit limits fast. A pay-per-use API key (OpenRouter and similar) is the other option.
- Long sessions use more of your limit, because the whole conversation is carried along. Start a fresh chat now and then with a short handoff note. See [guide 4](04-prompting-and-workflow.md).

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
- **[universal-modder](https://github.com/rehan-remade/universal-modder):** an open-source set of eleven agent skills plus a CLI, covering game recon, reverse engineering, asset generation, in-game testing, publishing, and a shared knowledge base of field notes. Install it with `npx skills add https://github.com/rehan-remade/universal-modder`, or clone the repo and start your agent inside it. Works with Claude Code, Codex, Cursor, Gemini CLI, Copilot and OpenCode. Its art tools need a separate fal API key, and it needs Python 3.10+ and ffmpeg. It limits itself to single-player or offline games you own and won't touch anti-cheat.

---

<sub>[Spot a mistake? [Edit this page on GitHub](https://github.com/trevaintdead/ai-game-modding-guides/edit/main/guides/01-choose-and-set-up-an-ai-agent.md).](https://github.com/trevaintdead/ai-game-modding-guides/edit/main/guides/01-choose-and-set-up-an-ai-agent.md) &middot; [Open an issue](https://github.com/trevaintdead/ai-game-modding-guides/issues/new) &middot; Part of [AI Game Modding Guides](https://github.com/trevaintdead/ai-game-modding-guides)</sub>
