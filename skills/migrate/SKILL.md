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
4. **Is there actually a legacy wiki to convert?** If `wiki/` is empty, contains only the Conjecture scaffold, or has no non-conjecture content (no legacy `type:` values, no pages outside the schema), STOP: "Nothing to migrate — this is not a conversion. Use /conjecture:init to scaffold, then /conjecture:ingest to extract predictions from your source material." Migrate *routes existing typed pages*; it cannot manufacture predictions from raw conversations or source docs. Reaching for migrate on a greenfield project silently degrades into an ingest, badly.

## Complexity Assessment

Before classifying pages, scan the wiki structure and assess migration difficulty. This determines whether a single pass will suffice or iteration is needed.

### Metrics to collect

| Metric | How to measure |
|---|---|
| **Max directory depth** | Deepest nested directory relative to `wiki/` (e.g., `wiki/sessions/2026-05-15/screens/` = depth 3) |
| **Binary asset count** | Files that are not `.md` (`.png`, `.jpg`, `.log`, `.txt`, `.tmp`, etc.) |
| **Archive/session nesting** | Directories named `*archive*`, `*sessions*`, `*overnight*`, or any date-stamped subdirectory containing further subdirectories |
| **Non-standard top-level dirs** | Directories under `wiki/` that are not in the conjecture schema (`axioms`, `frameworks`, `graveyard`, `knowledge`, `predictions`, `raw`) |
| **Distinct legacy type values** | Unique `type:` frontmatter values across all pages, excluding the conjecture template line |
| **Total .md page count** | All `.md` files excluding `CLAUDE.md`, `index.md`, `log.md` |

### Scoring

Assign points:

- Max depth <= 2: **0** | 3: **1** | >= 4: **3**
- Binary assets: 0: **0** | 1-10: **1** | > 10: **3**
- Archive/session nesting: 0 nested dirs: **0** | 1-3: **1** | > 3: **2**
- Non-standard top-level dirs: 0: **0** | >= 1: **2**
- Distinct legacy types <= 8: **0** | 9-10: **1** | >= 11: **2**

Sum the points:

| Score | Classification | Guidance |
|---|---|---|
| **0-1** | **SIMPLE** | Single pass will likely achieve >95% accuracy. Run once, review the plan, execute. Iteration unnecessary unless the plan surfaces ambiguous pages. |
| **2-4** | **MODERATE** | Expect ~93-95% first-pass accuracy. Run once, execute, then re-run to catch pages that need structural rewrite after their neighbors moved. |
| **5+** | **COMPLEX** | Expect <93% first-pass accuracy. Specific concerns below apply. Plan for 2-3 iterative runs. Flag the following for the user before proceeding: |

### Complexity flags (COMPLEX wikis only)

When classification is COMPLEX, list which flags fired and what they mean:

- **Deep nesting** (depth >= 4): Cross-reference repair will need multi-level `../` rewriting. Binary assets in nested dirs may have relative paths from `.md` pages that break when pages move.
- **Binary assets** (> 10): These files do not get classified but pages that reference them need path updates. The skill cannot read binaries to verify references. Flag all pages that link to binary assets for manual spot-check after migration.
- **Non-standard directories**: Pages in directories outside the schema (e.g., `sessions/`) have no obvious target. They will land in AMBIGUOUS at higher rates, requiring user decisions.
- **Many legacy types** (>= 11): More type values means more mapping edge cases. Types not covered by the mapping rules in Phase 1 will default to AMBIGUOUS.

### Output format

Print the assessment before proceeding to Phase 1:

```
## Complexity: [SIMPLE|MODERATE|COMPLEX] (score: N)

Scanned: M pages, D max depth, B binary assets, A archive dirs, X non-standard dirs, T legacy types

[If MODERATE or COMPLEX:]
Flags:
- [flag]: [one-line explanation of impact]

Recommendation: [single-pass / re-run after first pass / plan for 2-3 iterations]
```

Then proceed to Phase 1.

## Phase 1 — Classify

Read every `.md` file under `wiki/` (excluding CLAUDE.md, index.md, log.md). For each page:

**Already migrated?** If the page is in the correct target directory AND has valid conjecture frontmatter (type, status, created, last_verified) AND has the required structural sections for its type — skip it.

**Classify by reading body content, not just frontmatter.** The legacy `type` field is a hint, not truth. Read the page and determine what it actually is.

### Type mapping rules

**findings, decisions, sources** — Usually claims with evidence → `knowledge/`. BUT: if a "finding" reads as a value commitment ("don't do X, do Y instead") rather than an observation ("we observed X"), it's a `framework/`. Read the body — the label lies. Maturity is assigned in Step 2 using the unified maturity rule below.

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

