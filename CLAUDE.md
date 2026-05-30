# Conjecture — Development Notes

Conjecture is a Claude Code plugin that installs a falsification-driven project wiki.

The plugin exists because ordinary AI project wikis tend to promote plausible guesses into durable "knowledge." Conjecture makes the opposite move: every important claim starts as a prediction with a fail condition, and only earns authority after observation.

## Product shape

User-facing summary:

> A project wiki for testing your development beliefs before they harden into facts.

Core loop:

```text
predict -> observe -> notice the gap -> update
```

The loop does not end at the wiki. Confirmed beliefs change the project — code, benchmarks, a strategy doc — and each change is itself a new prediction. That outer loop is `/conjecture:exploit`.

The wiki is not a knowledge dump. It is memory for that loop:

- open predictions hold untested beliefs
- confirmed predictions hold beliefs that survived a test
- refuted predictions hold wrong beliefs and what they revealed
- knowledge pages hold evidence-backed patterns
- frameworks and axioms hold lenses that must periodically be challenged

## Structure

```text
conjecture/
├── .claude-plugin/plugin.json   # plugin manifest
├── skills/                      # Claude Code skills exposed by the plugin
│   ├── init/                    # scaffold wiki in a target project
│   ├── migrate/                 # convert non-conjecture wiki to schema
│   ├── test-hypotheses/         # run an open prediction's test, resolve at n>=3
│   ├── exploit/                 # apply confirmed knowledge to the project (outer loop)
│   ├── ingest/                  # enrich wiki from raw material
│   ├── distill/                 # compress pages without losing load-bearing facts
│   └── wiki-compile/            # audit whether the wiki is learning
├── protocol/
│   ├── CLAUDE.md                # canonical wiki schema template
│   └── scaffold/                # starter index/log for /conjecture:init
└── README.md                    # public explanation
```

## Development rules

- Keep skills project-agnostic. Read the target project's `wiki/CLAUDE.md`, `wiki/index.md`, and `wiki/log.md`; do not hardcode Iris, VimX, or this repo.
- Preserve the distinction between raw input and knowledge. Transcripts, benchmark output, and user notes are evidence sources, not truth.
- Prefer updating existing wiki pages over creating new ones. More pages are not progress unless they improve prediction quality.
- Keep refutations visible. Wrong predictions are product value, not cleanup.
- When changing the protocol, ask what decision the change will alter and what future observation would prove it wrong.
- Applying knowledge is itself a prediction. `exploit` files a bet for every application and calibrates to blast radius (act on reversible changes, propose high-stakes ones); it never rewrites the project on "wiki says so."

## Testing

Test skills inside a project that already has a Conjecture-style wiki. The original live instance is `~/Projects/iris`; VimX has the precursor wiki that exposed the failure mode this plugin is meant to fix.
