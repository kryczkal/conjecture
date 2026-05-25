# Conjecture

A Claude Code plugin for running your project like an ongoing experiment.

Not a notes wiki. Not "let the LLM dump insights into markdown."

**If a claim matters, it does not get to become knowledge just because it sounds right.** It starts life as a conjecture. Then you test it. Then it either survives, dies, or gets revised.

## Why

In `vimx`, one strong story was that affordance-typed tools (`press`, `type`, `toggle`, `select`) were the killer feature. It sounded right. It felt true.

Benchmarks showed the important part was not the affordance distinction at all — for normal HTML, the DOM already encodes that. The real value was scanner quality: better labels, better disambiguation, less noise.

Without Conjecture, "affordance typing is the core moat" would have stayed filed as knowledge. With Conjecture, it entered as a bet, got tested, got partially refuted, and the system learned something sharper.

That is the difference between learning and folklore.

## The loop

**predict -> observe -> notice the gap -> update**

Skip the prediction and you cannot tell whether you learned or just rationalized afterward. Skip the record of failure and you lose the thing that would have made the next session smarter.

Wrong predictions are first-class. They are where the model of the project actually improves.

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
