---
name: init
description: >
  Scaffold a Conjecture wiki in the current project. Creates the wiki/ directory
  with the full protocol (learning atom, governor, predictions, evidence maturity),
  starter pages, and all required directories. Non-destructive — if wiki/ already
  exists, offers to migrate instead of overwrite. Use /conjecture:init to start
  reasoning with conjectures in any project.
argument-hint: [--force to overwrite existing wiki/CLAUDE.md]
---

# Initialize Conjecture Wiki

Scaffold the Conjecture reasoning protocol in the current project.

## Step 1: Check existing state

Check if `wiki/` already exists in the current project.

**If wiki/ exists AND wiki/CLAUDE.md exists:**
- Read the existing wiki/CLAUDE.md
- Tell the user: "This project already has a wiki. Options: (1) overwrite wiki/CLAUDE.md with the Conjecture protocol, (2) leave as-is."
- If the user passed `--force`, overwrite without asking
- Do NOT touch existing wiki content (predictions/, knowledge/, etc.) — only the schema file

**If wiki/ exists but NO wiki/CLAUDE.md:**
- Proceed — add the protocol to the existing wiki structure

**If wiki/ does not exist:**
- Proceed with full scaffold

## Step 2: Create directory structure

Create all directories that don't already exist:

```
wiki/
├── predictions/
│   ├── open/
│   ├── confirmed/
│   └── refuted/
├── knowledge/
├── frameworks/
├── axioms/
├── graveyard/
└── raw/
```

Use `mkdir -p` — safe to run on existing directories.

## Step 3: Copy protocol

Read `${CLAUDE_SKILL_DIR}/../../../protocol/CLAUDE.md` (the canonical Conjecture protocol template).

Write it to `wiki/CLAUDE.md`.

## Step 4: Create starter pages

**wiki/index.md** (only if it doesn't exist):
```markdown
# Index

What this project currently believes. Updated as predictions are confirmed/refuted and knowledge accumulates.

## Active predictions

(none yet — use /conjecture:predict before your next significant task)

## Knowledge

(none yet — findings will accumulate as predictions are tested)

## Frameworks

(none yet — interpretive lenses emerge from patterns in confirmed predictions)
```

**wiki/log.md** (only if it doesn't exist):
```markdown
# Log

Chronological record of wiki operations. Append-only.

## [YYYY-MM-DD] init | Conjecture wiki initialized

Scaffolded wiki/ with the Conjecture protocol. Ready for first prediction.
```

(Replace YYYY-MM-DD with today's date.)

## Step 5: Update project CLAUDE.md

If the project has a `CLAUDE.md` at its root, check if it already references `wiki/`. If not, append:

```markdown

## Wiki

Knowledge synthesis lives in `wiki/`. Schema at `wiki/CLAUDE.md`. Operations: predict, ingest, distill, wiki-compile.
```

If there is no root `CLAUDE.md`, do NOT create one — that's the user's choice.

## Step 6: Report

Tell the user:

```
Conjecture wiki initialized in wiki/.

Next steps:
- /conjecture:predict before your next task (state what you expect)
- /conjecture:ingest <file> to enrich the wiki with external knowledge
- /conjecture:wiki-compile to check learning health
```

## Rules

- Never overwrite existing wiki content (predictions, knowledge, etc.)
- Only overwrite wiki/CLAUDE.md with --force or explicit user approval
- Create starter pages only if they don't exist
- The protocol template is the source of truth — read it from the plugin, don't hardcode it
