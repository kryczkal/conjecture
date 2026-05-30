---
created: 2026-05-29
last_verified: 2026-05-30
type: prediction
status: open
context: [protocol-design, loop-stall, usage-audit]
tested_on: [claude-opus-4-8]
long-horizon: testable after the pump runs on a stalled wiki (swarm/jarvis) for ~4 weeks
---

# A resolution pump — not more prediction surface — is what closes a stalled Conjecture loop

Across 9 wikis using the engine, the loop runs fully ONLY where an external, recurring ground-truth engine sits next to the wiki and emits observations as a byproduct of other work (iris: skill-evolution tournaments; vimx: h2h bench harness + a refuting owner; conjecture-self: the maintainer's build sessions). Where that engine is absent (screenx), in a foreign grammar (swarm/jarvis ship+friction loops, frozen by migration), or too young (nozzz), the loop runs one-directionally: theory→open, history→confirmed, with no return arc. The prediction: adding a **resolution pump** — a `/conjecture:resolve` skill (or a scheduled wiki-compile mode) whose job is to walk `predictions/open/`, actively procure the observation each one needs (run the bench, query the corpus, run the build/e2e, diff the code against the claim), and resolve it with an n≥3 gate — will close the loop in a currently-stalled wiki *without a human authoring each resolution*.

## Mechanism

Every resolution path in the current system is **passive**. `grep -rniE 'resolve open|run the test|fetch evidence|pull observation|retest' skills/` returns empty. wiki-compile §1 only resolves predictions whose evidence is *already in the page body*, and explicitly classifies "requires a new experiment" as "Cannot fix". The governor only *measures* after resolutions exist. predict/ingest/migrate/init all *create* pages. No operation's job is to take an open prediction and go get the observation that closes it. So the "observe" step is silently outsourced to an external engine that must exist, run continuously, and speak prediction-grammar — three conditions that hold only in the flagship instances. The proof that the bottleneck is resolution and not prediction: **iris ran the fullest loop in the fleet (59 confirmed / 12 refuted) with ZERO predict-skill invocations.** Predict-skill usage does not predict loop health; engine presence does.

## Decisions

- If true: build `/conjecture:resolve` as the keystone skill — it implements the protocol's `Test` operation, writes the scorecard into the FILE (not chat), bakes in the n≥3 gate ([[resolution-needs-gate]] already predicts this is needed), and is schedulable so the observation supply does not depend on the maintainer doing unrelated work. Demote the `predict` skill (redundant — the loop runs without it).
- If false: the missing piece is something else (an executable test-plan discipline, or simply more predict invocations), and a resolve skill would not move a stalled wiki.

## Test plan

1. Add a resolution pump (skill or scheduled mode) to a currently-stalled wiki — swarm or jarvis are the cleanest cases (0 predictions, a live native engine to wire in).
2. Over ~4 weeks, observe whether `predictions/confirmed/` and `predictions/refuted/` gain entries.
3. Distinguish: did entries appear because the pump procured observations autonomously, or because a human hand-authored each resolution?

## Fail condition

Refuted if EITHER: (a) a resolution pump is added to a stalled wiki and over ~4 weeks no predictions resolve *without* a human manually authoring each resolution; OR (b) the loop still doesn't move because the predictions are unresolvable-in-principle (no test the project can run) — which would relocate the root cause to "predictions written without an executable test plan" rather than "no actor executes the test". Also weakened if a stalled wiki spontaneously starts resolving purely because the operator typed `/conjecture:predict` more often, with no new engine and no resolve mechanism (that would refute "engine, not predict-skill, is the variable").

## Observations (n=cross-project, audit 2026-05-29)

**iris (full loop):** 59 confirmed / 12 refuted / 12 open, 100% scorecard discipline — driven by evolve-tournaments + transcript mining, 0 predict-skill invocations. Open-inflation only at the engine's blind edge (8/12 open are dev-practice bets needing a live build session the engine can't reach).

**vimx (full loop):** bench harness + a human who refutes mid-implementation; 8/8 resolved carry scorecards; self-diagnosed "bottleneck is testing, not hygiene — the learning pipeline is stalled at the run-the-experiment step".

**nozzz (half loop):** 18 confirmed / 0 refuted; disconfirming evidence (failed protocols, "sleep-avoidance paradox") was appended as caveats to confirmed pages instead of refuting them — no skill defines a "reopen/refute on contradiction" action.

**swarm / jarvis (empty):** migrate faithfully produced 0-prediction knowledge dumps; both have a live native learn-from-failure loop (F-numbered frictions) that was never wired to emit predictions; the schema sits beside the real loop, not inside it.

**conjecture-self (partial):** resolved 6 predictions in one 2.5h build session, then dormant 4 days — resolution happens exactly as fast as the maintainer does skill-dev work, and stalls the instant it stops.

## Milestone (2026-05-30) — the "if true" action was executed, NOT the bet resolved

The keystone skill now exists: the `resolve` skill was renamed to `test-hypotheses` (the protocol's `Test` operation, with the n≥3 gate baked into §3 and scorecards written into the file), and the `predict` skill was deleted as redundant — exactly the "If true → build the pump, demote predict" decision this prediction prescribes. **This is acting on the prediction on faith, not confirming it.** Building the pump is a precondition; the bet is about whether the pump *closes a stalled loop without a human authoring each resolution*, which can only be observed by running it on a stalled wiki (swarm/jarvis) over ~4 weeks. Status stays **open** — long-horizon. Resolving here would be the precise completion bias [[resolution-needs-gate]] exists to stop. Trigger: re-test after the first scheduled `/conjecture:test-hypotheses --all` cadence has run against a stalled wiki for the test-plan horizon.
