---
created: 2026-05-25
last_verified: 2026-05-25
type: prediction
status: open
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

## Observations (n=3, mixed results)

**vimx (2026-05-25):** 97% first-pass accuracy. 3 minor issues. Direction: against prediction (above 95% threshold).

**swarm (2026-05-25):** ~95% first-pass accuracy. Remaining: TODO markers in structural sections, extractable predictions from raw session data, inline prose references to old dir names, verbose pages needing distill. Direction: borderline.

**jarvis (2026-05-25):** ~90% first-pass accuracy. Remaining: evidence field TODOs on 20 knowledge pages, 8 pages arguably deserving T1 not T0, cross-ref depth issues from nested archives, broken session screen paths after move. Direction: supports prediction.

Pattern: accuracy correlates with wiki messiness. Clean wikis (vimx) → first pass is enough. Messy wikis (jarvis) → iteration helps. The predicted mechanism (interdependencies causing cascading issues) was wrong for all three — the actual driver is structural complexity (archive nesting, binary asset references, deep relative paths).
