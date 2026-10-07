# 11. Models and What to Spend

What to pay, and what to point it at. Everything here was checked against the providers' own pricing pages in October 2026. Plans change, so verify before you commit.

## The short version

| Budget | What to get | What you get |
|--------|--------------|--------------|
| Free | [OpenCode](https://opencode.ai) with a free model | Enough to try things and follow the guides |
| About $10 | [OpenCode Go](https://opencode.ai/go) | The best value. DeepSeek V4.1 Flash and friends at high volume |
| $20 | [Claude Pro](https://claude.com/pricing) | The best single subscription, and the pick if you're buying one plan |
| $40 | OpenCode Go Plus | More of the same models. Rarely the best call over Claude Pro |
| $100 | Claude Max 5x | For when $20 runs out mid-project |
| $200 | Claude Max 20x | Only once you've proven you need it |
| Pay per token | Any provider's API, usually via OpenRouter | When usage caps are the problem rather than budget |

Reach for a different model and the price moves a lot. DeepSeek V4.1 Flash is $0.15 and $0.60 per million tokens off-peak through OpenCode Go, and a direct API comes out close to that. Claude's current flagship, Opus 5.5, is $4 and $20. Per token means you never hit a wall, and you also never get a flat monthly rate, which suits people who work in bursts.

## About $10: OpenCode Go

Go is a $10 subscription giving access to the strongest open coding models. DeepSeek V4.1 Flash is the one to start with: cheap, fast, and good enough for most of this work.

DeepSeek V4.1 Flash through Go costs $0.15 per million input tokens and $0.60 output off-peak, doubling to $0.30 and $1.20 at peak. Peak is 01:00 to 04:00 and 06:00 to 10:00 UTC on weekdays. Run your builds overnight and you pay the off-peak rate.

Its $60 monthly allowance works out to roughly 26,000 requests per five-hour window, or 130,000 a month. That is a lot of work.

Go's limits are the same shape as Claude's: 20% of the monthly allowance per five hours, 50% weekly, 100% monthly.

Go Plus is $40 with higher limits. Only worth it if you're running many agents in parallel or you keep hitting Go's cap and don't want to switch to Claude.

## $20: Claude Pro

The default recommendation if you're buying one subscription. Most people doing this work are on it.

$20 monthly, or $17 monthly on annual billing ($200 up front). Claude Code is included.

Limits are a rolling five-hour session window plus a weekly cap that resets at a fixed time assigned to your account. Expect roughly 3 to 4 hours of constant use per five-hour window on a top model. A from-scratch basketball game came in comfortably on the $20 tier.

Treat the session window as the real constraint. The weekly limit rarely bites if you hand chats over with a [`STATUS.md`](../templates/STATUS-handoff.md) file instead of letting one grow for a week.

## If you want to upgrade, go in order

**$20, max it out. Then $100, max it out. Then $200.**

The reasoning is that each tier is worth it only if you exhausted the one below.

| Tier | Price | Pro equivalent |
|------|-------|----------------|
| Pro | $20 | 1x |
| Max 5x | $100 | 5x |
| Max 20x | $200 | 20x |

Two things people get wrong here:

- **The 5x and 20x multiples apply to the five-hour session window**, not to your weekly allowance. The $200 plan's weekly allowance is roughly double the $100 plan's, not twenty times.
- **A weekly limit sits on top of the session window.** Upgrading multiplies your session capacity; it doesn't remove the weekly cap.

Max is monthly only. Upgrading mid-cycle charges prorated.

If you're paying per token instead, a route that works: OpenRouter with a strong open model. VS Code plus Roo Code with OpenRouter and a DeepSeek model is a common combination when a Claude plan is out of reach.

### Two things that change the bill more than the model

Both matter more here than in most projects, because this work is context-heavy. You are feeding the agent decompiled output, whole repositories, and design docs.

**Context window is 1M tokens on both main options, at standard price.** DeepSeek V4.1 Flash has a 1M context. Opus 5.5 includes the full 1M window at the normal per-token rate, so a 900k request costs the same rate as a 9k one. There is no long-context surcharge to worry about. DeepSeek caps output at 384K.

**Prompt caching cuts the cost of repeating context by 90%.** A cache read bills at 0.1x the base input price. Writing to the 5-minute cache costs 1.25x base input, so caching pays for itself after one read. The 1-hour cache costs 2x, so it needs two reads to break even. Both DeepSeek and Claude support it.

That matters because the expensive thing in a long modding session is the same repository and the same decompiled files going back in on every turn. Cache the stable prefix and stop paying full price for it.

Ask the agent to enable it rather than assuming it is on:

```
This project has a large stable context: docs/DESIGN.md, the game format
spec, and the files we've already analysed. Enable prompt caching on that
prefix so we stop paying full input price for it every turn. Show me the
cache hit rate in the usage output.
```

**Batch API is half price** on both input and output, for work that can happen in the background. Re-compiling a batch of assets or running the same analysis over a hundred data files is the case for it. It is no use for interactive work, since the results come back later.

One caveat on Claude specifically: Opus 5.5 uses a newer tokenizer that produces roughly 30% more tokens for the same text than earlier models did. Comparing two models on cost without checking which tokenizer they use is comparing different units.

## OpenAI and ChatGPT models

Not recommended right now, for two reasons: they perform worse on this kind of work than Claude at comparable tiers, and they give less usable usage per subscription.

**If you already have a ChatGPT subscription, keep it.** The models are very good and the allowance is decent. There's no reason to cancel. Don't switch for this recommendation.

This is a community read, not a benchmark result, and it will go stale. If you disagree and get better results on a real project, that counts, and the page should change.

## Free options that aren't nothing

Before spending anything:

- **Free models inside OpenCode.** Several are free for a limited time, including DeepSeek-adjacent options and some stealth models. Free models get cut off and rate-limited, so treat them as good for learning the workflow and bad for a serious project.
- **Local models on your own hardware.** They don't work well for this. A 12 GB GPU isn't enough for a good local coding model. Your GPU being a 5090 makes no difference to a cloud model, since that work runs on the provider's servers.
- **Pay-per-token APIs.** Cheap enough to try something, and you stop when you stop. Good for a weekend project, bad for anything long.
- **Cheaper Claude tiers.** Running Sonnet instead of a top model costs less per token and is usually good enough for the bulk of a project. Save the expensive model for the part where you're stuck.

## What a plan gets you that tokens don't

Subscription plans aren't only about price. Three things are easier on a plan and awkward per token:

- **No usage ceiling.** Per-token billing means a runaway loop costs real money instead of hitting a wall.
- **Priority access.** Max plans get served ahead of free users at busy times, which matters when you're mid-project.
- **Credits included.** Paid plans come with usage credits for image and asset generation, which you otherwise buy separately.

The reason people stay on a subscription is the ceiling, not the price. Predictable, short work is cheaper per token.

## Open questions

- Which free model is genuinely best inside OpenCode. Nobody has run the comparison.
- Whether free plans can complete a real project. Most people hit limits fast.
- Whether local models can handle a real project on 12 GB of VRAM.
- Whether any of this holds on Windows vs Linux. Every project in these guides is Windows-only anyway.

If you find out, post it on the Discord or open a pull request.

## Where to check current prices

- [Claude pricing](https://claude.com/pricing) and [Max plan details](https://support.claude.com/en/articles/11049741-what-is-the-max-plan)
- [OpenCode Go](https://opencode.ai/go) and [Zen pricing](https://opencode.ai/docs/zen/)
- [DeepSeek API pricing](https://api-docs.deepseek.com/quick_start/pricing)
- [Claude API pricing](https://platform.claude.com/docs/en/about-claude/pricing), which is the fuller page and has the per-model table the marketing page hides

Subscribe to a monthly plan rather than annual until you know how much you use. The annual discount is 15% on Claude Pro, which isn't worth paying for a plan you might outgrow in a month.

---

<sub>[Spot a mistake? [Edit this page on GitHub](https://github.com/trevaintdead/ai-game-modding-guides/edit/main/guides/11-models-and-cost.md).](https://github.com/trevaintdead/ai-game-modding-guides/edit/main/guides/11-models-and-cost.md) &middot; [Open an issue](https://github.com/trevaintdead/ai-game-modding-guides/issues/new) &middot; Part of [AI Game Modding Guides](https://github.com/trevaintdead/ai-game-modding-guides)</sub>
