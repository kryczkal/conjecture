# Conjecture

Conjecture is a Claude Code plugin for keeping project knowledge honest.

It turns "I think this is true" into:

```text
prediction -> test -> result -> update
```

Instead of letting guesses harden into wiki facts, Conjecture makes important claims start as predictions with fail conditions.

## Why

LLMs are good at writing confident explanations. Wikis are good at making old explanations look authoritative. That combination quietly creates fake knowledge:

> a guess gets written down, future sessions read it as fact, and the project starts steering around something nobody tested.

Conjecture is for work that spans many Claude sessions, where project direction depends on assumptions that might be wrong.

It helps you keep untested ideas, stale docs, and refuted product stories from steering the project as if they were facts. The goal is not more documentation. The goal is better steering.

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
| `/conjecture:test-hypotheses` | Run an open prediction's test and book the result — confirmed or refuted (n>=3 gated) |
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
