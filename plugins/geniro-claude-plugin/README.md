# Geniro Claude Plugin

Multi-agent orchestrator plugin for the Geniro platform. Provides a full development pipeline with **self-improving knowledge**: architect designs the plan, validators verify it, engineers implement, reviewers + security auditor + test reviewer catch problems, and the system learns from every task.

## Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                         /orchestrate                                 │
│                     (Pipeline Controller)                            │
│                                                                     │
│  0. Load feature spec (if from backlog) + knowledge base            │
│  1. Architect analyzes & designs spec with execution waves          │
│  1b. Skeptic + Completeness Validator verify spec (parallel)        │
│  2. User reviews & approves the plan                                │
│  3. Engineers implement (API + Web, following waves)                 │
│  4. Reviewer + Security Auditor + Test Reviewer (parallel)          │
│  4b. Integration test gate                                          │
│  5. User feedback loop                                              │
│  6. Final verification & summary                                    │
│  7. Extract & save learnings to knowledge base                      │
└──┬─────┬──────┬──────┬──────┬──────┬──────┬──────┬─────────────────┘
   │     │      │      │      │      │      │      │
┌──▼──┐┌─▼───┐┌─▼───┐┌─▼───┐┌─▼───┐┌─▼───┐┌─▼───┐┌─▼──────┐┌─▼──────┐
│archi││ api ││ web ││revie││skept││secur││compl││test    ││clean  │
│tect ││agent││agent││wer  ││ic   ││ity  ││ete- ││revie- ││up     │
│     ││     ││     ││     ││     ││audit││ness ││wer    ││       │
│opus ││opus ││opus ││opus ││opus ││opus ││opus ││opus   ││haiku  │
└─────┘└─────┘└─────┘└─────┘└─────┘└─────┘└─────┘└───────┘└───────┘
                  │
            ┌─────▼─────┐
            │ .claude/   │  Persistent learnings
            │ project-   │  fed back into every
            │ knowledge/ │  future task
            │            │
            │ Patterns   │  Feature backlog in
            │ Gotchas    │  .claude/project-
            │ Decisions  │  features/
            │ Feedback   │
            └────────────┘
```

## Installation

```bash
claude plugin add ./geniro-claude-marketplace
```

**Important:** Always run Claude Code from the **root directory** that contains all Geniro projects side by side:

```
Geniro/                          ← run `claude` from HERE
├── geniro/                      # API backend (NestJS)
├── geniro-web/                  # Web frontend (React + Vite)
├── geniro-claude-marketplace/   # This plugin repo
└── .claude/                     # Shared config, knowledge, features
    ├── project-knowledge/       # Accumulated learnings
    └── project-features/        # Feature backlog
```

The plugin expects this layout. Agents navigate into `geniro/` and `geniro-web/` from the root CWD. Knowledge and feature specs are stored in `.claude/` at the root level so they're shared across all repos.

## Available Commands

### `/geniro-claude-plugin:orchestrate [feature description | feature: <name> | next]`

The main entry point. Runs the full pipeline: load knowledge → architect → validate spec → user approval → implement → review (3 agents) → integration tests → deliver → archive feature → save learnings.

**Examples:**
```
# Ad-hoc feature description
/geniro-claude-plugin:orchestrate Add a GraphRevisionProgress WebSocket event that shows per-node rebuild progress during live updates

# Implement a feature from the backlog (created with /new-feature)
/geniro-claude-plugin:orchestrate feature: graph-revision-progress

