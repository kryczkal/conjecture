---
created: 2026-05-25
last_verified: 2026-05-25
type: prediction
status: confirmed
context: [skill-design, brainstorm]
tested_on: [claude-opus-4-6]
---

# Iterative migration produces better results than single-pass

Running migrate repeatedly until compile reports zero violations produces higher-quality migration than a single pass — but for a different reason than predicted.

## Mechanism (REVISED)

~~Original: interdependencies between pages cause cascading issues.~~ Wrong on all 3 tests. Batch cross-ref repair handled interdependencies in one sweep.

**Actual mechanism:** structural complexity drives first-pass accuracy down. Nested archives, binary asset references, deep relative paths, many non-claim page types — these features compound errors that a second pass catches. The driver is wiki messiness, not page interdependency.

## Decisions

Migrate is designed for re-running. Skip already-migrated pages. The workflow is: migrate → compile → migrate → compile → done. Single-pass is sufficient for clean, simple wikis (<50 pages, flat structure).

## Fail condition

First-pass >95% on a ~46-page wiki. Met on vimx (97%), not met on swarm (95%) or jarvis (90%).

## Observations (n=3)

**vimx:** 97% first-pass (3 minor issues). **swarm:** ~95% (TODO markers, extractable predictions, inline references). **jarvis:** ~90% (evidence TODOs on 20 pages, 8 over-demoted, cross-ref depth issues, broken session paths).

Pattern: accuracy = f(structural complexity). 97% on clean wiki, 90% on messy wiki.

## Scorecard

```yaml
prediction_accuracy: partial
surprise: high
what_the_prediction_missed:
  - the mechanism was completely wrong (interdependencies → structural complexity)
  - the relationship is continuous, not binary
  - vimx nearly refuted the prediction — premature n=1 refutation was corrected
  - the skill should still be designed for re-running but the reason is different than predicted
```
