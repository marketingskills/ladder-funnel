# LadderFunnel Architect

An AI skill for building **paid acquisition funnels** that replace SaaS free trials with a $7 intro product → order bump → recurring upsell structure.

This is the open-source skill definition used by the `ladderfunnel-architect` agent. It contains the full methodology: offer strategy, copywriting formulas, ad frameworks, email ascension sequences, and the Funnel Blocks spec.

## What's Inside

| File | Purpose |
|------|---------|
| `SKILL.md` | Main agent instructions — strategy, copy, ads, email, and build guidance |
| `references/concept.md` | The core methodology: why free trials fail and how paid acquisition fixes it |
| `references/offer/framework.md` | Offer stack generation ($7 intro + bump + upsell, or $27 alternative) |
| `references/copy/pages.md` | Copywriting formulas for all 4 funnel pages |
| `references/ads/strategy.md` | Facebook ad campaign structure, creative frameworks, visual styles |
| `references/email/ascension.md` | 30-day email sequence to convert intro buyers into subscribers |
| `references/build/blocks.md` | The Funnel Blocks spec — portable JSON block config for self-hosting |
| `references/build/api.md` | LadderFunnel Pro API reference for hosted platform users |

## How to Use

This skill is designed for AI agent platforms (Claude, Codex, Gemini, etc.). To use it:

1. **Install the skill** — copy the files to your agent's skill directory
2. **Ask for help** — describe your SaaS and the skill will guide you through the 5 phases: Offer Stack → Funnel Copy → Ads Strategy → Email Ascension → Tracking & Build
3. **Choose a build path** — the skill presents 3 options:
   - **Funnel Blocks** — lightweight hosted blocks platform (portable JSON config, no lock-in)
   - **LadderFunnel Pro** — full hosted platform with courses, AI, and Meta campaigns
   - **Self-host** — render the same blocks inside your own app

## License

MIT — use freely, modify, redistribute. This is a community skill for the marketing skills ecosystem.
