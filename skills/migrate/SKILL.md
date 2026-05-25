---
name: migrate
description: >
  Convert a non-conjecture wiki to the conjecture schema. Classifies every page
  by reading its body content, builds a plan with CLEAR/AMBIGUOUS/SKIP buckets,
  then executes: move, rewrite frontmatter, add missing structural sections,
  distill, repair cross-references, rebuild index. Destructive — requires
  version control. Designed for iterative runs until clean.
effort: max
---

This is a destructive operation. Before anything else:

1. Verify the working directory is a git repo. If not: error with "migrate requires version control — commit your wiki before running." Stop.
2. Check for uncommitted changes in wiki/. If dirty: warn "uncommitted changes in wiki/ — commit or stash before migrating." Stop.
3. Read `wiki/CLAUDE.md`, `wiki/index.md`, `wiki/log.md` (last 10 entries).

## Phase 1 — Classify

Read every `.md` file under `wiki/` (excluding CLAUDE.md, index.md, log.md). For each page:

**Already migrated?** If the page is in the correct target directory AND has valid conjecture frontmatter (type, status, created, last_verified) AND has the required structural sections for its type — skip it.

**Classify by reading body content, not just frontmatter.** The legacy `type` field is a hint, not truth. Read the page and determine what it actually is.

### Type mapping rules

**findings, decisions, sources** — Usually claims with evidence → `knowledge/`. BUT: if a "finding" reads as a value commitment ("don't do X, do Y instead") rather than an observation ("we observed X"), it's a `framework/`. Read the body — the label lies. Evidence strength determines maturity:
- STRONG evidence (data, measurements, replicated observations) — T1 or T2
- MODERATE evidence (single observation, anecdotal) — T0
- WEAK or NONE — T0, flag for evidence gathering

**hypotheses** — Status determines target:
- `open`, `parked`, untested — `predictions/open/` (parked = untested, just deprioritized; note this in body)
- `confirmed`, `partially-shipped` — `predictions/confirmed/` (for partial: note partial nature in body)
- `refuted`, `partially_refuted` — `predictions/refuted/` (partial refutation means the main claim was tested and found wanting)
- `superseded` — `graveyard/` (add trigger: "reactivate if the replacement approach fails or context changes")

**principles** — Map to `frameworks/` by default. Exception: if the principle governs how the system makes decisions (meta-methodology, epistemological rules like "benchmarks validate, principles decide"), map to `axioms/` — it's constitutional, not testable within the system.

**gaps** — Two paths:
- Fixed AND verified — `graveyard/` with trigger condition ("reactivate if fix regresses")
- Open or partially fixed — `knowledge/` (an observation about what's missing is knowledge)

**benchmarks** — `raw/` as immutable data. If the page contains interpretive conclusions not already captured elsewhere, flag: "interpretive content may need extraction to knowledge/." Exception: planned benchmarks with pre-registered predictions and retraction thresholds are `predictions/open/`, not raw — they make testable bets.

**sessions, transcripts, logs** — `raw/` unconditionally.

**Non-claims** (contacts, procedures, reference material) — `raw/` or flag for manual review if unclear.

### Evidence-anchored justification

Every classification must cite the specific evidence that drove it. Format: `evidence: [what in the page body led to this classification]`. No classification without justification. Evidence strength (STRONG/MODERATE/WEAK/NONE) determines maturity tier.

### Collision check

Before assigning a target path, check if a file with the same name already exists there. If collision detected: flag it, never silently overwrite.

## Phase 2 — Plan

Present the plan. Do not execute until the user approves (unless `--auto` flag was passed).

```
## Migration Plan (N pages)

### CLEAR — auto-migrate (M pages)
current → target | type(maturity) | evidence: [what drove this] | confidence: HIGH

### AMBIGUOUS — needs decision (L pages)
current → default target | competing: X or Y | why ambiguous: [specific reason] | confidence: LOW

### SKIP — already in schema (J pages)
path | reason

### Minor fixes — in schema but need frontmatter update (K pages)
path | fix needed

### Post-migration TODO
- [ ] Add scorecard to predictions/confirmed/X.md (evidence exists in body)
- [ ] Extract interpretive claims from raw/Y.md to knowledge/
- [ ] Review low-confidence classifications

Proceed? [y/n/adjust]
```

If `--auto`: print the plan, then execute immediately without waiting.

## Phase 3 — Execute

Process each non-skipped page through this pipeline:

### Step 1 — Move file
Move to target directory. Create directories if needed.

### Step 2 — Rewrite frontmatter
Replace legacy frontmatter with conjecture schema:

```yaml
---
created: [preserve original or use today]
last_verified: [today]
type: prediction | knowledge | framework | axiom | graveyard
status: open | confirmed | refuted        # predictions
        active | superseded | archived     # knowledge, frameworks
maturity: T0 | T1 | T2 | T3              # knowledge only
---
```

Demote claims to T0 unless the page body contains strong evidence justifying a higher tier.

### Step 3 — Structural rewrite
Add missing sections per type template. NEVER fabricate content — use TODO markers.

**Prediction:** prediction (one sentence) / mechanism (why you expect this) / test plan / fail condition. If confirmed/refuted: add `## Scorecard` — generate from body evidence if it exists, otherwise `[needs evidence from actual test]`.

**Knowledge:** evidence / mechanism / implication. Tag maturity tier.

**Framework:** commits to / blind to / predictions generated. If no downstream predictions exist, note `[no predictions generated yet — framework is untested]`.

**Axiom:** rule / why / abandon when / last challenged.

**Graveyard:** original content / trigger condition / last reviewed.

### Step 4 — Distill body
Compress verbose sections. Preserve: evidence, counter-evidence, measurements, code literals, fail conditions, rejection reasons. Strip: scaffolding, framing, narrative transitions, redundant cross-references.

### Step 5 — Cross-reference repair
After ALL moves complete: grep the entire wiki for old paths, replace with new paths.

**Exception:** Do NOT rewrite paths in `log.md` entries that predate this migration. Old log entries are historical records. Only update: `index.md`, body links in content pages, frontmatter `evidence:` fields.

**External links:** Preserve links pointing outside `wiki/` (e.g., `../../research/...`, `../../bench/...`). When a page moves between directories at the same depth, external relative links stay valid. When depth changes, rewrite the `../` prefix to maintain the correct path.

### Step 6 — Rebuild index and log
Rewrite `index.md` to reflect new structure. Append to `log.md`:

```
## [YYYY-MM-DD] migrate | moved N pages, M ambiguous flagged, J skipped
```

## Constraints

- NEVER delete pages. Move only. Flag potential deletions for user review.
- NEVER fabricate content. Missing sections get TODO markers, not hallucinated text.
- NEVER silently overwrite. Collisions are flagged.
- Preserve refutations as first-class. Wrong predictions with explanations are product value.
- Scorecards only from actual evidence. No evidence in the body = `[needs evidence from actual test]`.
- Designed for re-running. Skip already-migrated pages. Catch pages moved but still needing structural rewrite. Classify new pages fresh.
