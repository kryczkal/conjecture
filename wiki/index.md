# Index

What this project currently believes. Updated as predictions are confirmed/refuted and knowledge accumulates.

## Open predictions

- [scorecards-only-from-evidence](predictions/open/scorecards-only-from-evidence.md) — never hallucinate scorecards. **n=2, hard test not yet done.**
- [resolution-needs-gate](predictions/open/resolution-needs-gate.md) — protocol needs enforcement gate. **n=2: one failure, one self-correction. 8 more resolutions needed.**
- [compile-redirects-to-migrate](predictions/open/compile-redirects-to-migrate.md) — compile should redirect unmigrated wikis to migrate. **n=4: mixed-schema positive case finally tested (synthetic), redirect + raw/-exclusion fire correctly; need real mid-migration wikis to resolve.**
- [resolution-pump-closes-stalled-loops](predictions/open/resolution-pump-closes-stalled-loops.md) — the missing piece is a resolution pump, not more prediction surface. **Pump built (test-hypotheses skill, predict deleted); long-horizon — needs ~4wk run on a stalled wiki.**
- [exploit-closes-outer-loop](predictions/open/exploit-closes-outer-loop.md) — the gap between a learning wiki and an improving project is a missing *export* operation, not missing knowledge. **exploit skill built; long-horizon — first observation from before/after test on real wikis.**

## Confirmed predictions

- [compile-migrate-split](predictions/confirmed/compile-migrate-split.md) — split produces better skills. **n=3, exact.**
- [classification-requires-body](predictions/confirmed/classification-requires-body.md) — frontmatter type alone insufficient. **n=3, exact.**
- [iterative-migration](predictions/confirmed/iterative-migration.md) — iteration beats single-pass on complex wikis. **n=3, partial (mechanism wrong).**
- [demote-claims-on-migration](predictions/confirmed/demote-claims-on-migration.md) — demote claims to T0. **n=3, partial (too aggressive for shipped-code).**
- [compile-fix-interleaved](predictions/confirmed/compile-fix-interleaved.md) — interleaved fixes execute 2x more than deferred. **n=3, exact.**
- [compile-fewer-deeper](predictions/confirmed/compile-fewer-deeper.md) — compressed audit finds same actionable issues. **n=3, exact.**

## Refuted predictions

(none)

## Knowledge

- [migration-accuracy-scales-with-complexity](knowledge/migration-accuracy-scales-with-complexity.md) — T1: accuracy scales with complexity. Mechanism tested: depth-aware cross-ref repair improved accuracy.

## Raw

- [2026-05-25-compile-migrate-split](raw/2026-05-25-compile-migrate-split.md) — spec from brainstorm session

## Frameworks

(none yet)

## Axioms

- [self-correction](axioms/self-correction.md) — Axiom zero: a biased system self-corrects faster than it drifts if challenge mechanisms are built in. **last_challenged: never.**
