---
name: evolve
description: >
  Genetically improve any skill via parallel agent testing. Classifies the target skill's
  archetype, identifies failure modes, then runs a progressive search (screening → focused
  → breeding) with three-layer blind-spot detection: cross-evaluation, adversarial aggregation,
  and forced lenses. The fitness function IS the skill — different criteria produce fundamentally
  different winners. Invoke with /evolve <skill-name> [iterations].
argument-hint: <skill-name> [iterations=3]
effort: max
---

# Evolve — Genetic Skill Refinement

Improve any skill by testing prompt variants against real inputs and selecting for quality. Uses three-layer blind-spot detection: cross-evaluation (gene-level biases), adversarial aggregation (category-level omissions), and forced lenses (framework-level diversity). Produced `/distill` (27 agents), `/bug-hunter` (54 agents), `/architect` (24 agents), `/brainstorm` (40 agents), and `/evolve` itself (3 iterations, 20 agents).

---

## Process

### Step 0: Read the target skill

Read the target skill's SKILL.md. Understand:
- What it does (the task)
- What "good output" looks like (the goal)
- What its phases are (if multi-phase, each phase can be evolved independently)
- What the current prompt structure is (the baseline gene)

### Step 1: Classify the skill archetype

**This determines everything downstream.** Classify the target into one of 4 archetypes:

| Archetype | Examples | Eval strategy | Output constraint |
|-----------|---------|---------------|-------------------|
| **DISCOVERY** | bug-hunter, architect, review | 1-pass. Self-eval counterproductive. | "Exactly N findings" or "the ONE thing" |
| **GENERATION** | distill, tight-prose | 2-pass (generate→evaluate). | Template conformance + binary rubric |
| **INTERACTIVE** | brainstorm | Per-phase evolution. Self-play. | Question quality + output quality |
| **PROCESS** | evolve | Test planning quality. | Meta-properties (efficiency, adaptability) |

The archetype drives: evaluation strategy (2-pass vs 1-pass), criteria dimensions, gene design axes, and how to interpret self-scores.

### Step 2: Identify failure modes

**The lens grinder.** Before designing criteria, identify the top 3 failure modes for this specific skill:

1. **Uselessness** — What makes the output completely useless? Concrete scenario.
2. **Harmful** — What makes it worse than not using the skill at all?
3. **Subtle** — What passes casual inspection but is fundamentally wrong?

Each must be a specific scenario with an example, not a generic category. This step calibrates what "bad" looks like, which sharpens the criteria's ability to distinguish good from looks-good-but-isn't.

Why this matters: failure calibration improved criteria precision by ~1 point (3.0/3 vs 2.0/3 avg). Meta-tested: 4 agents, failure-calibrated plans scored 12/15 vs 10.5/15 without.

### Step 3: Define evaluation criteria

Design 4-5 scoring dimensions (0-3 each, total /15) informed by BOTH archetype and failure modes.

For each criterion, label whether it:
- Measures an **archetype quality signal** (what good looks like), OR
- **Guards a failure mode** (what bad looks like — specify which one)

