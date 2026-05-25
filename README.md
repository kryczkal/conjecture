# Conjecture

A Claude Code plugin that stops your project wiki from becoming fiction.

## The problem

I ran Karpathy-style LLM wikis across three projects. They all degraded the same way: plausible first impressions got stored with the same authority as verified findings. Six weeks later — 40 pages of confident claims, no idea which ones are true. I wasn't building with knowledge. I was building with vibes that had been sitting in markdown long enough to feel like facts.

In `vimx`, the wiki confidently stated that affordance-typed tools — `press`, `type`, `toggle`, `select` — were the core differentiator. Multiple sessions reinforced it. Benchmarks across 50+ sites showed the type system adds zero information over raw HTML tags. The real moat was scanner quality: 7x fewer empty labels, 2.25x fewer duplicates. The boring part nobody tested was the valuable part.

## The fix

New insight enters as a **prediction** with an explicit fail condition. It becomes knowledge only after surviving contact with reality. Refuted predictions stay in `refuted/` — a documented wrong belief is more useful than an undocumented right guess. A governor tracks whether prediction accuracy is improving or pages are just accumulating.

**predict -> observe -> notice the gap -> update**

## Commands

| Command | What it does |
|---|---|
| `/conjecture:init` | Create the `wiki/` structure in the current project |
| `/conjecture:predict` | State a conjecture before acting, with a fail condition |
| `/conjecture:ingest <file>` | Read raw material as evidence — surface tensions, don't promote opinions |
| `/conjecture:distill <page>` | Compress a page while preserving load-bearing content |
| `/conjecture:wiki-compile` | Audit: is the system learning or merely accumulating? |

## Install

```
/plugin marketplace add kryczkal/conjecture
/plugin install conjecture@kryczkal
```

Or: `git clone https://github.com/kryczkal/conjecture.git && ln -s "$(pwd)/conjecture" ~/.claude/plugins/data/conjecture`

Full protocol: [`protocol/CLAUDE.md`](protocol/CLAUDE.md)

## License

MIT
