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

## Observations (n=3)

**vimx (2026-05-25):** 7/67 pages (10.4%) changed by body. Frontmatter-only was 89.6% correct — barely below 90% threshold. Direction: weakly supports.

**swarm (2026-05-25):** 7/41 pages (17.1%) changed by body. Frontmatter-only was 82.9% correct. Key changes: 2 principles→axioms (constitutional rules), 1 decision→T2 (production-verified), 1 gap→knowledge (partially-addressed), 1 status→knowledge. Direction: supports.

**jarvis (2026-05-25):** 14/78 pages (17.9%) changed by body. Frontmatter-only was 82.1% correct. Key changes: gaps with unusual statuses (refuted, closed-by-decision), principles→frameworks, status pages→raw or knowledge depending on content. Direction: supports.

Frontmatter-only accuracy: vimx 89.6%, swarm 82.9%, jarvis 82.1%. vimx was the outlier — its type naming was unusually consistent. On messier wikis, body reading catches 17-18% of pages. All three below the 90% fail threshold.