Every classification must cite the specific evidence that drove it. Format: `evidence: [what in the page body led to this classification]`. No classification without justification.

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

#### Maturity assignment (knowledge pages only)

Assign maturity by reading the page body and matching the FIRST tier whose criteria are ALL met. Check from T2 down to T0. NEVER assign T3 during migration (requires cross-context replication that a single migration cannot verify).

**T2 — tested mechanism.** ALL of the following are present in the page body:
1. A stated mechanism (an explicit WHY — not just "we chose X" but "we chose X because Y").
2. The mechanism was tested by an intervention that could have failed — at least ONE of:
   - before/after measurement (benchmark, metric, perf number)
   - shipped code verified on a real target (commit hash + real-device or emulator confirmation, e2e run, production verification)
   - user tested and approved with a specific verdict (quoted approval, explicit "approved", "confirmed working")
3. The test result is recorded in the page (not just "we shipped it" but what happened after shipping).

**T1 — draft mechanism.** ALL of the following are present in the page body:
1. A stated mechanism (an explicit WHY).
2. Evidence that supports the mechanism — at least ONE of:
   - code references (specific file paths, function names, commit hashes)
   - architectural reasoning with concrete trade-off analysis (named alternatives with specific reasons each was rejected)
   - session data, user quotes, or observations that motivated the claim
3. The mechanism has NOT been tested by an intervention (no before/after, no production verification, no user sign-off on the outcome). If it has, it is T2, not T1.

**T0 — pattern only.** Default. Assign when:
- The page states a pattern or decision but provides no mechanism (no WHY), OR
- The page has a mechanism but no supporting evidence (pure assertion), OR
- The page is a gap/observation/feature request with no causal claim