# Implement the next approved feature from the backlog
/geniro-claude-plugin:orchestrate next
```

When implementing from the backlog, the spec is loaded from `.claude/project-features/` and archived to `.claude/project-features/completed/` on success.

### `/geniro-claude-plugin:plan [task description]`

Run just the architect to produce an implementation-ready specification without executing it.

**Example:**
```
/geniro-claude-plugin:plan Add per-graph concurrency with locks to the revision queue
```

### `/geniro-claude-plugin:new-feature [feature description]`

Create a new feature or task spec via structured interview. Interviews you about the feature, explores the codebase, and saves a complete spec to `.claude/project-features/`. Use when you want to plan a feature before implementing it.

**Example:**
```
/geniro-claude-plugin:new-feature Add a graph template marketplace where users can share and import graph configurations
```

After the interview, implement with:
```
/geniro-claude-plugin:orchestrate feature: graph-template-marketplace
```

### `/geniro-claude-plugin:features [list|next|complete|status|show]`

Manage the feature backlog. List all features, check what's next, mark features as complete.

**Examples:**
```
/geniro-claude-plugin:features list
/geniro-claude-plugin:features next
/geniro-claude-plugin:features complete graph-template-marketplace
/geniro-claude-plugin:features show thread-auto-naming
```

### `/geniro-claude-plugin:spec [feature description]`

Conduct a structured requirements interview (ad-hoc, does not save to backlog). Three rounds: general understanding → code-informed questions → edge cases. For saved specs, use `/new-feature` instead.

**Example:**
```
/geniro-claude-plugin:spec Quick question about how graph execution handles concurrent triggers
```

### `/geniro-claude-plugin:api-task [task description]`

Directly run a backend-only task in `geniro/`. Skips architect and reviewer.

**Example:**
```
/geniro-claude-plugin:api-task Add retention pruning to graph-revision.service.ts — delete revisions older than the last 50 per graph
```

### `/geniro-claude-plugin:web-task [task description]`

Directly run a frontend-only task in `geniro-web/`. Skips architect and reviewer.

**Example:**
```
/geniro-claude-plugin:web-task Add a progress bar to the revision applying toast that shows completedNodes/totalNodes
```

### `/geniro-claude-plugin:review [what to review]`

Run the reviewer on any changes.

**Examples:**
```
/geniro-claude-plugin:review recent changes to the graph revision service
/geniro-claude-plugin:review all uncommitted changes
```

### `/geniro-claude-plugin:skeptic [spec to validate]`

Validate an architect specification against the actual codebase. Detects mirages: references to nonexistent files, functions, or patterns.

**Example:**
```
/geniro-claude-plugin:skeptic [paste architect spec here]
```

### `/geniro-claude-plugin:security-audit [what to audit]`

Run an OWASP Top 10 focused security audit on recent code changes.

**Example:**
```
/geniro-claude-plugin:security-audit recent changes to the auth module
```

### `/geniro-claude-plugin:learn [add/view/search/cleanup/validate/stats]`

Manage the knowledge base manually. View accumulated learnings, add entries, search, clean up stale knowledge, validate health, or get stats.

**Examples:**
```
/geniro-claude-plugin:learn view
/geniro-claude-plugin:learn add TypeORM migrations must be run before integration tests when schema changes
/geniro-claude-plugin:learn search WebSocket
/geniro-claude-plugin:learn cleanup
/geniro-claude-plugin:learn validate report
/geniro-claude-plugin:learn stats
```

### `/geniro-claude-plugin:validate-knowledge [fix|report]`

Run a dedicated knowledge base health check: detect stale references, duplicates, and generic framework knowledge.

**Examples:**
```
/geniro-claude-plugin:validate-knowledge report
/geniro-claude-plugin:validate-knowledge fix
```

## Model Configuration

| Component | Model | Rationale |
|-----------|-------|-----------|
| Orchestrator | Sonnet | Coordination logic — fast and cost-effective |
| Architect | Opus | Deep analysis and design require strongest reasoning |
| API Agent | Opus | Complex implementation with strict quality bar |
| Web Agent | Opus | Complex implementation with strict quality bar |
| Reviewer | Opus | Thorough code review requires deep understanding |
| Skeptic | Opus | Verifying spec claims against codebase requires precision |
| Security Auditor | Opus | OWASP analysis requires deep security understanding |
| Completeness Validator | Opus | Requirements traceability requires analytical reasoning |
| Test Reviewer | Opus | Test quality evaluation requires understanding intent vs. assertion |
| Cleanup Agent | Haiku | Fast garbage detection and removal — simple file/process checks |
| Learn Manager | Sonnet | Knowledge CRUD — straightforward operations |
| Knowledge Validator | Sonnet | File existence checks — straightforward operations |
| Feature Interview | Opus | Deep requirements gathering and codebase exploration |
| Feature Manager | Haiku | Simple file listing and status management |

## Feature Backlog System

The plugin supports a **spec-first workflow**: create feature specs via structured interview, then implement them from the backlog.

### Workflow

1. **Create a feature spec**: `/new-feature Add user authentication with OAuth`
   - Agent interviews you (3 rounds: understanding → code-informed → edge cases)
   - Saves spec to `.claude/project-features/<kebab-name>.md`

2. **Manage the backlog**: `/features list` to see all features and their status

3. **Implement from backlog**: `/orchestrate feature: <name>` or `/orchestrate next`
   - Loads the spec, sets status to `in-progress`
   - Runs the full pipeline (architect → implement → review → test)
   - On completion, archives to `.claude/project-features/completed/`

### Feature Spec Format

Each feature is a Markdown file with YAML frontmatter:
- `name`: kebab-case identifier
- `status`: `draft` → `approved` → `in-progress` → `completed`
- `size`: S, M, or L
- `type`: feature, bugfix, refactor, or task
- `created` / `updated`: timestamps

### File Structure

```
.claude/
├── project-features/               # Feature backlog
│   ├── graph-template-market.md    # Approved, ready to implement
│   ├── thread-auto-naming.md       # In progress
│   └── completed/                  # Archived completed features
│       └── runtime-health-check.md
└── project-knowledge/              # All accumulated learnings
    ├── api-learnings.md            # API patterns, gotchas, commands
    ├── web-learnings.md            # Web patterns, gotchas, components
    ├── architecture-decisions.md   # Design choices with rationale
    └── review-feedback.md          # Recurring reviewer findings
