---
created: 2026-05-25
last_verified: 2026-05-25
type: knowledge
status: active
maturity: T0
tested_on: [claude-opus-4-6]
context: [migrate-testing]
---

# Migration first-pass accuracy scales inversely with wiki structural complexity

First-pass accuracy: 97% (clean/67 pages) → 95% (moderate/41 pages) → 90% (messy/78 pages). The structural features that predict lower accuracy: nested archives, binary asset references, deep relative paths, many non-claim page types (person, component, contract, status).

## Evidence

Three migrate runs on three repos with different structures:
- vimx (67 pages, 8 types, flat structure): 97%, 3 minor issues
- swarm (41 pages, 8 types, moderate structure): 95%, TODO markers + extractable predictions
- jarvis (78 pages, 11 types, nested archives + binary assets): 90%, evidence TODOs + cross-ref depth issues + broken session paths

## Mechanism (draft)

Migrate's cross-reference repair uses grep-and-replace on old paths. This works for flat moves (same depth). When pages move between directories at different nesting depths (e.g., `_archive/components/` → `graveyard/`), relative path prefixes (`../`) must be recalculated. The skill doesn't account for depth changes, causing breakage proportional to the archive nesting.

## What would prove this wrong

A structurally complex wiki (>80 pages, nested archives) that achieves >95% first-pass accuracy. This would suggest the correlation is with something else (page count, type diversity, content quality) rather than structural complexity.
