# Conjecture — Development

Claude Code plugin for falsification-driven reasoning. Skills + wiki protocol template.

## Structure

```
conjecture/
├── .claude-plugin/plugin.json   # plugin manifest
├── skills/                      # all skills (discovered by plugin system)
│   ├── init/                    # scaffold wiki in any project
│   ├── predict/                 # predictions + scorecards
│   ├── distill/                 # wiki page compression
│   ├── ingest/                  # multi-layer wiki enrichment
│   ├── evolve/                  # genetic skill improvement
│   ├── wiki-compile/            # governor + lint + action plan
│   └── brainstorm/              # ideas → committed specs
├── protocol/
│   ├── CLAUDE.md                # canonical wiki schema template
│   └── scaffold/                # directory template for /conjecture:init
└── README.md
```

## How skills work

Skills are SKILL.md files. The plugin system discovers them from `skills/`. Each skill reads `wiki/CLAUDE.md` in the target project to understand the wiki's structure before operating.

## Adapting skills

Skills must be project-agnostic. Never hardcode paths to a specific project. Read `wiki/CLAUDE.md` for schema, `wiki/index.md` for state, `wiki/log.md` for recent activity.

## Testing

Test skills by running them in a project with a Conjecture wiki. The iris project (`~/Projects/iris`) is the original instance.
