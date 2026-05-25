---
created: 2026-05-25
last_verified: 2026-05-25
type: prediction
status: refuted
context: [skill-design, brainstorm]
tested_on: [claude-opus-4-6]
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

## Scorecard

```yaml
prediction_accuracy: wrong
surprise: medium
what_the_prediction_missed:
  - first-pass accuracy was ~97% — only 3 minor issues remained (frontmatter miss, directory-level cross-ref, raw type string)
  - the mechanism was wrong — interdependencies between pages did NOT cause cascading issues in practice
  - the skill's cross-ref repair pass (grep old paths, replace with new) handled interdependencies in a single sweep
  - iteration is still useful for SAFETY (revert and retry) but not for ACCURACY
```

**Evidence:** First migrate run on vimx (67 pages) achieved ~97% accuracy. The 3 remaining issues were minor mechanical misses (one frontmatter field, one directory-level link, one non-schema type string in raw/), not classification errors or content loss. A second run would clean those up but the value is marginal. The predicted mechanism (moving page A breaks classification of page B) did not materialize — the batch cross-ref repair handled all interdependencies.