```

## Self-Improvement System

The plugin maintains a persistent knowledge base in project-specific `.claude/project-knowledge/` directories that grows with every task.

### How It Works

1. **Before each task** (Phase 0) — the orchestrator reads all knowledge files and extracts entries relevant to the current task
2. **During delegation** — relevant knowledge is passed to agents as "Knowledge Context" so they can avoid known pitfalls and follow proven patterns
3. **After each task** (Phase 7) — the orchestrator reviews the full execution (architect spec, engineer reports, reviewer feedback, security findings, mirage patterns) and extracts new learnings
4. **Agents contribute** — engineers and reviewer report discoveries in their output, which the orchestrator saves
5. **Health checks** — the `/validate-knowledge` command detects stale references, duplicates, and generic framework knowledge

### What Gets Captured

- **Patterns** — reusable approaches for specific feature areas (API and Web separately)
- **Gotchas** — things that went wrong, with root cause and prevention
- **Architecture decisions** — significant design choices with context and rationale
- **Review feedback** — recurring issues with frequency tracking (including security findings)
- **Mirage patterns** — what the architect got wrong in specs and why (for future accuracy)
- **Useful commands** — non-obvious CLI workflows
- **Test patterns** — effective testing approaches for specific scenarios
- **Component patterns** — reusable UI patterns (Web)

### Knowledge Files

All files in `.claude/project-knowledge/`:

| File | Contents |
|------|----------|
| `api-learnings.md` | API patterns, gotchas, test patterns, commands |
| `web-learnings.md` | Web patterns, gotchas, component patterns, commands |
| `architecture-decisions.md` | Design choices with rationale and consequences |
| `review-feedback.md` | Recurring reviewer findings, security patterns, quality trends |

## Agents

| Agent | Role | Works In | Specialization |
|-------|------|----------|---------------|
| `architect-agent` | Design | Both repos | Codebase analysis, implementation specs with execution waves, risk assessment, test scenarios |
| `api-agent` | Implement | `geniro/` | NestJS services, TypeORM entities, Zod DTOs, Vitest tests, migrations |
| `web-agent` | Implement | `geniro-web/` | React components, Refine hooks, Ant Design UI, Socket.io events |
| `reviewer-agent` | Quality gate | Both repos | Code review, AI-pattern detection, security checklist, architecture fit, test quality |
| `skeptic-agent` | Spec validation | Both repos | Mirage detection — verifies spec references against actual codebase |
| `security-auditor-agent` | Security review | Both repos | OWASP Top 10 audit adapted for NestJS + React stack |
| `completeness-validator-agent` | Requirements check | N/A (reads spec) | Bidirectional traceability — requirements ↔ spec ↔ tests |
| `test-reviewer-agent` | Test quality | Both repos | Litmus test, assertion quality, pyramid balance, scenario coverage |
| `cleanup-agent` | Post-pipeline cleanup | Both repos | Deletes leftover screenshots, temp files, stops lingering servers, detects garbage |

## Full Pipeline Flow

```
You describe a feature
        │
        ▼
┌─────────────────┐
│  0. FEATURE &    │  Load feature spec from backlog (if applicable).
│     KNOWLEDGE    │  Load accumulated learnings. Pass relevant
│                  │  context to all downstream agents.
└────────┬────────┘
         ▼
