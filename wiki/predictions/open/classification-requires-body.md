---
created: 2026-05-25
last_verified: 2026-05-25
type: prediction
status: open
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

## Observations (n=1, not yet confirmable)

**vimx (2026-05-25):** 7/67 pages (10.4%) had classification changed by body content. Frontmatter-only mapping was 89.6% correct — barely below the 90% fail threshold. The 7 changed pages were high-impact: benchmarks-validate-principles-decide (principle→axiom), vimx-bench-v1 (benchmark→prediction), expose-primitives (finding→framework candidate). Direction: weakly supports prediction.

Needs: iris run + another repo. vimx had relatively consistent naming (type matched content ~90%). A messier wiki could push the number much higher or lower. The 0.4% margin is too thin to call on n=1.
