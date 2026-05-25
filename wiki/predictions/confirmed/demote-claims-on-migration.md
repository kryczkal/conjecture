---
created: 2026-05-25
last_verified: 2026-05-25
type: prediction
status: confirmed
context: [skill-design, brainstorm, evidence-maturity]
tested_on: [claude-opus-4-6]
---

# Legacy claims should be demoted to T0 on migration

Demote to T0 unless the page body contains strong evidence meeting a higher tier. The maturity tier is a measure of how rigorously the claim has been tested, not a trust score.

## Mechanism

Pre-conjecture claims weren't subjected to the predict→observe→gap→update cycle. Preserving an inherited T2 rating for a claim that was never explicitly tested at T2 corrupts the maturity signal.

## Decisions

Migrate sets all claim pages to T0 unless evidence clearly meets a higher tier. But: the blanket rule is too aggressive for shipped-code wikis where component pages have strong implementation evidence (30+ commits, production measurements). Consider a refinement: "demote to T0 unless page contains verified implementation evidence."

## Fail condition

>70% re-promoted after user review. Not met: max was 23.5% (jarvis).

## Observations (n=3)

**vimx:** 8 demoted, 0% re-promoted. **swarm:** 11 demoted, 0% (2 borderline). **jarvis:** 34 demoted, 23.5% arguably over-demoted (component/contract pages with shipping evidence).

Still needs actual USER review, not just LLM review. The 23.5% on jarvis is LLM assessment.

## Scorecard

```yaml
prediction_accuracy: partial
surprise: medium
what_the_prediction_missed:
  - jarvis showed blanket T0 is too aggressive for shipped-code wikis (23.5% over-demoted)
  - the rule works for decision/finding-heavy wikis but not component/contract-heavy ones
  - needs a refinement distinguishing "no evidence" from "implementation evidence that doesn't match the conjecture test format"
```