**Rules:**
- Dimensions must be INDEPENDENT (scoring high on one doesn't guarantee high on another)
- At least one must be a **false-positive detector** — catches output that LOOKS good but IS wrong
- At least one must measure what the skill should NOT do (restraint, signal/noise, padding avoidance)
- If a criterion could apply unchanged to a different skill type, it is too generic

### Step 4: Choose test inputs

Pick 2-3 diverse, real inputs from the user's repos. They must:
- Vary in complexity
- **Trigger at least one identified failure mode each**
- Include one edge case that would fool a naive approach

For INTERACTIVE skills, the agent plays BOTH roles (self-play). For PROCESS skills, test planning quality.

### Step 5: Design initial gene variants

Create 3 genuinely different prompt variants — different structural approaches, not tweaks. Use archetype-specific design axes:
- DISCOVERY: nose-first vs. systematic vs. trace-based
- GENERATION: strict-template vs. comprehension-first vs. anchor-based
- INTERACTIVE: assumption-first vs. constraint-anchored vs. minimal-questions

**Every gene MUST include an output constraint.** A gene without one produces padding.

**Forced lens assignment (when applicable):** If the target skill has reasoning frameworks (e.g., architect's 6 framework files) or blind-spot categories (e.g., brainstorm's 4-category pre-scan), assign each screening agent a mandatory lens — one framework or category per agent. The lens constrains the analytical DOMAIN (where to look) but not the APPROACH (how to look). This guarantees perspective diversity structurally rather than hoping for it through gene design alone.

### Step 6: Run progressive search

Replace flat "N genes × M inputs × K iterations" with a 3-round progressive pipeline:

**Round 1 — Screening (4 agents: 2 genes × 2 inputs)**

Spawn 2 maximally different genes on 2 inputs. Each agent:
1. Reads the relevant code
2. Applies the gene
3. Produces output + self-scores
4. Reports **"Top 3 things I missed"**

Goal: identify the **highest-variance criterion** — which dimension differentiates gene quality most?

**Cross-evaluation sub-step (recommended):** After self-scoring, each agent evaluates one other agent's output — same input, different gene. Discrepancies ≥2 points on any dimension flag perceptual biases the self-evaluating gene literally cannot see. Example: Gene B self-scores Restraint 3/3 because its framework defines multi-finding as thoroughness, while Gene A cross-scores it 1/3. This signal is stronger than self-reported misses for mutation targeting. Cost: +N scoring-only agents per round.

After screening: **aggregate blind spots** (Step 7).

**Round 2 — Focused (6 agents: 3 genes × 2 inputs)**

Design 3 genes that vary ONLY the high-variance dimension. Hold everything else constant.
- 2 genes explore the high-variance dimension from different angles
- **1 gene must target the top blind spot** from Round 1's aggregation

Same inputs as Round 1 for comparability. After focused: aggregate blind spots again.

**Round 3 — Breeding (6 agents: 2 genes × 3 inputs)**

Cross the focused-round winner with the blind-spot gene. Add one NEW input stressing the identified blind spot.

**Skip if the focused round has a clear winner** (score gap > 2 on the targeted dimension).

Total: 10-16 agents per cycle (+ cross-eval and adversarial overhead).

### Step 7: Aggregate blind spots (three-layer detection)

After EACH round, collect blind-spot signals from three sources, weighted by reliability:

**Layer 1 — Self-report (baseline):** Each agent's "Top 3 things I missed." Cheapest signal. Catches self-aware gaps only — things the agent suspects but didn't pursue. Cannot catch confidently wrong assessments.

**Layer 2 — Cross-evaluation discrepancies (Round 1):** Where self-score ≫ cross-score on a dimension, the agent has a perceptual blind spot — its gene defines a failure as success. Invisible to self-report because the agent genuinely believes it performed well. Most valuable in Round 1 when gene diversity is highest.

**Layer 3 — Adversarial aggregation agent (every round):** Spawn one additional agent that reads ALL outputs from the round. This agent:
- Never sees the original task prompt or gene descriptions (input asymmetry forces different reasoning)
- Uses mechanical coverage checking: compare outputs against file tree, git log, dependency graph to find uncovered territory
- Identifies convergent assumptions (things every agent treated as obvious but never questioned)
- Produces alternative solutions no agent explored
- Feeds back via: forced coverage constraints on next-round genes, assumption inversions, and alternative gene seeds

**Aggregation procedure:**

1. Collect all signals from all three layers
2. Cluster by semantic similarity
3. Weight: adversarial findings > cross-eval discrepancies > self-reported misses
4. **Systematic** (flagged by adversarial OR ≥75% of self-reports): structural gap — must address
5. **Perceptual** (flagged by cross-eval discrepancy ≥2): gene-level bias — mutation target
6. **Incidental** (single agent self-report only): gene variance handles it

The top systematic blind spot drives:
- A mutation target for the next round
- A new test input in the breeding round
- A candidate evaluation criterion

**Same-model ceiling:** All three layers use the same underlying model. Biases baked into the model's training distribution cannot be caught by any within-model mechanism. The operator reading actual outputs is the only genuine external validator. Acknowledge this limit — don't claim blind-spot detection is complete.

### Step 8: Iterate

Run 2-3 progressive search cycles. Convergence: same gene wins 2 consecutive cycles → stop.

Measure convergence by ranking stability, not absolute scores (self-scores are inflated but rankings are preserved).

**Phase decomposition:** For multi-phase skills (brainstorm), evolve each phase independently before the full pipeline.

### Step 9: Write the updated skill

Take the winning gene and write it as the new SKILL.md. Preserve:
- The skill's name, description, and argument-hint
- Any supporting files (scripts, templates)
- The original skill's intent (don't change what it does, change how well it does it)

Report: what changed, why, and the score improvement from baseline to final.

---

## Key findings from prior genetic searches

Patterns from 8 skill refinements (~170 agents). Apply to most skills:

**1. The fitness function IS the skill.** Wrong criteria → wrong skill. Bug-finding criteria applied to architect produced a bug-hunter in an architect costume.

**2. "Follow your nose" beats checklists for discovery.** "Check everything" produces lower quality than "follow suspicion." Save checklists for VERIFICATION, not DISCOVERY.

**3. Fewer findings = deeper findings.** "Exactly 3" beats "3-7." The constraint eliminates padding and forces prioritization.

**4. Self-evaluation is archetype-dependent.** 2-pass (generate→evaluate) helps GENERATION, hurts DISCOVERY. The evaluator cuts real findings as "speculative" in discovery skills.

**5. Blind spots are systematic, not random.** Every gene variant misses the same categories (product/UX, operational, security, flag variants, interactive gates). Aggregation catches what gene-tweaking cannot.

