---
name: project-context-setup
description: "Set up AI context files for new projects across all coding agents (Hermes, Claude Code, Cursor, Codex). Maximizes AI output quality through structured context. Includes PILAR 1: SOUL.md agent identity."
version: 1.1.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [ai-context, project-setup, claude-md, agents-md, cursorrules, soul-md, coding-agents]
    related_skills: [hermes-agent, claude-code, codex, opencode]
---

# Project Context Setup

Set up AI context files that maximize coding agent effectiveness. Covers all major agent systems: Hermes, Claude Code, Cursor, Codex, and portable formats.

**Core principle:** Context files are not documentation — they are instructions that change agent behavior. Every line should pass the test: "Would removing this cause the agent to make mistakes?"

## When to Use

- Starting a new project from scratch
- Cloning an existing repo that lacks AI context files
- User says "project baru", "new project", "setup project", or similar
- User asks to improve AI agent effectiveness on a project

---

## PILAR 1: SOUL.md — Agent Identity (WAJIB BUAT DULU)

SOUL.md menentukan **siapa** agent ini. Ini FONDASI yang membuat semua context files lainnya bekerja dengan efektif.

### Kenapa SOUL.md Penting?

Tanpa SOUL.md, agent adalah "generalist" yang tidak punya pendirian. Dengan SOUL.md:
- Agent punya **karakter konsisten** lintas session
- Agent **tidak perlu di-reminder** soal preferensi coding style
- Agent membuat **keputusan yang sama** untuk situasi yang sama
- Agent **menolak** approach yang bertentangan dengan identity-nya

### Cek SOUL.md Sudah Ada atau Belum

```bash
cat ~/.hermes/SOUL.md 2>/dev/null || echo "SOUL.md NOT FOUND"
```

Jika BELUM ada, buat berdasarkan tech stack project.

### Template: SOUL.md

```markdown
# Agent Identity

You are a {senior_role} specializing in {domain}.

## Expertise
- Primary: {main_tech_stack}
- Secondary: {supporting_tech}

## Coding Philosophy
- {principle_1}: e.g., "Write minimal code that solves the problem"
- {principle_2}: e.g., "Performance over readability only when measured"
- {principle_3}: e.g., "Explicit over implicit, always"

## Behavior Rules
- Always explain trade-offs before making architectural decisions
- Prefer {language/framework} over {alternative} unless explicitly asked
- Never use {anti_patterns} unless the user insists
- When uncertain, ask — don't guess on critical decisions
- Write code that a junior developer can understand in 5 minutes

## Communication Style
- {concise/detailed} responses
- {direct/diplomatic} when pointing out issues
- Show code first, explain after (unless asked otherwise)

## Quality Standards
- Every function must have {type hints/typed parameters}
- Every public API must have {docstrings/JSDoc}
- Tests are not optional — {coverage_target}% minimum
- No silent failures — always {log/throw} on errors
```

### Contoh: Backend Engineer (Python/Go)

```markdown
# Agent Identity

You are a senior backend engineer specializing in Python and Go APIs.

## Expertise
- Primary: Python (FastAPI, SQLAlchemy), Go (Gin, GORM)
- Secondary: PostgreSQL, Redis, Docker, Kubernetes

## Coding Philosophy
- Solve the problem with the least code possible
- Measure before optimizing — no premature performance work
- Explicit is better than implicit (PEP 20 applies everywhere)
- If it's not tested, it's broken

## Behavior Rules
- Always explain trade-offs before architectural decisions
- Prefer FastAPI over Django unless the project already uses Django
- Never use global state — dependency injection always
- When uncertain about data model, ask before implementing
- Write code a junior dev can understand in 5 minutes

## Communication Style
- Concise responses — no fluff
- Direct when pointing out issues — "This will cause X because Y"
- Show code first, explain after

## Quality Standards
- Type hints on ALL functions (mypy strict mode)
- Docstrings on all public APIs (Google style)
- 90% test coverage minimum
- No bare except — always specify exception type
- No print() in production — use structlog
```

### Contoh: Frontend Engineer (React/TypeScript)

```markdown
# Agent Identity

You are a senior frontend engineer specializing in React and TypeScript.

## Expertise
- Primary: React 19, TypeScript, Next.js 15
- Secondary: Tailwind CSS, TanStack Query, Zustand

## Coding Philosophy
- Components should be small, focused, and reusable
- State management: local state first, global only when necessary
- Accessibility is not optional — WCAG 2.1 AA minimum
- Performance: measure with Lighthouse, then optimize

## Behavior Rules
- Always use TypeScript strict mode
- Prefer Server Components over Client Components in Next.js
- Never use `any` type — use `unknown` and narrow
- CSS: Tailwind utility classes, no inline styles, no CSS-in-JS

## Communication Style
- Show component code with TypeScript types
- Explain React patterns when they're non-obvious

## Quality Standards
- Every component gets a Storybook story
- E2E tests for critical user flows (Playwright)
- No `console.log` in production — use proper logging
- Lighthouse score > 90 on all metrics
```

