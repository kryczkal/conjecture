# Conjecture

Conjecture is a Claude Code plugin for keeping project knowledge honest.

It turns "I think this is true" into:

```text
prediction -> test -> result -> update
```

Instead of letting guesses harden into wiki facts, Conjecture makes important claims start as predictions with fail conditions.

## Why use it

LLMs are good at writing confident explanations. Wikis are good at making old explanations look authoritative. That combination quietly creates fake knowledge:

> a guess gets written down, future sessions read it as fact, and the project starts steering around something nobody tested.

This is what Conjecture prevents.

Use it when:

- you are building with Claude across many sessions
- project direction depends on assumptions you have not verified
- your wiki/README/CLAUDE.md keeps going stale
- agents keep repeating old claims without checking code or benchmarks
- you want wrong beliefs preserved instead of forgotten

## What it helps with

Conjecture helps you catch:

- **fake principles**: ideas written down too early and treated as rules
- **stale docs**: old architecture pages trusted over current code
- **wrong product stories**: "this is the moat" claims that benchmarks later refute
- **misleading validation**: measurements that prove safety but not direction
- **wiki bloat**: pages accumulating without improving decisions

The goal is not more documentation. The goal is better steering.

## The basic move

Bad wiki entry:

```md
Typed browser actions are our core differentiator.
```

Conjecture entry:

```md
Prediction: Typed browser actions will reduce wrong-action errors.
Fail condition: raw HTML lets the agent infer the same actions with no mismatches.
Result: refuted. Across 1,500+ elements, typing added no information; scanner label quality mattered.
```

The first entry sounds like knowledge. The second entry can be tested, lost, and used to change direction.

## Commands

| Command | What it does |
|---|---|
| `/conjecture:init` | Create a `wiki/` built around predictions |
| `/conjecture:predict` | State a falsifiable bet before work |
| `/conjecture:ingest <file>` | Pull testable claims from notes, logs, transcripts, benchmarks |
| `/conjecture:distill <page>` | Shorten a page without losing decisions or evidence |
| `/conjecture:wiki-compile` | Check whether the wiki is learning or just growing |

## Install

```sh
/plugin marketplace add kryczkal/conjecture
/plugin install conjecture@kryczkal
```

Or clone it directly:

```sh
git clone https://github.com/kryczkal/conjecture.git
ln -s "$(pwd)/conjecture" ~/.claude/plugins/data/conjecture
```

Full protocol: [`protocol/CLAUDE.md`](protocol/CLAUDE.md)

## License

MIT
