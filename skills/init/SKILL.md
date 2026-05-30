---
name: init
description: >
  Scaffold a Conjecture wiki in the current project. Copies the bundled scaffold
  (full directory tree, starter index.md/log.md, and the Axiom zero page) plus the
  canonical protocol. Non-destructive — never clobbers existing wiki content; if
  wiki/CLAUDE.md already exists, offers to migrate instead of overwrite. Use
  /conjecture:init to start reasoning with conjectures in any project.
argument-hint: [--force to overwrite existing wiki/CLAUDE.md]
---

# Initialize Conjecture Wiki

Scaffold the Conjecture reasoning protocol in the current project.

The bundled scaffold at `protocol/scaffold/` is the single source of truth for the starter
tree and pages. This skill copies it rather than hardcoding its contents — so the scaffold
and the installed wiki can never drift apart.

## Step 1: Check existing state

Check if `wiki/` already exists in the current project.

**If wiki/ exists AND wiki/CLAUDE.md exists:**
- Read the existing wiki/CLAUDE.md
- Tell the user: "This project already has a wiki. Options: (1) overwrite wiki/CLAUDE.md with the Conjecture protocol, (2) leave as-is, (3) run /conjecture:migrate to convert it to the schema."
- If the user passed `--force`, overwrite wiki/CLAUDE.md without asking
- Do NOT touch existing wiki content (predictions/, knowledge/, etc.) — only the schema file

**If wiki/ exists but NO wiki/CLAUDE.md:**
- Proceed — add the scaffold + protocol to the existing wiki structure (non-clobbering)

**If wiki/ does not exist:**
- Proceed with full scaffold

## Step 2: Copy the bundled scaffold (non-destructive)

The scaffold — the full directory tree (`predictions/{open,confirmed,refuted}`, `knowledge`, `frameworks`, `axioms`, `graveyard`, `raw`), the starter `index.md` and `log.md`, and `axioms/self-correction.md` (Axiom zero, which the protocol references) — is bundled with the plugin. Copy it without clobbering anything that already exists:

!`SRC="${CLAUDE_SKILL_DIR}/../../protocol/scaffold"; [ -d "$SRC" ] && cp -rn "$SRC/." wiki/ && echo "scaffold copied from $SRC" || echo "ERROR: scaffold not found at $SRC"`

`cp -rn` is no-clobber: existing `index.md`, `log.md`, predictions and knowledge pages are never overwritten. This guarantees the protocol's `axioms/self-correction.md` (Axiom zero) always exists — without it the governor and meta-governor have nothing to ground out to.

## Step 3: Install the protocol

The canonical protocol template is the source of truth — read it from the plugin, do not hardcode it:

!`cp "${CLAUDE_SKILL_DIR}/../../protocol/CLAUDE.md" wiki/CLAUDE.md 2>/dev/null && echo "protocol installed to wiki/CLAUDE.md" || echo "ERROR: protocol/CLAUDE.md not found"`

(If wiki/CLAUDE.md already existed, only overwrite it with `--force` or explicit user approval per Step 1.)

## Step 4: Stamp dates and append the init log entry

The bundled scaffold uses `YYYY-MM-DD` placeholders. Replace every `YYYY-MM-DD` with today's date in the copied files (`wiki/index.md`, `wiki/log.md`, `wiki/axioms/self-correction.md`), then append the init entry to `wiki/log.md`:

```markdown
## [YYYY-MM-DD] init | Conjecture wiki initialized

Scaffolded wiki/ from the bundled scaffold + protocol. Ready for first prediction.
```

(Replace YYYY-MM-DD with today's date.)

## Step 5: Update project CLAUDE.md

If the project has a `CLAUDE.md` at its root, check if it already references `wiki/`. If not, append:

```markdown

## Wiki

Knowledge synthesis lives in `wiki/`. Schema at `wiki/CLAUDE.md`. Operations: test-hypotheses, ingest, distill, wiki-compile.
```

If there is no root `CLAUDE.md`, do NOT create one — that's the user's choice.

## Step 6: Report

Tell the user:

```
Conjecture wiki initialized in wiki/.

Next steps:
- File your first prospective bet (predictions/open/) with a fail condition before your next task
- /conjecture:test-hypotheses to run an open prediction's test and book the result
- /conjecture:ingest <file> to enrich the wiki with external knowledge
- /conjecture:wiki-compile to check learning health

The loop only learns once predictions get tested. File your first prospective bet
(predictions/open/) with a fail condition before the next significant task, then
/conjecture:test-hypotheses it once you have evidence.
```

## Rules

- Never overwrite existing wiki content (predictions, knowledge, etc.) — `cp -rn` enforces this
- Only overwrite wiki/CLAUDE.md with --force or explicit user approval
- The scaffold and protocol templates are the source of truth — copy them, don't hardcode their contents
- `axioms/self-correction.md` (Axiom zero) must exist after init — it is the foundation the governor references
