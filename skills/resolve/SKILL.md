---
name: resolve
description: >
  Close the loop. Take open predictions, actively RUN their tests, and resolve them
  to confirmed/refuted with a scorecard — gated at n>=3 so completion bias can't close
  a bet early. This is the half of the loop no other skill owns: observe a prediction
  against reality. Use after running an experiment, on a schedule, or when open
  predictions pile up. Invoke with /conjecture:resolve [slug | --all | --stale].
argument-hint: [prediction slug | --all | --stale | --force]
effort: max
---

# Resolve — close open predictions against reality

`predict` opens a bet. `resolve` closes it. Every other skill creates or audits pages; this one does the act the product is named for — run the test, see what happened, and book the result honestly.

Resolution is an **action, not a report**. If a prediction's test can be run, run it. Do not defer to the user what you can execute yourself.

## 0. Read state

Read `wiki/CLAUDE.md` (scorecard format, maturity ladder, the governor, Rule 8 on context), `index.md`, and the last 5 `log.md` entries.

Determine the target set:
- a **slug** → resolve that one prediction
- `--all` → walk every page in `predictions/open/`
- `--stale` → only predictions open past their own test-plan horizon (or >14d with no new evidence)
- no argument → walk all open predictions and triage which are testable now

## 1. Read the bet (per prediction)

Read the prediction, its **mechanism**, **test plan**, **fail condition**, and any **observations already recorded**. Count the independent observations that already exist (call it `n_prior`).

## 2. Procure the observation — the step every other skill punts

This is the heart of the skill. Execute the test plan literally, using whatever the project allows right now:

- if the test plan names a command (bench, build, e2e, unit test) → **run it**
- if it names a corpus/codebase query → grep/read the code or transcripts and compute the answer
- if it names a behavior to observe → reproduce it and record what happens
- if it is an LLM behavioral claim → run the prompt/scenario on the stated model

Record the **raw result** (numbers, output, what actually happened), not a summary judgment. `wiki-compile` resolves predictions whose evidence is already written down; `resolve` *generates* that evidence. If a prediction has no executable test plan, that is itself a finding — go to §5.

## 3. The n>=3 gate — the reason this skill exists

Count **independent** observations supporting a verdict. Independent means different runs, contexts, days, codebases, or models — NOT the same observation counted twice. For LLM behavioral claims, independent requires a different session/model (protocol Rule 8).

```
n = n_prior + new independent observations this run
if n < 3:
  record the new observation in the prediction body, update last_verified,
  KEEP status: open. Report "n of 3 — not resolving yet."
  DO NOT move the file.
```

Clean numbers from one run are an **observation, not a verdict**. The conjecture self-wiki once resolved 4 predictions at n=1, got caught, and reverted (`predictions/open/resolution-needs-gate.md`) — that completion bias is exactly what this gate exists to stop. `--force` may override the gate, but you must log the override and the reason.

## 4. Resolve (only past the gate)

Compare the observations to the **fail condition**:

- fail condition NOT met across n>=3 → **confirmed**
- fail condition met → **refuted**
- still inconclusive after the test-plan horizon → **graveyard** with a trigger ("reactivate when ___")

Then, for confirmed/refuted:

1. Write the scorecard YAML **into the file** (not just chat — chat scorecards evaporate):
   ```yaml
   prediction_accuracy: exact | partial | wrong
   surprise: low | medium | high
   what_the_prediction_missed: []
   ```
2. Change `status: open` → `confirmed`/`refuted`. Update `last_verified` and ensure `tested_on`/`context` are filled.
3. Move the file to `predictions/confirmed/` or `predictions/refuted/`.
4. Update `index.md`.

**Refutations are first-class.** A refuted prediction with a clear "what it revealed" is worth more than a lucky confirmation — never delete it, and never **soften a contradiction into a caveat**. If the evidence disconfirms, you REFUTE; you do not append "but…" to the body and leave `status: confirmed`. (That is the nozzz failure mode: a wrong belief keeping its authority as a footnote.)

5. **Observe** — ask whether the resolved bet reveals a pattern worth a `knowledge/` page (T0 at n>=3). Extract it if so; don't force one if not.

## 5. Long-horizon / unreachable predictions

If a prediction cannot be tested on the relevant timescale (a 60-day outcome, behavior of code that doesn't exist yet), do not let it inflate `open/` silently. Either:
- add a note to the body: `long-horizon: testable after <condition/date>`, or
- move it to `graveyard/` with an explicit trigger condition.

Make un-reachability **visible** so the governor's open count means something.

## 6. Running unattended / on a schedule

The observation supply should not depend on the maintainer doing unrelated work. `resolve` is designed to be wrapped:

- `/loop` or `/schedule` can run `/conjecture:resolve --all` on a cadence (e.g. weekly). Pair it with the governor: `govern` *measures* the trend, `resolve` *procures* the observations it measures.
- For a wiki with a native engine (a friction log, CI, an autonomous loop), the highest-leverage move is to wire that engine to file observations or call `resolve` — that is how a stalled wiki starts closing its loop without a human authoring each resolution.

## 7. Log and report

Append to `wiki/log.md`:
```
## [YYYY-MM-DD] resolve | N tested, X confirmed, Y refuted, Z held (n<3), W long-horizon
```

Report: each resolution with its verdict + surprise; what stayed open and why (n<3, or not yet testable); any knowledge extracted. If this run pushes the wiki across a ~20-resolution boundary, suggest `/conjecture:wiki-compile` to recompute the governor trend.

## Rules

1. Resolution is an action. If the test can be run, run it — don't report it as "needs a new experiment."
2. The n>=3 gate is the point. n=1 clean numbers are an observation; hold the prediction open, don't close it.
3. Independent observations only. The same observation counted twice is n=1.
4. Refutations are first-class — never delete, never soften into a caveat. Disconfirming evidence is a status change, not a footnote.
5. Scorecards go in the FILE.
6. Unreachable predictions get flagged or graveyard'd — never left to inflate `open/`.
7. Every claim carries context: `tested_on`, date, codebase, conditions (Rule 8).