┌─────────────────┐
│  1. ARCHITECT    │  Explores both codebases, designs minimal changes,
│                  │  produces file-level spec with execution waves
│                  │  and test scenarios.
└────────┬────────┘
         ▼
┌─────────────────┐
│ 1b. VALIDATE     │  Skeptic + Completeness Validator run in parallel.
│     SPEC         │  Skeptic catches mirages (nonexistent references).
│                  │  Completeness catches dropped requirements.
│                  │  Failures → back to architect for revision.
└────────┬────────┘
         ▼
┌─────────────────┐
│  2. USER REVIEW  │  You see the plan: scope, risk, files, approach,
│                  │  validation results. Approve or request changes.
└────────┬────────┘
         ▼
┌─────────────────┐
│  3. IMPLEMENT    │  api-agent + web-agent work from the spec.
│                  │  Follow execution waves for parallelism.
│                  │  Each runs pnpm run full-check before reporting.
└────────┬────────┘
         ▼
┌─────────────────┐
│  4. REVIEW       │  Three agents run in parallel:
│                  │  - reviewer-agent (code quality, architecture)
│                  │  - security-auditor-agent (OWASP Top 10)
│                  │  - test-reviewer-agent (test quality)
│                  │  Blocking issues → fix loop back to step 3
│                  │  All clear → proceed
└────────┬────────┘
         ▼
┌─────────────────┐
│ 4b. INT TESTS    │  Discover + run all related integration tests.
│                  │  Write missing tests if needed.
└────────┬────────┘
         ▼
┌─────────────────┐
│  5. FEEDBACK     │  Present results to user. Iterate on changes
│                  │  until satisfied.
└────────┬────────┘
         ▼
┌─────────────────┐
│  6. SUMMARY &    │  Archive feature to completed/ (if from backlog).
│     CLEANUP      │  Final build verification, cleanup-agent sweep,
│                  │  summary report with files, decisions, steps.
└────────┬────────┘
         ▼
┌─────────────────┐
│  7. LEARN        │  Extract patterns, gotchas, decisions, security
│                  │  findings, and mirage patterns. Save to
│                  │  knowledge base for future tasks.
└─────────────────┘
```

## Plugin Structure

```
geniro-claude-marketplace/
├── .claude-plugin/
│   └── plugin.json              # Plugin manifest
├── agents/
│   ├── architect-agent.md       # Architect: analyze & design specs
│   ├── api-agent.md             # Engineer: API backend
│   ├── web-agent.md             # Engineer: Web frontend
│   ├── reviewer-agent.md        # Reviewer: quality gate
│   ├── skeptic-agent.md         # Skeptic: spec validation
│   ├── security-auditor-agent.md # Security: OWASP Top 10 audit
│   ├── completeness-validator-agent.md # Completeness: requirements traceability
│   ├── test-reviewer-agent.md   # Test reviewer: test quality evaluation
│   └── cleanup-agent.md        # Cleanup: post-pipeline garbage sweep
├── hooks/
│   └── hooks.json               # Lifecycle hooks
├── skills/
│   ├── orchestrate/
│   │   └── SKILL.md             # Full pipeline command (supports feature backlog)
│   ├── new-feature/
│   │   └── SKILL.md             # Feature spec creation via interview
│   ├── features/
│   │   └── SKILL.md             # Feature backlog management (list/next/complete)
│   ├── plan/
│   │   └── SKILL.md             # Architect-only command
│   ├── spec/
│   │   └── SKILL.md             # Requirements interview (ad-hoc)
│   ├── api-task/
│   │   └── SKILL.md             # Direct API task command
│   ├── web-task/
│   │   └── SKILL.md             # Direct Web task command
│   ├── review/
│   │   └── SKILL.md             # Direct review command
│   ├── skeptic/
│   │   └── SKILL.md             # Standalone spec validation
│   ├── security-audit/
│   │   └── SKILL.md             # Standalone security audit
│   ├── learn/
│   │   └── SKILL.md             # Knowledge base management
│   └── validate-knowledge/
│       └── SKILL.md             # Knowledge base health check
├── settings.json                # Default permissions
└── README.md
```

## Recommended: Secret Scanning

For additional security, install [gitleaks](https://github.com/gitleaks/gitleaks) and add it as a pre-commit hook in your Geniro repositories:

```bash
# Install gitleaks
brew install gitleaks

# Run manually before committing
gitleaks detect --source . --verbose
```

This catches accidentally committed secrets, tokens, and API keys before they reach the repo.
