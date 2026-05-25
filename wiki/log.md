# Log

Chronological record of wiki operations. Append-only.

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
