---
created: 2026-05-25
last_verified: 2026-05-25
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

## Observations (n=3, all pre-flight passed — fail condition untested)

**vimx:** 0 non-conjecture types. Pre-flight passed. Useful as confirmation.

**swarm:** 0 non-conjecture types. Pre-flight passed.

**jarvis:** 1 non-conjecture type (duplicate frontmatter artifact, below >3 threshold). Edge case: raw/ pages have `type: raw` which is not in the valid set — pre-flight should exclude raw/ pages.

All 3 wikis were already migrated. The fail condition (mixed-schema compile producing useful output) has not been tested. Need a partially-migrated wiki to resolve this prediction.
