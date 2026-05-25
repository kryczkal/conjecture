---
created: 2026-05-25
last_verified: 2026-05-25
type: prediction
status: open
context: [skill-design, brainstorm, evidence-maturity]
tested_on: [claude-opus-4-6]
---

# Legacy claims should be demoted to T0 on migration

When migrating a pre-conjecture wiki, claims about how something works should be demoted to T0 (pattern, no mechanism required) unless they have strong evidence already embedded in the page. The maturity tier is not a trust score for the author — it's a measure of how rigorously the claim has been tested through the conjecture loop.

## Mechanism

Pre-conjecture claims weren't subjected to the predict→observe→gap→update cycle. They may be correct, but their evidence hasn't been evaluated against conjecture's maturity ladder (T0: n>=3 observations, T1: proposed mechanism, T2: tested mechanism, T3: cross-context). Preserving an inherited T2 rating for a claim that was never explicitly tested at T2 corrupts the maturity signal.

## Decisions

- If true: migrate sets all claim pages to T0 unless evidence in the page clearly meets a higher tier. The user manually promotes after review.
- If false: migrate preserves the original evidence level. Demotion creates unnecessary churn — the user re-promotes most pages to their original tier anyway.

## Test plan

1. Migrate vimx wiki with demotion (all claims → T0).
2. User reviews each demoted page. Records: would I promote this back? To what tier?
3. Count: what percentage of demoted pages get re-promoted to their original tier?

## Fail condition

>70% of demoted pages get re-promoted to their original tier or higher after user review, meaning demotion was unnecessary churn that wasted the user's time.

## Observations (n=1, not yet confirmable)

**vimx (2026-05-25):** 8 pages demoted to T0. 0% re-promoted on review. Closest call: auto-rescan-after-mutation (17 sessions, proposed mechanism, borderline T1). Direction: supports prediction.

Needs: iris run + a wiki where the original author was more rigorous about evidence (vimx's pre-conjecture wiki was written quickly — a more methodical wiki might have pages that legitimately earned higher tiers). Also needs actual USER review, not just LLM review — the test agent judged demotions, not the project owner.
