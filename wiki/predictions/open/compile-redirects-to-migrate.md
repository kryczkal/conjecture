---
created: 2026-05-25
last_verified: 2026-05-30
type: prediction
status: open
context: [skill-design, wiki-compile, compile-migrate-split]
---

# Compile should detect unmigrated pages and redirect to migrate instead of trying to fix them

The compile-migrate-split prediction was confirmed: compile assumes wiki is in schema. But the current compile has no awareness of migrate — if it encounters legacy types or pages in wrong directories, it tries to fix them (type coherence check, step 22) instead of saying "run /conjecture:migrate."

## Mechanism

Without a redirect, compile will flag every unmigrated page as a type-coherence or frontmatter violation. This floods the findings with migration issues that compile can't fix (it doesn't have the classification engine). The user gets a wall of "fix type: finding → type: knowledge" flags when they should get one line: "run /conjecture:migrate first."

## Decisions

- If true: add a pre-flight check to compile. If >3 pages have non-conjecture types, abort the full audit and print: "This wiki has N unmigrated pages. Run /conjecture:migrate before compile."
- If false: compile should handle mixed-schema wikis gracefully, auditing what it can and flagging the rest.

## Test plan

1. Run compile on a wiki with 50% migrated / 50% legacy pages.
2. Does the output usefully distinguish migration issues from compile issues?
3. Would the user know to run migrate first?

## Fail condition

Compile on a mixed-schema wiki produces a useful, prioritized action plan that correctly distinguishes "run migrate" from "fix these specific issues." If compile handles mixed wikis well, the redirect is unnecessary.

## Observations (n=4: 3 real clean-wiki negative cases + 1 synthetic mixed positive case)

**vimx:** 0 non-conjecture types. Pre-flight passed. Useful as confirmation.

**swarm:** 0 non-conjecture types. Pre-flight passed.

**jarvis:** 1 non-conjecture type (duplicate frontmatter artifact, below >3 threshold). Edge case: raw/ pages have `type: raw` which is not in the valid set — pre-flight should exclude raw/ pages.

**synthetic mixed wiki (2026-05-30, test-hypotheses run, claude-opus-4-8):** First test of the *positive* case. Built a 50/50 fixture — 3 valid pages (prediction/prediction/knowledge) + 4 legacy pages (`decision`, `reference`, `note`, `finding`) across `docs/` and `notes/`, plus a `raw/` page typed `raw`. Ran §0 pre-flight literally. Result: counted **4 invalid > 3 → fired the redirect** (`"This wiki has 4 unmigrated pages. Run /conjecture:migrate first." + STOP`) — a single prioritized directive, NOT 4 separate "fix type:" flags. `raw/` was correctly excluded (without exclusion it would have miscounted as 5). Boundary check: removing one legacy page → 3 invalid → pre-flight does **not** fire (compile proceeds to per-page audit). Threshold is exactly `>3`, matching the "if true" decision.

**Status: still open — not resolving.** This is n=1 for the mixed-schema positive case, and it is *synthetic* (a fixture I built to fire — the weakest evidence class). The gate needs ≥3 independent observations, ideally real mid-migration wikis at varying ratios. Also note the fail condition is now partly stale: it describes the counterfactual "compile *without* the redirect handles mixed wikis gracefully," but the redirect is now built into §0, so that counterfactual can't be observed directly anymore. What the new evidence *does* establish: the redirect-as-built behaves correctly across both branches and the boundary.
