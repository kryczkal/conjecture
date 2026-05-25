# Conjecture

Conjectures and refutations for software.

A Claude Code plugin that turns every change into a falsifiable prediction. You predict what will happen, do the work, record what actually happened, and the gap between prediction and reality is where you learn.

## Install

```
/plugin marketplace add kryczkal/conjecture
/plugin install conjecture@kryczkal
```

Or manually:
```bash
git clone https://github.com/kryczkal/conjecture.git
cd conjecture && ln -s "$(pwd)" ~/.claude/plugins/data/conjecture
```

## Use

In any project:

```
/conjecture:init              # scaffold a wiki in this project
/conjecture:predict           # state what you expect before acting
/conjecture:ingest <file>     # enrich the wiki with external knowledge
/conjecture:distill <page>    # tighten a wiki page to minimum viable content
/conjecture:wiki-compile      # lint the wiki, check learning health
/conjecture:evolve <skill>    # genetically improve any skill
/conjecture:brainstorm        # turn ideas into committed spec files
```

## What it does

Every project gets a `wiki/` directory — a self-learning knowledge base where:

- **Predictions** are the core unit. A bet specific enough to be wrong.
- **Knowledge** is observations with evidence, tagged by maturity (T0-T3).
- **Frameworks** are interpretive lenses that generate predictions. A framework that hasn't generated a prediction in 30 days is dead weight.
- **Axioms** are constitutional rules. Can't be tested — but must be challenged.
- **The governor** tracks whether you're actually learning (prediction accuracy trend) or just accumulating pages.

The learning atom: predict → observe → notice the gap → update. The wiki is the loop's memory.

## The protocol

The full protocol lives in [`protocol/CLAUDE.md`](protocol/CLAUDE.md). Key ideas:

1. Every change is a conjecture first
2. Conjectures must have fail conditions
3. Refutations are first-class — never delete a wrong prediction
4. Patterns are actionable without mechanisms (T0 is first-class)
5. The system declares its own epistemology and its blind spots
6. Less is more — every page earns its keep or gets deleted
7. The system examines itself — axioms get challenged, the governor tracks learning rate

## Skills

| Skill | What it does |
|-------|-------------|
| `init` | Scaffold `wiki/` with the Conjecture protocol |
| `predict` | State a prediction before acting, scorecard after |
| `ingest` | Three-layer enrichment (implicit assumptions → tensions → explicit claims) |
| `distill` | Compress a wiki page to minimum viable content with coverage gating |
| `wiki-compile` | Governor + evidence audit + structural lint + action plan |
| `evolve` | Genetically improve any skill via parallel agent testing |
| `brainstorm` | Turn ideas into committed spec files via decision mapping and Q&A |

## Philosophy

From Karl Popper's *Conjectures and Refutations*: we don't learn by being right. We learn by being wrong in specific, trackable ways. The rate of becoming less wrong is the only metric that matters.

This plugin operationalizes that insight for software development. Every hypothesis about your code, your architecture, your process — make it a prediction, test it, record the result. Over time, patterns emerge: where your mental model is accurate, where it drifts, and where to invest attention.

## License

MIT
