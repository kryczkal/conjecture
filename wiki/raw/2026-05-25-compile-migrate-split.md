# Spec: Compile + Migrate Split

## Goal

Make conjecture's wiki engine do the work it identifies. Split into two skills: `migrate` converts non-conjecture wikis to the conjecture schema (heavy, iterative, destructive); `compile` audits and fixes wikis already in schema (routine, safe, fast). Out of scope: scale optimization for 200+ page wikis (that's `/evolve` territory), and the `--auto` flag for migrate (add after the core works).

## Architecture

```
User installs conjecture into project with existing wiki
    │
    ▼
/conjecture:init          ← scaffold directories + protocol (exists, unchanged)
    │
    ▼
/conjecture:migrate       ← NEW: classify, move, rewrite legacy pages
    │  (iterative — run until clean)
    ▼
/conjecture:compile       ← CHANGED: assumes schema. audit + fix + governor
    │  (routine — run periodically)
    ▼
/conjecture:predict       ← unchanged
/conjecture:ingest        ← unchanged
/conjecture:distill       ← unchanged
```

Data flow for `migrate`:

```
read all pages → classify each (claim? data? reference?) → build plan
    │
    ▼
show plan to user → user approves/adjusts → execute full sweep
    │
    per page: move file → fix frontmatter → structural rewrite → distill body
    │
    ▼
fix cross-references (all internal links updated) → update index → log
```

## Changes

| File | Change |
|------|--------|
| `skills/migrate/SKILL.md` | NEW — full migrate skill prompt |
| `skills/wiki-compile/SKILL.md` | CHANGE — remove migration concerns, add execute-on-findings, keep governor |
| `skills/init/SKILL.md` | CHANGE — mention migrate as next step when existing wiki detected |
| `protocol/CLAUDE.md` | CHANGE — add Migrate to Operations section |
| `CLAUDE.md` | CHANGE — add migrate to skill list in Structure section |
| `.claude-plugin/plugin.json` | CHANGE — register migrate skill |

## User-confirmed decisions

| # | Decision | Choice | Source |
|---|----------|--------|--------|
| 1 | Where executor lives | Split: migrate (new) + compile (existing). Migrate handles conversion; compile assumes schema. | Error scenario A revealed migration has different failure modes than routine audit. |
| 2 | Confirmation model | Show plan → user approves → execute. `--auto` flag skips confirmation. | User preference — trust after first run. |
| 3 | Migration depth | Move files → structural rewrite (add missing sections) → distill body. Full pipeline. | User confirmed: distill needs structurally proper page first. |
| 4 | Sweep mode | Full sweep per run. Accept imperfection. Flag low-confidence. Run iteratively until clean. | User: "iterations iterations iterations." |
| 5 | Scorecards | Generate only from actual evidence. No evidence → flag, don't fill. Never hallucinate. | User: "hallucinating stuff on legacy knowledge is what conjecture was supposed to prevent." |
| 6 | Content classification | Read body content, not just frontmatter type. Ask: "Is this a claim about how something works?" Claims → conjecture type. Raw data → raw/. Reference material → outside schema. | Provocative assumption 1 was wrong — many pages aren't conjectures at all. |
| 7 | Maturity on migration | Demote claims to T0 unless page has strong evidence. Non-claims don't get maturity tiers. | Provocative assumption 2: "claims should be demoted." |
| 8 | Destructive warning | Migrate must warn that it's potentially destructive. Must be under version control. Error if not a git repo. | User: "explicitly say to any users that running this on an existing wiki is a potentially destructive operation." |

## Technical decisions

### 1. Classification algorithm

**Option A — Keyword heuristic:** scan body for prediction-like language ("I expect," "hypothesis," "if X then Y") vs. data-like content (tables, measurements, URLs, contact info). Fast but brittle.

**Option B — Structural analysis:** check what sections the page has (test plan → prediction, evidence → knowledge, abandon_when → axiom). More robust, leverages conjecture's own templates.

**Option C — LLM classification with the skill prompt itself:** the skill prompt instructs Claude to classify. Claude reads the page and decides. Most accurate, no separate algorithm needed.

**Pick: C.** The migrate skill IS an LLM prompt. Claude reads each page and classifies it. No separate algorithm to maintain. The skill prompt provides classification criteria (claim vs. data vs. reference) and Claude applies judgment. This is what skills are for.

### 2. Cross-reference repair

When files move (e.g., `findings/X.md` → `knowledge/X.md`), all internal links must update. Approach: after all moves, grep the entire wiki for old paths and replace with new paths. This catches frontmatter `evidence:` fields, body prose links, and index entries. Run as a single pass after all moves complete.

### 3. Migration plan format

The plan shown to the user before execution. Format:

```
## Migration Plan (N pages)

### Move + reclassify (M pages)
- findings/X.md → knowledge/X.md (claim, demoted to T0, weak evidence)
- hypotheses/Y.md → predictions/open/Y.md (has fail condition)
- benchmarks/Z.md → raw/Z.md (raw data, not a claim)

### Structural rewrite (K pages)
- knowledge/X.md: add fail_condition, mechanism sections
- predictions/open/Y.md: add scorecard (evidence exists: accuracy=partial, surprise noted in body)

### Skip (J pages)
- contacts.md: not a claim, not raw data. Leaving as-is.

### Low confidence (L pages)
- principles/P.md: could be framework OR axiom. Defaulting to framework. Review.

Proceed? [y/n/adjust]
```

## Tests

1. **Fresh wiki (no legacy):** migrate reports "nothing to migrate" and exits cleanly.
2. **Single legacy finding with evidence:** migrated to knowledge/, frontmatter updated, maturity set from evidence, body distilled, cross-refs updated.
3. **Single legacy finding WITHOUT evidence:** migrated to knowledge/ at T0, no scorecard generated, flagged for evidence.
4. **Non-claim page (contacts, procedures):** left in place or moved to raw/, not forced into conjecture schema.
5. **Legacy hypothesis with fail condition:** migrated to predictions/open/, structural sections preserved.
6. **Legacy hypothesis confirmed but no scorecard:** migrated to predictions/confirmed/, scorecard generated only if evidence in body.
7. **Cross-reference repair:** after moving A.md, all pages linking to A at old path now link to A at new path.
8. **No git repo:** migrate errors with "requires version control" before doing anything.
9. **Second run after partial migration:** migrate picks up remaining unmigrated pages, skips already-migrated ones.
10. **Compile after migrate:** compile runs clean on a fully-migrated wiki with no legacy-schema findings.
