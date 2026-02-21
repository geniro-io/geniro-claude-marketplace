# Geniro Claude Plugin — Development Guidelines

## Version Bumping (MANDATORY)

**After every change to the plugin, you MUST bump the version before committing.**

Version is stored in TWO places — both must be updated:

1. `plugins/geniro-claude-plugin/.claude-plugin/plugin.json` → `"version"` field
2. `.claude-plugin/marketplace.json` → `plugins[0].version` field

### Versioning Rules

Follow semantic versioning (`MAJOR.MINOR.PATCH`):

- **PATCH** (e.g., 1.2.0 → 1.2.1) — bug fixes, typo corrections, wording improvements, minor clarifications in agent/skill instructions
- **MINOR** (e.g., 1.2.0 → 1.3.0) — new features, new agents, new skills, significant behavior changes to existing agents/skills, new workflow phases, new sections in agent instructions
- **MAJOR** (e.g., 1.2.0 → 2.0.0) — breaking changes to the plugin structure, renamed agents/skills, removed functionality, changes that require users to reconfigure

### How to Bump

After making changes, update both files:

```bash
# plugin.json
# Change: "version": "X.Y.Z" → "version": "X.Y.Z+1"

# marketplace.json
# Change: "version": "X.Y.Z" → "version": "X.Y.Z+1"
```

Both versions must always match.

### After Every Change

When you finish making changes to the plugin, always:

1. **Bump the version** in both `plugin.json` and `marketplace.json` (see rules above)
2. **Suggest committing and pushing** — remind the user to commit with a descriptive message and push to remote:
   ```
   I've updated the plugin to version X.Y.Z. Ready to commit and push:
   git add -A && git commit -m "feat/fix/refactor: <description>" && git push origin main
   ```
3. Never leave uncommitted version bumps — the bump and the changes should go in the same commit.

## Plugin Structure

```
geniro-claude-plugin/
├── .claude-plugin/marketplace.json    # Marketplace catalog (root)
└── plugins/geniro-claude-plugin/      # The actual plugin
    ├── .claude-plugin/plugin.json     # Plugin manifest
    ├── CLAUDE.md                      # This file
    ├── agents/                        # Agent definitions (.md)
    ├── skills/                        # Skill definitions (SKILL.md)
    ├── hooks/hooks.json               # Hook configurations
    ├── knowledge/                     # Persistent knowledge base
    ├── settings.json                  # Permission settings
    └── README.md                      # Documentation
```

## Key Conventions

- **Orchestrator** (sonnet) — coordinates only, never explores code. Delegates all exploration to the architect.
- **Architect** (opus) — explores codebases, produces specs, implements minor improvements directly.
- **API/Web agents** (opus) — implement code following the architect's spec.
- **Reviewer** (opus) — reviews code, loops with implementing agents until approved.
- **Knowledge base** — files in `knowledge/` persist across sessions. Paths in skills reference `geniro-claude-plugin/knowledge/`.
