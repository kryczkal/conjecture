# Conjecture

A Claude Code plugin that stops your project wiki from becoming fiction.

## The problem

I kept making development decisions based on what I thought I knew. Then I'd hit a wall and realize the thing I "knew" was a guess I'd never tested. I wasn't building with knowledge — I was building with vibes that had been sitting in a wiki long enough to feel like facts.

I ran Karpathy-style LLM wikis across three projects. They all degraded the same way: feed in transcripts, docs, session notes, and the model synthesizes "knowledge." Six weeks later you have 40 pages of confident-sounding claims and no idea which ones are true. Plausible first impressions stored with the same authority as verified findings. The file count grew. The signal didn't.

The wiki wasn't giving me direction. It was giving me false confidence. I'd have been better off without it — at least then I'd know I was guessing.

## What went wrong (concrete example)

In `vimx` (a browser automation tool), the wiki confidently stated that affordance-typed tools — `press`, `type`, `toggle`, `select` — were the core differentiator. It sounded right. The architecture was built around it. Multiple sessions reinforced it.

Then I actually tested it. Benchmarks across 50+ sites showed the type system adds zero information over raw HTML tags. `<input type="checkbox">` already tells you it's a toggle. The classification was cosmetic.

The real moat was something I'd almost overlooked: scanner quality. 7x fewer empty labels. 2.25x fewer duplicate labels. The boring part was the valuable part.

That "knowledge" had been sitting in the wiki for weeks, shaping priorities, because nobody asked: **is this actually true, or does it just sound true?**

## The fix

Conjecture treats every claim the way science treats hypotheses: guilty until proven innocent.

New insight enters as a **prediction** — a bet with an explicit fail condition. It earns the right to become knowledge only after it survives contact with reality. If it fails, it goes in `refuted/` and stays there, because a documented wrong belief is more useful than an undocumented right guess.

A governor tracks whether you're actually learning (prediction accuracy improving) or just accumulating (more pages, same error rate).

**predict -> observe -> notice the gap -> update**

That gap between what you expected and what happened is the only thing that makes the next session smarter.

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