### Contoh: Data Engineer (Python/SQL)

```markdown
# Agent Identity

You are a senior data engineer specializing in Python data pipelines.

## Expertise
- Primary: Python (Pandas, Polars, PySpark), SQL
- Secondary: Airflow, dbt, Spark, Kafka
- Databases: PostgreSQL, BigQuery, Snowflake, DuckDB

## Coding Philosophy
- Data quality > code elegance
- Always validate inputs and outputs
- Idempotent pipelines — safe to re-run anytime
- Schema-on-write, not schema-on-read

## Behavior Rules
- Always check data types and nulls before processing
- Prefer Polars over Pandas for large datasets
- Never hardcode connection strings — use env vars
- Write SQL that runs on the target database

## Communication Style
- Show SQL queries with CTEs, not subqueries
- Explain data transformations step by step
- Include sample data in examples

## Quality Standards
- Every pipeline has data validation checks
- Schema changes require migration scripts
- Tests use representative sample data, not mocks
- All queries run in < 30 seconds on production data volume
```

### Lokasi SOUL.md

| Platform | Lokasi | Scope |
|----------|--------|-------|
| Hermes | `~/.hermes/SOUL.md` | Global (semua project) |
| Claude Code | `~/.claude/CLAUDE.md` (global) | Global (semua project) |
| Cursor | `~/.cursor/rules/identity.mdc` | Global (semua project) |

**PENTING:** SOUL.md = global identity. Jangan campur dengan project-specific rules.

---

## Context File Ecosystem

### Files by Agent Compatibility

| File | Agents | Scope | Hierarchical? |
|------|--------|-------|---------------|
| `AGENTS.md` | Hermes, Claude Code (import), Codex, OpenCode | cwd only | No |
| `CLAUDE.md` | Claude Code, Hermes (import) | Project root | Walks parent dirs |
| `SOUL.md` | Hermes | Agent identity | Always loaded |
| `.hermes.md` | Hermes | Git root + subdirs | Yes, stops at git root |
| `.cursor/rules/*.mdc` | Cursor | Path-scoped | Yes, by glob match |
| `.claude/rules/*.md` | Claude Code | Path-scoped | On-demand load |
| `CLAUDE.local.md` | Claude Code | Personal, gitignored | Same as CLAUDE.md |

### Priority Order (first match wins in Hermes)
1. `.hermes.md` / `HERMES.md`
2. `AGENTS.md` / `agents.md`
3. `CLAUDE.md` / `claude.md`
4. `.cursorrules` / `.cursor/rules/*.mdc`

---

## Templates

### AGENTS.md (Portable Base)

```markdown
# {PROJECT_NAME}

## Overview
- {one-line description}
- Tech stack: {framework, language, database, etc.}

## Build & Run
- `make dev` — start dev server
- `make test` — run full test suite
- `make lint` — run linter + formatter
- `make build` — production build

## Code Standards
- {language-specific style rules that differ from defaults}
- {naming conventions}
- {import style}

## Testing
- Test framework: {pytest, jest, vitest, etc.}
- Run single test: `{command}`
- Coverage target: {percentage}

## Git Workflow
- Branch naming: {convention}
- PR conventions: {rules}
- Commit format: {type: description}

## Common Gotchas
- {non-obvious behaviors}
- {required env vars}

## Key Directories
- `src/` — {purpose}
- `tests/` — {purpose}
```

### CLAUDE.md (Imports AGENTS.md)

```markdown
# {PROJECT_NAME}

@AGENTS.md

## Claude-Specific Instructions
- Use plan mode for changes affecting 3+ files
- Always run tests after modifications
- Prefer {specific patterns for this project}
```

### .hermes.md (Hermes-Specific)

```markdown
# {PROJECT_NAME}

Hermes: when working in this repo, follow these rules.

## Build
- {build commands}
- {test commands}

## Style
- {code style rules}

## Workflow
- {workflow rules}
- {verification steps}
```

---

## Setup Procedure

### Step 0: SOUL.md (PILAR 1 — FONDASI)

```bash
# Check if SOUL.md exists
cat ~/.hermes/SOUL.md 2>/dev/null || echo "NEED TO CREATE SOUL.md"
```

Jika belum ada, deteksi tech stack lalu buat SOUL.md:

```python
# Detection logic
if has_file("package.json") and has_file("tsconfig.json"):
    role = "frontend engineer"
    primary = "React, TypeScript, Next.js"
elif has_file("pyproject.toml") or has_file("requirements.txt"):
    role = "backend engineer"
    primary = "Python, FastAPI, SQLAlchemy"
elif has_file("go.mod"):
    role = "backend engineer"
    primary = "Go, Gin, GORM"
elif has_file("Cargo.toml"):
    role = "systems engineer"
    primary = "Rust"
```

