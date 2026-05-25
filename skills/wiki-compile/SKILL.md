---
name: wiki-compile
description: >
  Fix the wiki. Resolve predictions with existing evidence, repair broken schema,
  add missing scorecards, move files, update index. Report only what requires
  human judgment or a new experiment.
effort: max
---

Read every page in wiki/. Fix everything you can. Report what you can't.

## 0. Pre-flight

Count pages with `type` not in valid set (prediction | knowledge | framework | axiom | graveyard). Exclude `raw/`. If >3 invalid: "Run /conjecture:migrate first." Stop.

## 1. Fix predictions with existing evidence

For each open prediction: read the body. If it contains a result (measurements, outcome, "confirmed", "refuted", "inconclusive"), the prediction is ALREADY RESOLVED — it just hasn't been moved yet.

**For each resolvable prediction, execute NOW:**
1. Determine verdict from body content (confirmed/refuted/inconclusive→graveyard)
2. Add scorecard YAML to frontmatter (`prediction_accuracy`, `surprise`, `what_the_prediction_missed`)
3. Change `status: open` → `status: confirmed` or `status: refuted`
4. Move file to `predictions/confirmed/` or `predictions/refuted/` (or `graveyard/` if inconclusive)
5. Update `index.md`

Do not ask permission. Do not defer. If the evidence is in the body, act.

## 2. Fix resolved predictions missing scorecards

For each prediction in `confirmed/` or `refuted/` that lacks a scorecard: if the body narrative contains enough to derive `prediction_accuracy` and `surprise`, add the scorecard YAML now.

## 3. Fix schema violations

Execute all of these without reporting:
- Add missing frontmatter fields (`created`, `last_verified`, `type`, `status`) where values are derivable
- Fix invalid `type` values
- Update `last_verified` on pages you just touched
- Remove dead `index.md` entries (pointing to files that don't exist)
- Add orphan pages to `index.md` (pages that exist but aren't listed)
- Fix broken evidence links (target moved → update path; target deleted → remove link)
- Add `predictions_generated` to framework pages where downstream predictions are identifiable

## 4. Fix knowledge gaps

For each framework: if no prediction has been generated from it in 30 days, add a `## Status: dormant` note.

For `predictions_generated` on frameworks: if you can identify downstream predictions from cross-references, add them now.

## 5. Compute governor

After all fixes are applied:

1. Count scorecards: exact / partial / wrong. Compute score (exact=1, partial=0.5, wrong=0).
2. Compare to previous compile entry in `log.md`. Trend: improving / flat / declining.
3. If flat or declining: **GOVERNOR ALERT**.

## 6. Report

The report is ONLY what you couldn't fix. The structure:

```
## Governor
[N resolved, M open. Score. Trend.]

## Fixed this run (count)
[file | change]

## Cannot fix without human judgment (max 5)
[prediction/page | what's needed | why you can't do it yourself]
```

"Cannot fix" means: requires a new experiment, requires human preference input, requires running code, or involves a judgment call where two valid interpretations exist.

Do NOT list "actions for the user." If you can do it, do it. If you can't, say why.

Append to `wiki/log.md`: `## [YYYY-MM-DD] lint | <headline>`.
