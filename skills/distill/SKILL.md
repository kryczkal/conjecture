---
name: distill
description: >
  Distill a wiki page to minimum-viable information content using context-aware
  templates (4-part base, 5-part for decisions with evidence, 5-part for open
  hypotheses), gated by extraction-based coverage to prevent information loss.
  Three-pass: density-check → generate+evaluate → coverage-gate.
  Use when a wiki page has grown verbose, before adding a page to a context window,
  or after writing a first draft to strip scaffolding. Invoke with /distill [path].
---

Distill the target wiki page in three passes: (0) density pre-check, (1) apply distillation template + self-evaluate, (2) extraction-based coverage gate. Only replace if the distilled version preserves load-bearing content.

Read the file if a path is given.

---

## Pass 0 — Density Pre-Check

Estimate the page's scaffolding ratio before distilling:

- **Scaffolding:** framing, narrative transitions, introductory/concluding paragraphs, "see also", cross-references that duplicate frontmatter links, historical background, verbose procedure logs.
- **Load-bearing:** decisions, mechanisms, thresholds, code literals, test steps, failure conditions, alternatives with rejection reasons, evidence conclusions.

scaffolding_ratio = scaffolding_words / total_words

**If scaffolding_ratio < 0.20:** the page is already ≥80% load-bearing. Output the original unchanged: "Page is already ≥80% load-bearing content. Distillation skipped." Stop here.

**If scaffolding_ratio ≥ 0.20:** proceed to Pass 1.

---

## Pass 1 — Apply Distillation Template

Read frontmatter `type` and `status` first. Select the template:

### For type `decision`, or status `confirmed`

Rewrite the body as 5 parts (no prose between sections):

1. **Problem** — ONE sentence. Concrete failure without this knowledge. Name the failure, not the category.
2. **Fix** — ONE sentence. What changes in developer behavior. Mechanism visible (not outcome).
3. **Decisions** — bullet list. Each: `do X [because mechanism, ≤12 words]`. Mechanism = causal chain, not outcome. Order: highest consequence first. Preserve code literals, flag names, API names, exact thresholds verbatim.
4. **Failure modes** — one per line: `(Applies when [condition].)`. Specific conditions. Embed exact measurements inline when relevant.
5. **Evidence** — ONE line. Format: `Verified: [method] → [key measurement(s)], [conclusion].` Only include if the page has empirical verification or production data. Strip verbose procedure steps but keep the measurements and conclusion.

### For type `finding`, `principle`, `benchmark`, or `gap`

Rewrite the body as 4 parts (no headers, no prose between sections):

1. **Problem** — ONE sentence. Concrete failure without this knowledge.
2. **Fix** — ONE sentence. Mechanism visible.
3. **Decisions** — same format as above.
4. **Failure modes** — same format as above.

### For type `hypothesis` or `prediction` AND status `open`

Rewrite the body as 5 parts (no headers, no prose between sections):

1. **Claim** — ONE sentence. Core prediction.
2. **Mechanism** — ONE sentence. Causal chain — why it would be true.
3. **Decisions** — same format as above.
4. **Test** — copy test plan VERBATIM. Do not compress or paraphrase. Most load-bearing content.
5. **Failure modes** — same format as above.

**Strip:** scaffolding, framing, narrative context, cross-references, "see also" links, historical background, introductory sentences, transitional phrases, verbose verification procedures, commit hashes (unless referenced in a failure mode or mechanism).

**Keep verbatim:** code literals, flag names, exact thresholds, measurements, error strings, function names, API names, env var names and values, file paths.

---

## Pass 1b — Self-Evaluation

Read the Pass 1 output. Evaluate each part against these binary questions:

**Problem sentence:**
- Exactly one sentence?
- Names a concrete failure (not a category, not "causes problems")?

**Fix sentence:**
- Exactly one sentence?
- Mechanism visible — names what changes in behavior, not the outcome?

