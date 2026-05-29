---
name: ingest
description: >
  Enrich the wiki with external knowledge. Three-layer processing
  (implicit assumptions → tensions → explicit claims), surprise-ranked,
  targeted wiki changes. Input is raw data, not truth. Updates existing
  pages over creating new ones.
argument-hint: [file path, URL, or describe observation]
effort: max
---

# Wiki Ingestion — Multi-Layer Enrichment

Enrich the wiki with external knowledge. NOT an importer — an enricher. Every input is raw data to be questioned, compared against what the wiki already knows, and used to push the learning engine forward.

The highest-value ingestion finds what the input ASSUMES but doesn't state, where it DISAGREES with the wiki, and what it PREDICTS that hasn't been tested.

---

## Phase 1: Read Wiki State

Read:
1. `wiki/CLAUDE.md` — schema, page types, maturity ladder, operations
2. `wiki/index.md` — what the system currently believes
3. `wiki/log.md` — last 5 entries (recent activity, what's been ingested)
4. 3-8 linked pages relevant to the input topic

Build a mental model of: what's known, what's being tested, what maturity level each claim has, what the learning engine needs next (open predictions closest to testable, knowledge gaps, stale pages).

---

## Phase 2: Read Input

Read the input. Accept any of:
- File path to raw data (transcript, article, notes, skill prompt)
- Pasted content in chat
- Natural language observation ("I noticed X across 3 sessions")

If the input is a file not already in `wiki/raw/`, copy it there — raw data is immutable.

---

## Phase 3: Multi-Layer Extraction

Process the input at three layers, in order. Each finds different things.

### Layer 1: Implicit Assumptions

Before analyzing explicit claims, ask:
- What does this input TAKE FOR GRANTED? What structural choices does it assume without justifying?
- What does it NOT mention that an alternative approach would?
- What would someone DISAGREEING with this input's worldview point out as unexamined?
- What does the input's STRUCTURE (not content) reveal about its assumptions?

Extract 3-5 implicit assumptions. These are often higher-leverage than explicit claims because they're unexamined — they represent the input author's blind spots.

### Layer 2: Tensions

Look for where the input DISAGREES with, COMPLICATES, or CHALLENGES the wiki. Broader than pure contradiction:
- **Contradictions** — input says X, wiki says not-X
- **Complications** — input adds nuance that existing wiki position doesn't account for
- **Connections** — input makes visible a tension between two wiki pages that weren't previously linked
- **Gaps** — input implies something the wiki hasn't considered

Check tensions between BOTH explicit claims AND implicit assumptions (from Layer 1) and the wiki. The most valuable tensions connect previously unlinked wiki pages.

### Layer 3: Explicit Claims (lowest priority)

Map remaining explicit claims to wiki pages:
- Evidence for/against open predictions → update prediction's evidence
- Data points for existing knowledge pages → enrich with new observation
- Extensions to existing knowledge from a new context → note new context, consider maturity bump
- Confirmations of what's already known → SKIP (confirmation is cheap)

---

## Phase 4: Rank by Surprise

Score every candidate change (from all three layers):
- **0 — Confirms known**: Already in the wiki, same context → SKIP
- **1 — Same direction**: Adds a data point without new insight → LOW priority
- **2 — New context or connects**: Extends to new context, or links previously unconnected pages → FILE
- **3 — Contradicts or reveals gap**: Contradicts wiki content, or surfaces an unexamined assumption → FILE FIRST

File ONLY changes with surprise ≥ 2, in descending order.

If nothing scores ≥ 2, report that the input doesn't enrich the wiki and explain why. An ingestion that produces zero changes is a valid outcome — it means the wiki already absorbed the input's signal.

---

## Phase 5: Execute Wiki Changes

For each change (descending surprise), determine the right action:

| Action | When |
|--------|------|
| **Resolve an open prediction** (highest priority) | The input bears on an OPEN prediction's fail condition. If the input satisfies the fail condition → move the page to `predictions/refuted/`, set `status: refuted`, add the scorecard YAML (`prediction_accuracy`, `surprise`, `what_the_prediction_missed`). If it decisively confirms → move to `predictions/confirmed/` with a scorecard. This is the loop closing — do it before anything else the input enables. |
| **Update existing page** (preferred for non-resolving evidence) | Input enriches something already there — add evidence, note contradiction, connect to other pages, bump maturity if warranted |
| **New prediction** | Genuinely testable disagreement — must have mechanism, test plan, fail condition per wiki schema |
| **New knowledge** | Genuine pattern with n≥3 independent observations — not opinions rephrased as patterns |
| **Complementary pair** | New prediction that tests an unexamined axis of an existing prediction (e.g., depth vs. accuracy) |

**Never soften a contradiction into a caveat.** If the input disconfirms a prediction or a confirmed claim, resolve/reopen it as a status change — do NOT append "but…" to a page while leaving its `status: confirmed`. Disconfirming evidence that only lands as a caveat is the failure mode this product exists to prevent (a wrong belief keeps its authority).

Maximum 5 wiki changes per ingestion. Every change must satisfy the action layer: if removing it would change no decision, don't file it.

For each change, state:
- What it is (update vs. new page)
- The surprise score and why
- Why it earns its keep (what decision it changes or what tension it creates)

---

## Phase 6: Log and Report

1. Append to `wiki/log.md`:
   ```
   ## [YYYY-MM-DD] ingest | <source> — <summary>

   <what was ingested, what changes were made, what was skipped and why>
   ```
2. Update `wiki/index.md` if new pages were created
3. Report to user: what changed, what was skipped, what the wiki needs next

---

## Rules

1. **Input is raw data, not truth.** Every claim is questioned. Opinions without evidence are noted and skipped. The input author's assertions are not independent observations.
2. **Enrichment over import.** Updating existing pages is almost always better than creating new ones. The wiki doesn't need more pages — it needs richer pages.
3. **Implicit → tension → explicit.** Process in this order. The highest-leverage findings come from what the input assumes, not what it states.
4. **Surprise over confirmation.** Contradictions and gaps advance the learning engine. Confirmations don't. A single contradiction is worth more than ten confirmations.
5. **Every change must change a decision.** If removing the change would alter no behavior, don't make it.
6. **Maintain maturity ladder.** Don't promote claims beyond their evidence level. T0 requires n≥3 independent observations. T1 requires a proposed mechanism. T2 requires a tested mechanism. Counting the input author's repeated assertions is not n≥3.
7. **Name what you skipped.** The list of things NOT filed is as important as what was filed. It shows discrimination and prevents future re-processing of the same input.
8. **Connect, don't duplicate.** When the input touches multiple wiki pages, trace the connections. Linking previously unrelated pages is a high-value operation.