**Decision procedure for ambiguous cases:**
- "We chose X because Y" + commit hash but no post-ship verification = T1 (mechanism + code evidence, but untested).
- "We chose X because Y" + "shipped on branch Z" + "user said 'ok now its good. approved'" = T2 (mechanism tested by user verdict).
- "We chose X because Y" + "shipped on branch Z" + silence about outcome = T1 (shipped is not the same as verified; the test result must be recorded).
- Component overview pages that aggregate decisions from other pages: grade on THIS page's own evidence, not the evidence in linked pages. A summary that says "See [other-page]" without restating the evidence is T0 unless the summary itself contains mechanism + evidence.
- Pages with detailed "Alternatives considered" sections where each alternative has a specific rejection reason: T1 (the trade-off analysis IS the evidence for the mechanism, but the mechanism itself hasn't been tested by an intervention).
- Gap/observation pages ("the UI drifted", "cards render 0x0"): T0 regardless of detail level — an observation is not a mechanism.

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

After ALL moves complete, repair every reference in the wiki. This step has three sub-passes that must run in order.

#### 5a — Build the depth-aware rewrite table

For every file that moved, record: `old_wiki_path`, `new_wiki_path`, `old_depth` (directory components of old path's parent), `new_depth` (directory components of new path's parent). Example:

```
sessions/2026-05-14-overnight/journal.md → raw/2026-05-14-overnight-journal.md
  old_depth=2  new_depth=1  delta=-1
_archive/components/blackbox.md → graveyard/blackbox-component.md
  old_depth=2  new_depth=1  delta=-1
decisions/X.md → knowledge/X.md
  old_depth=1  new_depth=1  delta=0
```

For each moved file, also record its **asset root** — the directory from which the file's `screens/`, `ui-delta/`, `images/`, or other non-markdown sibling references originally resolved. For session files this is the session directory (e.g., `sessions/2026-05-14-overnight/`); for archive files it is the archive subdirectory. The asset root does NOT move with the file.

#### 5b — Rewrite references in every content page

Scan every `.md` file under `wiki/` (excluding `log.md` — handled separately in 5c). Find all references using these patterns:

1. **Markdown links:** `[text](path)` and `![alt](path)`
2. **Backtick-quoted asset paths:** `` `screens/foo.png` ``, `` `ui-delta/01.jpg` ``
3. **Bare prose asset paths:** `screens/foo.png` appearing outside backticks (less common but real)

For each reference found, apply these rules in order:

**Rule 1 — Target moved, same depth.** The link target matches an old path in the rewrite table, and source depth is unchanged. Replace the old filename/path portion with the new one. Relative prefix (`../`) stays the same.

**Rule 2 — Target moved, different depth.** The link target matches an old path, but source OR target changed depth. Recompute the full relative path:
- Count directory components from the source file's parent to wiki root = N (this many `../` segments).
- Append the new wiki-relative target path.
- Simplify. Example: from `graveyard/X.md` to `knowledge/Y.md` → `../knowledge/Y.md`. From `raw/X.md` to `sessions/2026-05-14-overnight/screens/foo.png` → `../sessions/2026-05-14-overnight/screens/foo.png`.

**Rule 3 — Source moved, asset ref unchanged.** The reference is to a non-markdown asset (`screens/`, `ui-delta/`, `images/`, `.png`, `.jpg`, `.log`) and the SOURCE file moved but the asset did NOT move. The asset ref was relative to the old location. Rewrite it to resolve from the new location:
- Compute the relative path from the source file's NEW parent directory to the asset root directory.
- Prepend that prefix to the existing ref. Example: `screens/foo.png` in a file that moved from `sessions/2026-05-14-overnight/` (where `screens/` was a sibling) to `raw/` → `../sessions/2026-05-14-overnight/screens/foo.png`.

**Rule 4 — External links (outside wiki/).** If a link uses `../../` (or deeper) to reach outside `wiki/` — e.g., `../../research/...` or `../../docs/...` — and the source file changed depth, recompute the `../` prefix. From depth N, reaching wiki's parent needs N+1 `../` segments. Adjust the prefix to match the new depth. If depth is unchanged, leave external links alone.

**Rule 5 — Sibling-relative bare-name links across directories.** Some pages link to siblings using just the filename: `(2026-05-14-visual-design-direction.md)`. If source and target were in the same directory pre-migration but are now in different directories, rewrite to the correct relative path. Example: if `2026-05-14-visual-design-v0.2-tinder-decision-pattern.md` moved from `decisions/` to `graveyard/` and links to `(2026-05-14-visual-design-direction.md)` which moved to `knowledge/`, rewrite to `(../knowledge/2026-05-14-visual-design-direction.md)`.

**Validation:** After rewriting, resolve every updated reference from its source file and confirm the target exists on disk. If a rewritten path does not resolve, flag it rather than committing a known-broken link.

#### 5c — Log.md: timestamp-gated selective repair

`log.md` is a historical record. Entries written before this migration contain old paths like `decisions/X.md`, `gaps/Y.md`, `sources/Z.md`. These are intentional historical references.

**Determine the gate date:** the date of the FIRST log entry created by this migration run. Every entry at or after that date was written with new paths in mind — rewrite those. Every entry BEFORE that date is historical — leave those untouched.

Concretely: the migration appends a `## [YYYY-MM-DD] migrate | ...` entry in Step 6. Any `##` entry whose date equals the migration date should have its paths updated. All earlier entries stay as-is, broken links and all.

**Asset refs in log.md** (e.g., `sessions/2026-05-14-overnight/screens/...`) follow the same rule: only update refs inside entries at or after the gate date.

### Step 6 — Rebuild index and log
Rewrite `index.md` to reflect new structure. Append to `log.md`:

```
## [YYYY-MM-DD] migrate | moved N pages, M ambiguous flagged, J skipped
```

### Step 7 — Loop-ignition check (after execution)

Migration installs the schema; it does not start the loop. Two post-checks:

1. **Zero-prediction warning.** If the migration produced NO pages in `predictions/` (the legacy wiki had no hypotheses/bets — a pure knowledge dump), warn loudly in the report: "This wiki is now a faithfully-typed snapshot, not a running loop. A knowledge dump migrated cleanly is still a knowledge dump. To start learning, file at least one prospective prediction (`predictions/open/` with a fail condition), or wire this project's existing automation (autonomous loop, CI hook, daily log) to file predictions." Do NOT fabricate predictions to avoid this warning — surface it honestly. Optionally, list 3-5 latent bets you noticed in the knowledge pages that the user could promote to `predictions/open/`.
2. **Axiom zero.** Ensure `axioms/self-correction.md` exists (the migrated `wiki/CLAUDE.md` references it). If missing, copy it from the bundled scaffold (`${CLAUDE_SKILL_DIR}/../../protocol/scaffold/axioms/self-correction.md`) so the governor/meta-governor have a foundation to ground out to.

## Constraints

- NEVER delete pages. Move only. Flag potential deletions for user review.
- NEVER fabricate content. Missing sections get TODO markers, not hallucinated text.
- NEVER silently overwrite. Collisions are flagged.
- Preserve refutations as first-class. Wrong predictions with explanations are product value.
- Scorecards only from actual evidence. No evidence in the body = `[needs evidence from actual test]`.
- Designed for re-running. Skip already-migrated pages. Catch pages moved but still needing structural rewrite. Classify new pages fresh.
