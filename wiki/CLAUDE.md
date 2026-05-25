# Conjecture — Wiki Protocol

Every page is a bet — a claim about how something works that is specific enough to be wrong. The wiki succeeds when it makes you wrong faster. Being wrong is the input. Being less wrong over time is the output. The rate of becoming less wrong is the only metric that matters.

**Axiom zero:** A biased system can self-correct faster than it drifts, even though it cannot verify its own foundations from outside. See `axioms/self-correction.md`.

## The learning atom

All learning is one operation: predict → observe → notice the gap → update the model that generated the prediction. This wiki is the loop's memory. Predictions are pages. Observations are test results. Updates are revisions.

## The governor

The loop alone is not enough. A system that updates without tracking whether updates improve predictions is changing without learning.

```
every ~20 confirmed/refuted predictions:
  is prediction accuracy improving over time?
  yes → the system is learning. continue.
  no  → the update mechanism is the problem.
        don't make more predictions.
        challenge your presuppositions.
```

Every confirmed or refuted prediction gets a scorecard:

```yaml
prediction_accuracy: exact | partial | wrong
surprise: low | medium | high
what_the_prediction_missed: []
```

The trend in these scorecards IS the system's learning rate.

## The meta-governor

The governor has presuppositions too. What counts as "accurate"? What counts as "improving"? Those ground out in axiom zero.

Periodically challenge the governor's own assumptions: state them explicitly, argue against them, revise if the challenge produces something better.

The presupposition test: take 3-5 assumptions the system operates under. For each, make the strongest possible case against it. Score: REVISE (productive revision), KEEP (challenge failed to move it), or BREAK (coherence lost). If most produce REVISE → the system is self-correcting.

## Structure

```
wiki/
├── CLAUDE.md           this file
├── index.md            what the system currently believes
├── log.md              chronological record (append-only)
├── predictions/
│   ├── open/           untested bets
│   ├── confirmed/      bets that survived testing (with scorecards)
│   └── refuted/        bets that were wrong (first-class — never delete)
├── knowledge/          observations, findings, patterns, benchmarks
├── frameworks/         interpretive lenses that generate predictions (not testable themselves)
├── axioms/             constitutional rules (challengeable but not testable)
├── graveyard/          latent knowledge with trigger conditions
└── raw/                immutable inputs (transcripts, logs, seeds)
```

## Frontmatter (required)

```yaml
---
created: YYYY-MM-DD
last_verified: YYYY-MM-DD
type: prediction | knowledge | framework | axiom | graveyard
status: open | confirmed | refuted        # predictions
        active | superseded | archived     # knowledge, frameworks
tested_on: []      # model IDs (e.g., claude-sonnet-4-6, claude-opus-4-6)
context: []        # where observed (e.g., skill-evolution, code-review)
---
```

`tested_on` and `context` are required for LLM behavioral claims.

## Evidence maturity ladder

- **T0 — Pattern** (n>=3, one context): Actionable for operational decisions. No mechanism required.
- **T1 — Draft mechanism** (proposed WHY): Actionable for targeted fixes.
- **T2 — Tested mechanism** (intervention confirmed the WHY): Transferable to new contexts.
- **T3 — Cross-context** (replicated across models/domains): Candidate for promotion to framework-level insight.

T0 is first-class. You don't need a mechanism to act on a pattern. You need a mechanism to generalize.

## The five layers (every mature page should accumulate these)

| Layer | Question | What to check |
|-------|----------|---------------|
| Action | What decision does this change? | If removing the page changes no behavior, delete it |
| Mechanism | Why does this happen? | The causal chain, not just correlation |
| Testing | What would prove this wrong? | Fail condition must be stated |
| Convergence | How broadly confirmed? | tested_on, context, replication count |
| Honesty | What is this blind to? | Framework that produced it, its blind spots |

## Page types

**Prediction** — the core unit. A bet about how something works, specific enough to be wrong. Must have: prediction (one sentence), mechanism (why you expect this), decisions (what changes if true), test plan (how to check), fail condition (what kills it). When confirmed/refuted, add the scorecard.

**Knowledge** — observations with evidence. Tagged with maturity tier (T0-T3). Encompasses findings, decisions, sources, benchmarks. The maturity ladder is the promotion path, not separate types.

**Framework** — value commitments and interpretive lenses. They generate predictions but are not themselves testable. Evaluated by fertility: does this framework produce predictions that turn out to be accurate? A framework that hasn't generated a prediction in 30 days is dead weight. Delete it. Must have: what it commits to, what it's blind to, what predictions it has generated.

**Axiom** — constitutional rules the system operates under. Can't be tested within the system because the system depends on them. CAN and MUST be challenged periodically. Must have: the rule, why it exists, what would make you abandon it (`abandon_when`), when it was last challenged.

**Graveyard** — knowledge that doesn't change a current decision but has a known trigger condition: "this becomes relevant when ___." Reviewed periodically. If the trigger can no longer occur, delete.

## Operations

**Predict** — write `predictions/open/<slug>.md`, update `index.md`, append `log.md`.
**Test** — run the test. Record what happened. Move to `confirmed/` or `refuted/`. Add scorecard. Extract findings to `knowledge/`.
**Observe** — when you notice a pattern (n>=3), file it as T0 knowledge. Ask: does this predict something untested?
**Challenge** — pick an axiom or framework. Argue the strongest case against it. REVISE/KEEP/BREAK.
**Govern** — every ~20 confirmed/refuted predictions: compute the accuracy trend. If improving → continue. If flat → challenge axioms. If declining → something is fundamentally wrong.
**Compile** — periodic lint. Check: stale pages (>30d unverified), orphans, broken links, predictions stuck open >14d, frameworks with no downstream predictions, graveyard pages past expiry, axiom challenge dates.

## Rules

1. Every change is a conjecture first.
2. Conjectures must have fail conditions.
3. Refutations are first-class. Never delete. A wrong prediction with a clear explanation of WHY it was wrong is more valuable than a correct prediction that got lucky.
4. Patterns (T0) are actionable without mechanisms.
5. The system declares its own epistemology and its blind spots.
6. Less is more. Every page earns its keep or gets deleted.
7. The system examines itself. Axioms get challenged. The governor tracks learning rate. When learning stalls, look inward before looking outward.
8. All claims carry context: model, date, codebase, conditions. No universal truths. Only conjectures that haven't been refuted yet, in the contexts where they've been tested.

## Read before acting

1. `index.md`
2. `log.md` last 5 entries
3. Relevant linked pages
