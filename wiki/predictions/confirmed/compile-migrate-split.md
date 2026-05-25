---
created: 2026-05-25
last_verified: 2026-05-25
type: prediction
status: confirmed
context: [skill-design, brainstorm]
tested_on: [claude-opus-4-6]
---

# Separating migrate from compile produces better skills than combining them

Migrate (convert non-conjecture wiki to schema) and compile (audit wiki already in schema) have different failure modes: misclassification/content loss vs. stale dates/broken links. Splitting them keeps each skill focused and prevents migration logic from diluting routine audit.

## Mechanism

Migration is a heavy, iterative, destructive operation that requires content classification, file moves, body rewrites, and cross-reference repair. Compile is a lightweight, routine, safe operation that checks schema compliance and runs the governor. Combining them would force compile to handle both "is this page in schema?" and "is this page healthy?" — two different questions with different error profiles.

## Decisions

- If true: build migrate as a separate skill. Compile assumes schema compliance.
- If false: merge migrate logic into compile. One skill does everything.

## Test plan

1. Build both skills separately. Run migrate on vimx wiki (46 legacy pages). Run compile after.
2. Measure: does compile find migration-related issues after migrate runs? If yes, the split leaks.
3. Measure: is migrate's prompt significantly different from compile's? If >70% shared logic, the split is artificial.

## Fail condition

Migrate turns out to be "compile with extra steps" — the separation creates unnecessary duplication of wiki-reading, classification, and fix-execution logic. Specifically: >70% of migrate's prompt content would work identically in compile.

## Scorecard

```yaml
prediction_accuracy: exact
surprise: low
what_the_prediction_missed:
  - migrate still needs a post-run compile pass to catch edge cases (~30% overlap in validation logic)
  - the classification engine (unique to migrate) is the dominant differentiator, not the fix-execution logic
```

**Evidence:** Ran migrate on vimx wiki (67 pages moved). Migrate's SKILL.md is 158 lines; compile's is 89 lines. ~30% overlap (frontmatter validation, broken link detection). The classification-by-body-content engine — the core of migrate — has zero overlap with compile. The split is justified.
