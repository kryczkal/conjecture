# Log

Chronological record of wiki operations. Append-only.

## [2026-05-25] test | live migrate run on vimx wiki — 4 predictions resolved

Ran migrate skill on vimx wiki (67 pages moved, 5 skipped, 1 minor fix). Built skill via /evolve (3 screening agents + adversarial validator, then bred winner).

**Resolved:**
- compile-migrate-split → CONFIRMED (exact). ~30% overlap with compile. Classification engine is unique to migrate.
- classification-requires-body → CONFIRMED (partial). 7/67 pages (10.4%) changed by body. Barely survived 90% fail threshold. But the 7 pages were disproportionately important.
- iterative-migration → REFUTED (wrong). First-pass accuracy 97%. The mechanism (interdependencies cause cascading issues) did not materialize. Batch cross-ref repair handled it.
- demote-claims-on-migration → CONFIRMED (exact). 0/8 demoted pages re-promoted. All demotions defensible.

**Still open:** scorecards-only-from-evidence (not directly tested by this run).

**Governor:** 4 resolved (3 confirmed, 1 refuted). 3 exact, 1 partial, 1 wrong. Accuracy: 60% exact, 80% confirmed. Next fire at ~20. Rate: 4 in day 1.

## [2026-05-25] ingest | brainstorm session — compile/migrate split

Source: brainstorm conversation + docs/specs/2026-05-25-compile-migrate-split.md. Filed 5 open predictions about how to build the migrate skill and update compile. All testable against vimx wiki (46 legacy pages) and iris wiki.

**Filed:** compile-migrate-split, classification-requires-body, scorecards-only-from-evidence, iterative-migration, demote-claims-on-migration.

**Skipped (surprise < 2):** None — all 5 conjectures were novel to this fresh wiki.

**Unfiled (surfaced but not yet a prediction):** "Distill works on legacy pages" — implicit assumption in the spec. Distill's templates expect conjecture-native structure; legacy pages have arbitrary structure. Could break the migrate pipeline's third step. Worth filing after the first migrate test reveals whether this is real.

## [2026-05-25] init | Conjecture wiki initialized

Scaffolded wiki/ with the Conjecture protocol. Ready for first prediction.
