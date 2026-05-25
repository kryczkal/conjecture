---
name: wiki-compile
description: >
  Is this wiki learning or accumulating? Governor-first audit: score resolved
  predictions, find what blocks the next resolution, check evidence, surface
  unfiled predictions, fix what you touch, report.
effort: max
---

Read every page in wiki/. Then:

## 0. Pre-flight

Count pages whose frontmatter `type` is not in the valid set (prediction | knowledge | framework | axiom | graveyard). Exclude pages in `raw/` — they use `type: raw` or legacy types, which is correct for immutable data. If >3 non-conjecture types outside raw/: print "This wiki has N unmigrated pages. Run /conjecture:migrate first." and stop.

## 1. Governor (the primary output)

The ONE question: is this wiki learning or accumulating?

1. Collect every prediction in `predictions/confirmed/` and `predictions/refuted/`. Extract scorecards: `prediction_accuracy` (exact/partial/wrong), `surprise` (low/medium/high), `what_the_prediction_missed`.
2. Flag resolved predictions missing scorecards. If the body has enough narrative to derive accuracy and surprise (e.g., body says "confirmed" with specific measurements), add the scorecard now.
3. Count: exact / partial / wrong. Trend vs. previous compile: improving / flat / declining / insufficient data.
4. If flat or declining: **GOVERNOR ALERT** — "Prediction accuracy is not improving. Consider running /challenge on axioms."
5. **Confirmable now:** open predictions with existing evidence sufficient to confirm/refute WITHOUT new tests. These are free learning.
6. **Headline:** `"N resolved, M scorecards, K open. [Learning/Accumulating]. [Trend]."`

**Fix now:** Add scorecard YAML blocks to resolved predictions where the body narrative makes accuracy and surprise derivable.

## 2. What blocks the next resolution?

For each open prediction, answer ONE question: what prevents confirmation or refutation right now?

- **EVIDENCE EXISTS** — test data is in the wiki; needs scorecard + move
- **TESTABLE NOW** — no infrastructure needed, just run the test
- **BLOCKED BY ___** — name the specific blocker
- **PARKED** — explicitly deferred; state whether park condition changed

Rank by resolution speed. **Top 3 to test next** with one-sentence rationale each.

**Fix now:** Move EVIDENCE EXISTS predictions to confirmed/refuted with scorecards. Update index.

## 3. Knowledge→prediction pipeline

For each knowledge page: does it contain testable claims not yet filed as predictions?

**Unfiled predictions:** state the claim, source page, suggested slug, fail condition. Max 5, ranked by learning value.

**Internal contradictions:** knowledge claims that conflict with open predictions. Name both pages and the specific conflict.

**Framework accountability:** frameworks missing `predictions_generated` — add the field with known downstream predictions. Frameworks that haven't generated a prediction in 30 days — flag as dead weight.

**Fix now:** Add `predictions_generated` to framework pages where downstream predictions are identifiable from wiki cross-references.

**Flag only:** Unfiled predictions, contradictions, reclassifications.

## 4. Evidence audit (governor-directed)

For pages the governor flagged (resolved predictions, confirmable-now candidates, EVIDENCE EXISTS predictions), check evidence quality:

1. Does cited evidence support the claim? Grade: **STRONG** / **WEAK** / **BROKEN**.
2. **Co-generated?** Prediction and evidence from the same session? Flag.
3. **Broken links?** Fix or remove.

Report totals. Name WEAK and BROKEN specifically.

**Fix now:** Remove broken evidence links. If the target file moved (e.g., `sessions/X.md` → `raw/X.md`), update the link.

**Flag only:** WEAK evidence grades, co-generated evidence, evidence direction reversals.

## 5. Schema essentials

Check all pages for:
- **Frontmatter:** `created`, `last_verified`, `type`, `status` present. `type` in valid set. LLM behavioral claims have `tested_on` + `context`.
- **Staleness:** `last_verified` older than 30 days.
- **Orphans:** pages not in `index.md`; index entries pointing to deleted files.
- **Stuck predictions:** `status: open` older than 14 days.
- **Fail conditions:** every open prediction has an explicit fail condition.
- **Maturity distribution:** knowledge pages by tier (T0/T1/T2/T3).

**Fix now:** Add missing frontmatter fields where values are unambiguous. Fix invalid type values. Update stale `last_verified` on pages just verified. Remove dead index entries. Add orphans to index.

**Flag only:** Type reclassifications, missing fail conditions, stuck predictions.

## 6. Report

```
## Governor
[Headline]
[Confirmable now — if any]

## Blockers (ranked by resolution speed)
[slug | category | the ONE thing]

## Fixed (count)
[file | what changed | which section caught it]

## Findings (max 6, ranked by learning impact)
[one-line title | files | fix or flag | why it matters]

## Actions (max 3, completable today)
[specific action targeting the learning bottleneck]
```

Append to `wiki/log.md`: `## [YYYY-MM-DD] lint | <governor headline>`.