**6. The "ONE thing" constraint forces depth.** "Find the ONE most important X" produces deeper analysis than "find 3-5 X's."

**7. Content-type awareness matters.** Skills processing different content types need branching logic. One template rarely works across all types.

**8. Output constraints are as important as discovery methodology.** The constraint (how many findings, how to rank) determines quality as much as discovery approach. Meta-tested: 71 agents.

**9. Context-first calibration beats nose-first.** Reading project docs BEFORE auditing produces more actionable results grounded in project patterns.

**10. Self-scores are inflated but rankings are preserved.** Agents overestimate by 0.5-1.5 points but rank genes correctly. Use for SELECTION, not absolute assessment. "Top 3 things I missed" is the critical signal.

**11. Failure calibration sharpens criteria precision.** Identifying concrete failure modes BEFORE designing criteria produces ~1 point better precision (3.0 vs 2.0 avg). Forces the designer to think about how output goes wrong while LOOKING right, producing false-positive detectors other approaches miss. Meta-tested: failure-calibrated scored 12/15 vs 10.5/15 without.

**12. Archetype classification is a confirmed prerequisite.** Every high-scoring plan started with explicit archetype classification. It drives criteria, gene design, evaluation strategy, and output constraints. Without it, plans default to one-size-fits-all. Meta-tested: 15 agents, 2 iterations.

**13. Blind-spot aggregation improves detection at zero cost.** Aggregating "Top 3 missed" across all agents in a round consistently improved blind-spot detection (2/3 vs 1/3). The intersection reveals systematic gaps no individual gene discovers. Meta-tested: 4 agents.

**14. Progressive search converges with fewer agents.** Screening→focused→breeding (10-16 agents) produces equivalent quality to 45-60 agent flat searches. Screening finds where to focus; focused round resolves it. Meta-tested: 15 agents, 2 iterations.

**15. Cross-evaluation catches perceptual frame biases.** Agents using different genes literally cannot see certain failures because their gene defines the failure as success. Example: a gene framing multi-finding as "thorough analysis" self-scores Restraint 3/3 while a cross-evaluator from a different gene scores 1/3. Self-report cannot surface this — the agent genuinely believes it performed correctly. Meta-tested: 10 agents, C4 improved 1/3 → 2/3.

**16. Adversarial agents need input asymmetry, not adversarial personas.** "Be critical" activates the same biases in different words. Instead: withhold the original task prompt, give only outputs + raw artifacts (file tree, git log). This forces gap-detection reasoning (comparing coverage against input space) instead of quality-evaluation reasoning. Mechanical coverage checking ("which directories got zero mentions?") catches what reasoning-based critique misses.

**17. Forced lens assignment guarantees diversity structurally.** When the target skill has reasoning frameworks or blind-spot categories, assigning each screening agent a single mandatory lens produces deeper analysis than letting agents choose freely. The lens constrains WHERE to look; the gene constrains HOW to look. Synthesis across lens-constrained outputs reveals root causes invisible to any single unconstrained analysis.

**18. Three blind-spot layers are complementary, not competing.** Cross-eval catches gene-level perceptual biases (definition mismatches). Adversarial catches category-level omissions (coverage gaps). Forced lenses catch framework-level tunnel vision (analytical frame lock). Each addresses a different stratum. All three improve blind-spot detection equally (C4: 1/3 → 2/3) but catch different things. No single mechanism reaches 3/3 due to the same-model ceiling. Meta-tested: 10 agents across 3 mechanisms.

---

## Anti-patterns

- **Don't optimize for self-score** — use for ranking only. Read outputs to verify.
- **Don't test on synthetic inputs** — real repo content reveals quality differences toy examples hide.
- **Don't run >3 genes per round** — 2-3 is the sweet spot. More wastes agents.
- **Don't skip "what did I miss?"** — the missed items reveal systematic blind spots for mutation.
- **Don't change >1 dimension per mutation** — can't tell which change helped.
- **Don't skip archetype classification** — one-size-fits-all plans score 1-2 on adaptation.
- **Don't design criteria without failure modes** — abstract "quality" criteria miss false-positive failures. Always identify what "looks good but IS wrong" looks like.
- **Don't use 2-pass for DISCOVERY skills** — the evaluator cuts real findings. Match eval strategy to archetype.
- **Don't rely solely on "Top 3 things I missed"** — self-report catches self-aware gaps only. Confidently wrong assessments are invisible. Use cross-eval and adversarial for the layers self-report can't reach.
- **Don't tell the adversarial agent to "be critical"** — same model, same biases, different words. Change what it SEES (input asymmetry) and what it DOES (gap-finding vs quality-judging).
- **Don't combine lenses × genes in full factorial** — 6 lenses × 3 genes = 18 agents is wasteful. Fix the gene, vary lenses; then fix top lenses, vary genes. The axes are independent.
