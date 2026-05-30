# Log

Chronological record of wiki operations. Append-only.

## [2026-05-31] evolve + test | exploit §4 fix kept; §1 skip-rule bug found on real wikis and fixed

`/evolve` on exploit + a before/after test on 3 real knowledge-rich wikis (iris, nozzz, vimx) via 6 dry-run subagents (3 before, 3 after), claude-opus-4-8.

**Evolve (one section):** attribution put the weakness in §4 (act-vs-propose) — worst friction in 3/3 before-runs (single axis; no rule for the split artifact-edit-vs-human-action case; the table contradicted the prose on cross-context/deferred changes). Rewrote §4 into a two-step split + escalation triggers (cross-context / source-defers-or-contests / not-contained). After-runs: §4 clean on iris ("the sharpest instruction in the skill"), narrower on nozzz, praised on vimx ("pre-solved a real adjudication"). Kept (10/12 → 12/12 on the act/propose axis).

**Test-found bug (second, separate fix):** with §4 fixed, 3/3 after-runs converged on a new worst friction — §1's `applications:`-backlink skip-rule is a no-op on any wiki not built by exploit, and on ingested-from-code wikis (vimx) it reports already-shipped knowledge as false gaps. Fixed §1 to check the **artifact, not the backlink**, and to scope out open predictions as sources.

**First observation for [exploit-closes-outer-loop]:** all 3 wikis had `grep applications:` = empty (0 of all confirmed pages applied) AND each surfaced ≥3 real high-leverage unapplied gaps; vimx was shipping a *refuted* belief (dead LiteLLM path). Premise supported, n=1, not resolving (gate held).

## [2026-05-30] build | exploit skill — the outer loop (wiki → project)

Built `/conjecture:exploit`: the mirror of ingest. ingest pulls the world into the wiki; exploit pushes confirmed knowledge back out to the project (code, benchmark, strategy doc, process) and un-applies refuted beliefs. Core invariant: **every application is itself a prediction** (confirmed-in-context is not true-everywhere), filed with a fail condition; execution calibrates to blast radius (act on reversible/Claude-doable changes, propose high-stakes/real-world ones). It is the enforcement arm of the Action layer — checks whether a page's claimed decision actually changed the artifact.

Extended the canonical protocol: new **The outer loop** section, **Exploit** operation, `applied_from`/`applications` frontmatter (symmetric to frameworks' `predictions_generated`), and a knowledge-fertility note (a confirmed page with `applications: []` hasn't changed the project).

**Filed:** [exploit-closes-outer-loop](predictions/open/exploit-closes-outer-loop.md) (open) — the export-gap bet, with a fail condition, rather than promoting the skill to law on vibes. First observation comes from a before/after `/evolve` test on iris/nozzz/vimx (in progress).

## [2026-05-30] test-hypotheses | 4 walked, 1 tested, 0 confirmed, 0 refuted, 2 held (n<3), 1 long-horizon, 1 blocked-on-user

First run of the renamed skill (`resolve` → `test-hypotheses`; `predict` skill deleted as redundant). Walked all 4 open predictions and triaged for testability now.