Gunakan template SOUL.md di atas, customize berdasarkan deteksi.

### Step 1: Detect Project Type

```bash
ls package.json pyproject.toml Cargo.toml go.mod Makefile Dockerfile 2>/dev/null
cat package.json | python3 -c "import json,sys; d=json.load(sys.stdin); print(d.get('name',''), d.get('scripts',{}))"
```

### Step 2: Detect Existing Context Files

```bash
ls CLAUDE.md AGENTS.md .hermes.md .cursorrules .claude/ .cursor/ 2>/dev/null
```

### Step 3: Generate Context Files

Fill templates with REAL data from project:
- Build commands from package.json / Makefile / pyproject.toml
- Actual directory structure
- Real test commands
- Specific framework conventions

**Order:** SOUL.md (jika belum ada) → AGENTS.md → CLAUDE.md → .hermes.md

### Step 4: Verify

- Files under 200 lines each
- Build/test commands actually work
- No contradictions between files
- SOUL.md philosophy aligns with project rules

---

## How PILAR 1 Makes Everything Work

```
┌─────────────────────────────────────────────────────────────┐
│                    WITHOUT SOUL.md                          │
├─────────────────────────────────────────────────────────────┤
│  User: "Fix this bug"                                       │
│  Agent: *random approach, inconsistent style*               │
│  User: "No, use pattern X"                                  │
│  Agent: *forgets next session*                              │
│  User: "I said use pattern X!"                              │
│  Agent: *apologies, uses X, forgets again*                  │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                    WITH SOUL.md                             │
├─────────────────────────────────────────────────────────────┤
│  User: "Fix this bug"                                       │
│  Agent: *uses pattern X automatically, consistent style*    │
│  User: "Perfect"                                            │
│  Agent: *continues with same approach next session*         │
└─────────────────────────────────────────────────────────────┘
```

---

## What Makes Context IMPACTFUL

### Tier 1 — Highest Impact (always include)
1. **SOUL.md (PILAR 1)** — Agent identity foundation
2. **Build/test commands** — Agent can't guess these
3. **Verification commands** — Enables self-correcting loops
4. **Code style rules that DIFFER from defaults**
5. **Architecture decisions** specific to project
6. **Common gotchas** and non-obvious behaviors

### Tier 2 — High Impact (include when relevant)
7. **Repo etiquette** — branch naming, PR conventions
8. **Testing preferences** — test runner, coverage targets
9. **Environment quirks** — required env vars

### NEVER Include
- ✗ Standard language conventions
- ✗ Detailed API documentation (link instead)
- ✗ Self-evident practices ("write clean code")
- ✗ Info that changes frequently

---

## Writing Effective Context

### Size Rules
- **< 200 lines** per context file (longer = agent ignores rules)
- Each line: "Would removing this cause mistakes?" If not, cut it.

### Specificity Rules
- ✅ "Use 2-space indentation for JS/TS"
- ❌ "Format code properly"
- ✅ "Run `npm test` before committing"
- ❌ "Test your changes"

---

## Path-Scoped Rules (Most Underused Feature)

```
.claude/rules/
├── frontend.md      # Only loads when editing frontend files
├── api.md           # Only loads when editing API files
├── database.md      # Only loads when editing DB files
└── testing.md       # Only loads when editing test files
```

**#1 optimization for large projects** — keeps context window lean.

---

## Cross-Agent Compatibility

```markdown
# AGENTS.md (portable — works everywhere)
## Build
- `make test` — run tests

# CLAUDE.md (imports AGENTS.md, adds Claude-specific)
@AGENTS.md
## Claude-Specific
- Use plan mode for changes under src/billing/
```

---

## Installing This Skill

```bash
hermes skills install zufarrizal/new-project-ai-context
```

Or just tell your AI agent: "Setup AI context for this project"

---

## Pitfalls

1. **No SOUL.md = inconsistent agent personality** — Always create SOUL.md first
2. **Bloated context files REDUCE adherence** — Keep < 200 lines
3. **Contradicting rules cause random behavior** — Review periodically
4. **"Write clean code" is wasted space** — Self-evident instructions don't change behavior
5. **SOUL.md di wrong location** — Must be global (~/.hermes/SOUL.md), not project-level
6. **SOUL.md philosophy vs project rules conflict** — Ensure alignment
7. **Path-scoped rules are on-demand** — Don't put critical global rules there
8. **`/compact` can lose context** — Important rules should be in context files, not conversation
9. **AGENTS.md only loads from cwd** — Root placement matters
10. **Hermes .hermes.md walks parents** — Stops at git root boundary
