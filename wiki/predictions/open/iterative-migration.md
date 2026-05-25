---
created: 2026-05-25
last_verified: 2026-05-25
type: prediction
status: open
context: [skill-design, brainstorm]
---

# Iterative migration produces better results than single-pass

Running migrate repeatedly on a wiki until compile reports zero schema violations produces higher-quality migration than attempting a perfect single pass. Each iteration catches issues the previous pass introduced (broken cross-refs from moves, structural rewrites that lost content, misclassifications revealed by seeing pages in their new context).

## Mechanism

Migration is a complex transformation with interdependencies — moving page A changes the context for classifying page B (which referenced A). A single pass can't account for all these interactions. Iteration lets the system converge: each pass fixes issues from the previous pass until the wiki stabilizes.

## Decisions

- If true: migrate is designed for re-running. It must detect already-migrated pages and skip them. The workflow is: migrate → compile → migrate → compile → done.
- If false: migrate should aim for single-pass correctness. Iteration adds overhead and confuses the user about whether the wiki is in a consistent state between runs.

## Test plan

1. Run migrate once on vimx wiki. Run compile. Count findings.
2. Run migrate again. Run compile again. Count findings.
3. Compare: did the second pass reduce findings? By how much?

## Fail condition

First-pass accuracy is >95% (compile finds <3 issues after first migrate run on a 46-page wiki). If the first pass is that clean, iteration adds overhead without meaningful improvement.