**Decisions:**
- Each bullet starts with lowercase imperative verb (`do`, `use`, `add`, `avoid`...)?
- "because" clause is causal mechanism, not outcome?
- No compound sentences (semicolons joining multiple thoughts = fail)?
- Code literals and exact values preserved verbatim?

**Failure modes:**
- Each is `(Applies when [specific condition].)`?
- Conditions are specific, not vague?

**Open hypotheses — Test section:**
- Verbatim copy present, untouched?

**Evidence line (decision pages):**
- Exactly one line?
- Contains the key measurement(s)?
- States the conclusion?

For any part that fails a check: rewrite that part only. Preserve passing parts.

---

## Pass 2 — Coverage Gate

Compare the distilled output against the original. Only replace if the distilled version is better.

### Step 1 — Extract load-bearing items from the ORIGINAL

List every item, classified by tier:

**Tier 1 — must preserve (100% required):**
- Decisions (the actual "do X" actions)
- Mechanisms (causal chains — WHY each decision works)
- Code literals used in implementation (function names, env vars, flag names, CLI flags)
- Exact thresholds and measurements that gate behavior

**Tier 2 — should preserve (≥90% required):**
- Alternatives considered + rejection reasons (the mechanism, not just "rejected")
- Failure modes and their specific conditions
- Trade-offs explicitly accepted

**Tier 3 — may strip (no minimum):**
- Verification procedures and verbose output logs
- Commit hashes and implementation refs
- Cross-references and "related" links
- Historical narrative and re-attributions
- Section headers, framing, transitions

### Step 2 — Check coverage

For each Tier 1 and Tier 2 item, verify it appears in the distilled output (same information, possibly reworded). Mark PRESERVED or LOST.

### Step 3 — Compute and gate

- tier1_coverage = tier1_preserved / tier1_total
- tier2_coverage = tier2_preserved / tier2_total
- token_reduction = 1 - (distilled_tokens / original_tokens)

**REPLACE if:** tier1_coverage = 1.00 AND tier2_coverage ≥ 0.90 AND token_reduction ≥ 0.20

**DO NOT REPLACE if:** any Tier 1 item LOST, OR tier2_coverage < 0.90, OR token_reduction < 0.20

If not replacing: output the ORIGINAL unchanged. Report which items were lost and why distillation was rejected.

---

## Output

Print the final distilled version (if gate passed) or the original (if gate rejected or fast-exit). No meta-commentary about what was stripped.

## Batch mode (`all`, a directory, or multiple pages)

The three passes above describe a SINGLE page. When asked to distill many pages (`/distill all`, a directory, or a list), do not improvise — follow this:

1. **Resolve the target list first.** Enumerate the exact page paths to be distilled and print them. Exclude `CLAUDE.md`, `index.md`, `log.md`, and anything under `raw/` (raw is immutable).
2. **One sub-agent per page.** Fan out, each sub-agent running the full Pass 0 → 1 → 2 pipeline on **exactly one** target file. The pipeline is identical to single-page mode — do not re-derive it in the sub-agent prompt; reference this skill.
3. **Hard scope per sub-agent.** Each sub-agent may read and write **only its one target page**. It must NOT touch `log.md`, `index.md`, sibling pages, or any other file. (The observed failure: a batch sub-agent edited `log.md` and was caught only by the human orchestrator. Forbid it.)
4. **The gate is per-page and binding.** A page is replaced only if its own coverage gate passes (Tier-1 = 1.00, Tier-2 ≥ 0.90, reduction ≥ 0.20). A failing page is left unchanged.
5. **Orchestrator aggregates, does not re-distill.** After sub-agents return, collect one summary: per page → replaced / skipped + reason + token reduction. Any `index.md`/`log.md` update is the orchestrator's single explicit final step, never a sub-agent's.

## Flags

- `--deep` — in Pass 1, replace Decisions bullets with `do X because [full mechanism, no word limit]`. Use when about to implement, not for quick reference.
- `--replace` — overwrite the source file with the distilled output, but ONLY if the coverage gate passes. If the gate rejects, the file is not modified. Reports gate results either way.
