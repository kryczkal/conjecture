---
created: 2026-05-25
last_verified: 2026-05-25
type: prediction
status: open
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

## Observations (n=3)

**vimx (2026-05-25):** 67-page wiki, 8 legacy types. ~30% overlap with compile. Direction: supports.

**swarm (2026-05-25):** 41-page wiki, 8 legacy types. ~35-40% overlap. Overlap was in frontmatter validation, broken link detection, index consistency. Classification engine, file moves, structural rewrites had zero overlap. Direction: supports.

**jarvis (2026-05-25):** 78-page wiki, 11 legacy types (including person, component, contract, status — types vimx and swarm didn't have). ~25-30% overlap. The most structurally different wiki tested. Direction: supports.

Range across 3 repos: 25-40% overlap. All well below the 70% fail threshold. The classification-by-body-content engine is consistently the dominant differentiator — it exists only in migrate.
