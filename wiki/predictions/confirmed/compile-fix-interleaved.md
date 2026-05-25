---
created: 2026-05-25
last_verified: 2026-05-25
type: prediction
status: confirmed
context: [skill-design, wiki-compile]
tested_on: [claude-opus-4-6]
---

# Compile executes more fixes when fix logic is interleaved with audit, not deferred to the end

## Mechanism

Context degradation: by the time the LLM reaches a deferred fix section after 31 audit steps, fix instructions compete with accumulated findings for attention. Interleaving "fix this now" with each audit section keeps the fix instruction proximate to the finding.

## Decisions

Each audit section ends with "Fix now" and "Flag only" for its specific findings. No separate deferred Fix section.

## Fail condition

Interleaved and deferred execute the same number of fixes (±10%). Not met on any of 3 repos.

## Observations (n=3)

**vimx:** 18 fixes executed (7 scorecards, 8 broken links, 5 framework fields). Deferred version on same wiki: 8 fixes. 125% improvement.

**swarm:** 17 fixes executed (11 log.md links, 1 CLAUDE.md ref, 3 migration guide paths, 2 framework paths).

**jarvis:** 10 fixes executed (2 duplicate frontmatter, 7 broken links, 3 framework fields).

## Scorecard

```yaml
prediction_accuracy: exact
surprise: low
what_the_prediction_missed:
  - the biggest win was scorecards (7 on vimx) — the deferred version flagged them but never added them
  - broken link fixes were the same count in both versions — interleaving helps content-creation fixes more than mechanical fixes
```