- **compile-redirects-to-migrate** — tested. Built a synthetic 50/50 mixed-schema wiki (3 valid + 4 legacy types + a `raw/` page) and ran §0 pre-flight literally: counted 4 invalid > 3 → fired the redirect as a single directive (not 4 per-page flags); `raw/` correctly excluded; boundary confirmed at `>3` (3 invalid → proceeds). First test of the positive case, but n=1 and *synthetic* → held open, not resolved. Fail condition is also now partly stale (the no-redirect counterfactual can't be observed since the redirect is built in).
- **resolution-pump-closes-stalled-loops** — long-horizon. Today's rename/delete executed its "if true → build the pump, demote predict" action, but the bet (does the pump close a *stalled* loop unaided over ~4wk?) is unobservable until run on swarm/jarvis. Logged the milestone, flagged `long-horizon`, kept open. Resolving here would be the exact completion bias resolution-needs-gate guards against.
- **resolution-needs-gate** — held at n=2. Noted that the gate is now skill-enforced (§3 of test-hypotheses), which confounds the original rules-only fail condition; flagged for fail-condition refinement on next compile.
- **scorecards-only-from-evidence** — blocked on user: its test plan needs the user's *independent* scorecards for vimx's 6 resolved predictions to compare against LLM-inferred ones. Cannot procure solo.

Net: 0 resolutions. The discipline held — a single clean synthetic run did not close a bet.

## [2026-05-29] audit | 9-wiki usage audit — filed resolution-pump prediction, scaffolded Axiom zero

Forensic audit of how the Conjecture engine is used across all 9 wikis + their session transcripts (23-agent workflow, adversarially verified). Core finding: the engine has no **resolution pump** — every skill implements only the intake half of the loop; nothing actively procures the observation that closes an open prediction. The loop runs fully only where an external recurring engine emits observations as a byproduct (iris, vimx, conjecture-self). Proof: iris ran the fullest loop in the fleet with ZERO predict-skill invocations — engine presence, not predict usage, predicts loop health.

**Filed:** [resolution-pump-closes-stalled-loops](predictions/open/resolution-pump-closes-stalled-loops.md) (open) — the keystone recommendation, filed as a prediction with a fail condition rather than promoted to skill law on vibes.

**Scaffolded:** axioms/self-correction.md (Axiom zero) — was referenced by CLAUDE.md line 5 but missing here and in every other wiki. The meta-governor could not ground out without it.

**Skill edits applied (working tree, uncommitted):** init now copies the bundled scaffold (kills hardcode-drift, ships the axiom); protocol backported iris's governor computation + coverage audit; ingest gained an explicit resolve/refute action + "never soften a contradiction into a caveat" rule; wiki-compile gained softened-contradiction + stale-open flags and a flat-at-capacity governor refinement; migrate gained a no-legacy-wiki pre-flight guard + a zero-prediction loop-ignition warning; distill gained a batch-mode spec. predict left unchanged (verdict: redundant — the loop runs without it).

## [2026-05-25] test | evolved compile on vimx/swarm/jarvis — 2 confirmed, 1 open

Ran evolved wiki-compile (governor-first, interleaved fixes, compressed audit) on all 3 repos.

**Confirmed:** compile-fix-interleaved (exact, n=3: 18/17/10 fixes executed vs ~8 deferred). compile-fewer-deeper (exact, n=3: 4/6 actionable across all repos, 0 actionable misses).

**Open:** compile-redirects-to-migrate (n=3 pre-flights passed but fail condition untested — all wikis already migrated). Edge case found: pre-flight should exclude raw/ pages. Fixed.

**Governor:** 6 confirmed (4 exact, 2 partial), 0 refuted. Resolution rate: 6 in day 1. 3 open predictions remain. Next fire at ~20.

## [2026-05-25] lint | first wiki-compile

Governor: 4 confirmed, 0 refuted, 4 scorecards (2 exact, 2 partial). Learning rate positive. Neither open prediction confirmable without new tests. Next governor fire at ~20.

Fixed: spec moved to raw/ (not a conjecture type). Added evidence field to knowledge page frontmatter.

Flagged: demote-claims needs user review (not just LLM). All predictions tested on single model only. No frameworks filed (implicit philosophy not made explicit).

Top action: run the scorecards-only-from-evidence test on vimx's 6 resolved predictions — 15 minutes, resolves an open prediction.

## [2026-05-25] ingest | evolved skill live test — knowledge bumped to T1

Source: evolved migrate skill live test on vimx (n=4). The three evolve mutations (complexity assessment, evidence-adaptive maturity, depth-aware cross-ref) all validated.

**Changed:** migration-accuracy-scales-with-complexity bumped T0→T1. The mechanism (depth-change cross-ref breakage) was addressed by intervention (three-sub-pass repair), and the intervention improved cross-ref accuracy from 33%→93% on jarvis dry-run. The mechanism is now tested, not just proposed.

**Skipped:** demote-claims-on-migration refinement (the evolved maturity rules implement it, but it's already captured in the prediction's scorecard). Complexity assessment MODERATE score for vimx (minor correction, surprise 1).

## [2026-05-25] ingest | n=3 resolution — 4 confirmed, 1 knowledge filed

Applied protocol rules to n=3 observations. The protocol's prediction resolution threshold is in the Test operation ("run the test, move to confirmed/refuted"), NOT the knowledge maturity ladder (T0 n>=3). With n=3 and clear fail conditions, 4 predictions are resolvable.

**Confirmed:** compile-migrate-split (exact), classification-requires-body (exact), iterative-migration (partial — mechanism wrong), demote-claims-on-migration (partial — too aggressive for shipped-code wikis).

**Knowledge filed:** migration-accuracy-scales-with-complexity (T0, n=3). First-pass accuracy inversely correlates with structural complexity.

**Still open:** scorecards-only-from-evidence (n=2, hard test not done), resolution-needs-gate (n=2, one failure + one self-correction).

**Governor:** 4 confirmed, 0 refuted, 4 scorecards. Accuracy: 2 exact, 2 partial, 0 wrong. Learning rate: positive (predictions improved from brainstorm to observation — iterative-migration's mechanism was revised, demotion rule got nuanced). Next fire at ~20.

**Skipped:** resolution-needs-gate observation update (this ingest is evidence for its fail condition — filed inline, not a separate change).

## [2026-05-25] test | migrate on swarm (n=2) + jarvis (n=3) — parallel runs

Ran migrate on swarm (41 pages, 8 types) and jarvis (78 pages, 11 types) in parallel. All predictions remain open with n=3 observations recorded.

**Key findings:**
- classification-requires-body: strengthened. 17% on swarm, 18% on jarvis vs 10% on vimx. Messier wikis depend more on body reading.
- iterative-migration: genuinely mixed. 97%/95%/90% first-pass across clean→messy wikis. The mechanism was wrong but a new pattern emerged: accuracy correlates with structural complexity.
- demote-claims-on-migration: jarvis showed 23.5% arguably over-demoted (component pages with shipping evidence). Blanket T0 may need nuance.
- scorecards-only-from-evidence: jarvis had 2 informal hypotheses that were tested but correctly got no scorecards.

## [2026-05-25] correction | premature resolution reverted — n=1 is not enough

Prematurely resolved 4 predictions as confirmed/refuted based on a single migrate run on vimx. Reverted all to open. One repo, one run, one model is n=1 — not confirmable per protocol (T0 requires n>=3). Observations recorded on each prediction page but no scorecards filed. This is the exact failure mode conjecture was built to prevent.

## [2026-05-25] test | live migrate run on vimx wiki — first observation

Ran migrate skill on vimx wiki (67 pages moved, 5 skipped, 1 minor fix). Built skill via /evolve (3 screening agents + adversarial validator, then bred winner). All 5 predictions remain OPEN with n=1 observations.

**Observations (direction only):**
- compile-migrate-split: 30% overlap → supports
- classification-requires-body: 10.4% body-changed → weakly supports (0.4% margin)
- iterative-migration: 97% first-pass → against
- demote-claims-on-migration: 0% re-promoted → supports (LLM review, not user)
- scorecards-only-from-evidence: not tested

**Next:** run on iris + one more repo. Get user review on demotions.

## [2026-05-25] ingest | brainstorm session — compile/migrate split

Source: brainstorm conversation + docs/specs/2026-05-25-compile-migrate-split.md. Filed 5 open predictions about how to build the migrate skill and update compile. All testable against vimx wiki (46 legacy pages) and iris wiki.

**Filed:** compile-migrate-split, classification-requires-body, scorecards-only-from-evidence, iterative-migration, demote-claims-on-migration.

**Skipped (surprise < 2):** None — all 5 conjectures were novel to this fresh wiki.

**Unfiled (surfaced but not yet a prediction):** "Distill works on legacy pages" — implicit assumption in the spec. Distill's templates expect conjecture-native structure; legacy pages have arbitrary structure. Could break the migrate pipeline's third step. Worth filing after the first migrate test reveals whether this is real.

## [2026-05-25] init | Conjecture wiki initialized

Scaffolded wiki/ with the Conjecture protocol. Ready for first prediction.
