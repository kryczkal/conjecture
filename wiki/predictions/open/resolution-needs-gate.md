---
created: 2026-05-25
last_verified: 2026-05-30
type: prediction
status: open
context: [protocol-design, self-correction]
tested_on: [claude-opus-4-6]
---

# The conjecture protocol needs an explicit resolution gate, not just stated rules

The protocol says T0 requires n>=3 observations, but nothing in the workflow prevents resolving a prediction after n=1. The predict, compile, and migrate skills all reference evidence rules but none enforce an observation count check before allowing a prediction to move to confirmed/refuted.

## Mechanism

Completion bias: an LLM (or human) running the predict→test→score loop wants to close the loop. When a test produces clean numbers, the operator treats them as a verdict rather than a single observation. The protocol's rules ("T0 requires n>=3") are stated but not enforced — they rely on the operator to self-regulate, which fails under completion pressure.

## Decisions

- If true: add an explicit gate to the predict skill and compile skill. Before moving a prediction to confirmed/refuted, check: how many independent observations exist? If <3, record the observation and keep the prediction open.
- If false: the stated rules are sufficient. Operators learn to self-regulate after making this mistake once.

## Test plan

1. Observe whether the same mistake (premature resolution) recurs after this prediction is filed.
2. If it recurs: the rules-only approach fails. Build the gate.
3. If it doesn't recur in the next 10 prediction resolutions: the correction was sufficient.

## Fail condition

The premature resolution mistake does not recur in the next 10 prediction resolutions across any project using conjecture. If operators self-correct after one incident, an enforcement gate is unnecessary overhead.

## Observations (n=2)

**conjecture self-test (2026-05-25):** Resolved 4 predictions after 1 test on 1 repo. User caught it. Reverted. The protocol's stated rules did not prevent the error — completion bias overrode them.

**conjecture ingest (2026-05-25):** After the correction, the operator (same LLM) correctly applied the protocol at n=3 — resolved 4 predictions with scorecards, left 2 open for insufficient evidence, filed new knowledge. The correction worked. This is evidence supporting the FAIL condition (operators self-correct after one incident). If this holds for 8 more resolutions, the gate is unnecessary.

**Note (2026-05-30):** The fail condition tests the *rules-only* world ("do operators self-correct without an enforced gate?"). That world no longer fully exists: the `test-hypotheses` skill now bakes the n≥3 gate into §3 as an explicit check. So future resolutions run *with* a structural gate, which confounds the original rules-only experiment — a clean "did they self-regulate unaided?" reading is no longer available. This prediction now mostly resolves toward its "if true → add an explicit gate" branch by construction. Held open at n=2 rather than reframed mid-flight; reconsider whether to refine the fail condition (e.g. "does the *enforced* gate actually catch a premature-resolution attempt in practice?") on the next compile.
