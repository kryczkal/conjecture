---
name: wiki-compile
description: >
  Periodic lint and learning health check for the wiki. Runs the governor
  (prediction accuracy trend), checks evidence quality, finds unfiled predictions,
  flags staleness, and produces a prioritized action plan. The governor fires first
  because learning health gates everything else.
effort: max
---

Read every page in wiki/. Then:

## Governor (learning health — always first)

1. Collect all confirmed/refuted predictions in `predictions/confirmed/` and `predictions/refuted/`. Extract scorecards: `prediction_accuracy` (exact/partial/wrong), `surprise` (low/medium/high), `what_the_prediction_missed`.
2. Check each has a scorecard. Flag missing or wrong-format scorecards.
3. Count: exact / partial / wrong. Compare to previous period if data exists. Report trend: improving / flat / declining / insufficient data.
4. If flat or declining: **GOVERNOR ALERT** — "Prediction accuracy is not improving. Consider running /challenge on axioms."
5. **Confirmable now:** which open predictions have existing test results or sufficient evidence to confirm/refute WITHOUT running new tests? These are free learning — surface them first.
6. **Project:** at the current resolution rate, when does the governor next fire (~20 confirmed/refuted)? What is the fastest path?
7. **Headline:** is this wiki learning or accumulating? One sentence with concrete numbers (open:resolved ratio).

## Knowledge→prediction pipeline

8. For each knowledge page: is it cited as evidence by any prediction? Does it contain testable claims not yet filed as predictions?
9. **Unfiled predictions:** for testable claims not yet filed, state: the claim, source page, suggested slug, fail condition. Max 5, ranked by learning value (learning gained regardless of outcome).
10. **Internal contradictions:** knowledge pages containing claims that conflict with open predictions. Flag the specific conflict and the pages involved.
11. **Framework accountability:** knowledge pages functioning as frameworks (generate predictions, self-describe as "more fundamental") — flag for reclassification. Frameworks: is `predictions_generated` complete?

## Evidence quality

12. For every page with `evidence:` entries: does the cited page's CONTENT support the specific claim? Grade each link:
    - **STRONG** — directly supports with data or argument
    - **WEAK** — tangentially related, "see also" relationship
    - **BROKEN** — file does not exist or content has no connection
13. Report totals (N STRONG / N WEAK / N BROKEN). Name only WEAK and BROKEN links specifically.
14. **Co-generated:** were the prediction and its evidence created in the same session from the same observations? Flag — shared origin is not independent confirmation.
15. **Bidirectional:** if page A cites page B as evidence, does B reference A? Flag gaps.
16. **Evidence direction:** evidence fields should point to what SUPPORTS this page, not what this page provides evidence FOR. Flag reversals.

## Test plan feasibility

17. For each open prediction, classify: **ALREADY PASSED** (test done, needs scorecard + move) / **TESTABLE NOW** / **BLOCKED** (by what) / **NEEDS INFRASTRUCTURE**.
18. **Top 3 to test next** with one-sentence rationale each.

## Systemic health

19. For each page, assess five-layer presence (action, mechanism, testing, convergence, honesty). Aggregate: which layers are systemically absent across the wiki? What does that mean for the learning loop?
20. Load-bearing pages (high citation count) with incomplete layers — fragile foundations.

## Schema + structural sweep

21. **Frontmatter** — every page has `created`, `last_verified`, `type`, `status`. LLM behavioral claims need `tested_on` + `context`.
22. **Type coherence** — content matches declared type? Type value in valid set (prediction | knowledge | framework | axiom | graveyard)?
23. **Staleness** — `last_verified` older than 30 days.
24. **Orphans** — pages not in `index.md`. Index entries pointing to deleted pages.
25. **Fail conditions** — every open prediction has an explicit fail condition?
26. **Near-duplicates** — would the same evidence confirm both? Flag pairs.
27. **Stuck predictions** — `status: open` older than 14 days.
28. **Frameworks** without downstream predictions in 30 days.
29. **Axioms** not challenged in 90 days.
30. **Graveyard** — pages past expiry. Knowledge that should move there.
31. **Maturity distribution** — knowledge pages by evidence tier (T0/T1/T2/T3).

## Fix

- Fix directly: broken links, stale dates, index drift, orphan listings, missing frontmatter fields, type values not in valid set.
- Flag only (never auto-fix): content accuracy, evidence quality grades, type reclassification, duplicate merges, missing fail conditions, unfiled predictions, internal contradictions, layer gaps.
- Never delete without explicit approval.

## Report

One-screen summary:

```
## Governor
[3 lines: counts, trend/projection, confirmable-now, headline]

## Findings (max 6, ranked by learning impact)
[each: one-line title | files affected | fix or flag | why this matters for learning]

## Action Plan (top 3 things to do TODAY)
[each: specific action, completable in one sitting, aimed at the highest-leverage learning bottleneck]
```

Every finding has a specific fix (file path, field name, old value → new value) or a specific flag (what to decide). The action plan is the primary output — actions target the learning bottleneck (prediction resolution rate), not mechanical cleanup, unless cleanup blocks learning.

Append to `wiki/log.md`: `## [YYYY-MM-DD] lint | <summary>`.
