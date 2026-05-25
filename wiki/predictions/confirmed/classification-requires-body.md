---
created: 2026-05-25
last_verified: 2026-05-25
type: prediction
status: confirmed
context: [skill-design, brainstorm, vimx-migration]
tested_on: [claude-opus-4-6]
---

# Frontmatter type alone is insufficient to classify legacy wiki pages

Legacy pages have types like "finding," "decision," "principle," "gap," "hypothesis." Not all of these are claims about how something works — some are raw data (benchmarks), reference material (contacts), or procedures. Classifying correctly requires reading body content and asking "is this a claim?"

## Mechanism

Frontmatter type is a label the author chose, not a semantic classification. A "finding" could be a knowledge page (claim with evidence), raw data (benchmark results), or a prediction (testable hypothesis labeled as "finding"). The body content reveals which. Non-claim pages (contact info, procedures, raw logs) don't fit the conjecture schema at all and shouldn't be forced into it.

## Decisions

- If true: migrate must read every page's body content. Classification is LLM judgment, not type mapping.
- If false: migrate can use a simple lookup table (finding→knowledge, hypothesis→prediction, etc.) and skip body reads for most pages.

## Test plan

1. Run frontmatter-only type mapping on vimx's 46 legacy pages. Record the mapping.
2. Run body-content classification on the same pages. Record the mapping.
3. Compare: how many pages get different classifications?

## Fail condition

Frontmatter type mapping (finding→knowledge, hypothesis→prediction, principle→framework, benchmark→raw, gap→prediction) is correct >90% of the time without reading body content. If simple mapping works, body-content classification is overhead.

## Scorecard

```yaml
prediction_accuracy: partial
surprise: high
what_the_prediction_missed:
  - body content only changed 7/67 classifications (10.4%) — barely past the 90% threshold
  - the prediction framed body-reading as essential; in practice it's a 10% edge-case handler
  - the 7 pages where it mattered were disproportionately important (axiom vs framework, prediction vs raw)
```

**Evidence:** 7/67 pages (10.4%) had classification changed by body content. Frontmatter-only mapping was 89.6% correct — the prediction survives its fail condition by 0.4%. But the 7 changed pages included high-impact reclassifications: benchmarks-validate-principles-decide (principle→axiom), vimx-bench-v1 (benchmark→prediction), expose-primitives (finding→framework candidate). Body reading is not essential for most pages but catches the ones where getting it wrong matters most.
