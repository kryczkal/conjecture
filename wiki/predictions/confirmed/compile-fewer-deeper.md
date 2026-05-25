---
created: 2026-05-25
last_verified: 2026-05-25
type: prediction
status: confirmed
context: [skill-design, wiki-compile]
tested_on: [claude-opus-4-6]
---

# Fewer compile checks produce deeper findings than the current 31-step audit

## Mechanism

31 steps forces the LLM to spread attention equally. A compressed audit targeting the highest-impact checks (governor, evidence, knowledge→prediction pipeline) goes deeper because more context budget is available per check.

## Decisions

Compress compile to ~18 focused checks across 5 sections (down from 31 across 7). Cut five-layer assessment, axiom/graveyard deadlines. Keep governor, evidence quality, and knowledge→prediction pipeline at full depth. Max 6 findings, max 3 actions.

## Fail condition

Compressed compile misses an actionable finding that the 31-step version catches. Not observed on any of 3 repos.

## Observations (n=3)

**vimx:** 4/6 findings actionable. No actionable misses. Dropped checks (five-layer, axiom-challenge, graveyard-expiry) were all no-ops.

**swarm:** 4/6 actionable. Same pattern. Zero predictions = structural root cause correctly surfaced as #1.

**jarvis:** 4/6 actionable. Same pattern. Edge case found: pre-flight should exclude raw/ types.

Consistent 4/6 actionable ratio across all 3 repos. The 6-finding cap forced ranking by learning impact, which is the right filter.

## Scorecard

```yaml
prediction_accuracy: exact
surprise: low
what_the_prediction_missed:
  - the dropped checks (five-layer, axiom deadlines, graveyard expiry) were ALL no-ops on every wiki tested
  - on a mature wiki (100+ pages, axioms >90 days old) the dropped checks might matter — untested
```
