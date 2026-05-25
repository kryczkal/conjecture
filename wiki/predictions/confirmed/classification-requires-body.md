---
created: 2026-05-25
last_verified: 2026-05-25
type: prediction
status: confirmed
context: [skill-design, brainstorm, vimx-migration]
tested_on: [claude-opus-4-6]
---

# Frontmatter type alone is insufficient to classify legacy wiki pages

Classifying correctly requires reading body content and asking "is this a claim about how something works?"

## Mechanism

Frontmatter type is a label the author chose, not a semantic classification. Body content reveals: principles that are actually axioms, findings that are actually frameworks, benchmarks that are actually predictions, gaps with unusual statuses (refuted, closed-by-decision) that need context to classify.

## Decisions

Migrate must read every page's body content. Classification is LLM judgment, not type mapping.

## Fail condition

Frontmatter-only mapping correct >90%. Not met on any of 3 repos.

## Observations (n=3)

**vimx:** 89.6% correct without body (7/67 changed). **swarm:** 82.9% (7/41). **jarvis:** 82.1% (14/78). Messier wikis depend more on body reading. The changed pages were disproportionately high-impact (axiom vs framework, prediction vs raw).

## Scorecard

```yaml
prediction_accuracy: exact
surprise: medium
what_the_prediction_missed:
  - vimx nearly killed the prediction (89.6% vs 90% threshold)
  - the effect is continuous, not binary — cleaner wikis need body reading less
  - the prediction framed it as essential; it's more accurate to say "essential for the 10-18% of pages where the stakes are highest"
```
