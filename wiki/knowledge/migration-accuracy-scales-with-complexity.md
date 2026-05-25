---
created: 2026-05-25
last_verified: 2026-05-25
type: knowledge
status: active
maturity: T1
tested_on: [claude-opus-4-6]
context: [migrate-testing]
evidence: [predictions/confirmed/iterative-migration.md, predictions/confirmed/demote-claims-on-migration.md]
---

# Migration first-pass accuracy scales inversely with wiki structural complexity

First-pass accuracy: 97% (clean/67 pages) → 95% (moderate/41 pages) → 90% (messy/78 pages). The structural features that predict lower accuracy: nested archives, binary asset references, deep relative paths, many non-claim page types (person, component, contract, status).

## Evidence

Three migrate runs on three repos with different structures:
- vimx (67 pages, 8 types, flat structure): 97%, 3 minor issues
- swarm (41 pages, 8 types, moderate structure): 95%, TODO markers + extractable predictions
- jarvis (78 pages, 11 types, nested archives + binary assets): 90%, evidence TODOs + cross-ref depth issues + broken session paths
- vimx re-run with evolved skill (67 pages, evolved maturity + cross-ref): 97%, 26 depth-aware cross-refs caught, 26/28 pages correctly tiered (10 T2, 16 T1, 2 T0)

## Mechanism (tested)

The primary driver is cross-reference depth changes during file moves. When pages move between directories at different nesting depths (e.g., `_archive/components/` → `graveyard/`), relative path prefixes (`../`) must be recalculated. Simple grep-and-replace misses these.

**Intervention (2026-05-25):** Evolved migrate skill with three-sub-pass depth-aware cross-ref repair (rewrite table → depth-adjusted rewrites → log.md timestamp gating). On vimx: caught 26 cross-refs the old approach missed. On jarvis dry-run: jumped from 33% to 93% cross-ref accuracy (66 additional fixes).

Secondary drivers: evidence-adaptive maturity (blanket T0 → differentiated T0/T1/T2) and complexity assessment (SIMPLE/MODERATE/COMPLEX scoring that correctly predicted all three accuracy bands).

## What would prove this wrong

A structurally complex wiki (>80 pages, nested archives) that achieves >95% first-pass accuracy WITH the old grep-and-replace approach. That would mean the depth-change mechanism is not the driver. The evolved skill's improvement already suggests the mechanism is correct — fixing it improved cross-ref accuracy from 33% to 93% on the hardest case.
