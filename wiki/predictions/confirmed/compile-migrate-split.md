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

Migration is a heavy, destructive operation that requires content classification, file moves, body rewrites, and cross-reference repair. Compile is a lightweight, routine, safe operation that checks schema compliance and runs the governor. The classification-by-body-content engine is the dominant differentiator — it exists only in migrate.

## Decisions

Build migrate as a separate skill. Compile assumes schema compliance.

## Fail condition

>70% shared prompt logic. Not met: 25-40% overlap across 3 repos.

## Observations (n=3)

**vimx:** ~30% overlap. **swarm:** ~35-40% overlap. **jarvis:** ~25-30% overlap. Range 25-40%, all below 70%. The overlap is in validation (frontmatter, broken links, index consistency). The classification engine, file moves, and structural rewrites have zero overlap.

## Scorecard

```yaml
prediction_accuracy: exact
surprise: low
what_the_prediction_missed: []
```
