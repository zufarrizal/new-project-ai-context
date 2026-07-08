---
name: project-context-setup
description: "Set up AI context files for new projects across all coding agents (Hermes, Claude Code, Cursor, Codex). Maximizes AI output quality through structured context. Includes PILAR 1: SOUL.md, PILAR 2: MCP Servers, PILAR 3: Hooks, PILAR 4: Skills."
version: 1.4.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [ai-context, project-setup, claude-md, agents-md, cursorrules, soul-md, mcp-servers, hooks, skills, coding-agents]
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

## PILAR 2: MCP Servers — External Tools

MCP (Model Context Protocol) Servers memberi agent akses ke tools external. Tanpa MCP, agent hanya bisa baca/tulis file dan jalankan shell commands. Dengan MCP, agent bisa **query database, kelola GitHub, test browser, akses API** — dan **verifikasi sendiri hasilnya**.

### Kenapa MCP Servers Penting?

```
┌─────────────────────────────────────────────────────────────┐
│                    WITHOUT MCP                              │
├─────────────────────────────────────────────────────────────┤
│  User: "Check if the migration works"                       │
│  Agent: "I can't access the database directly.              │
│          Can you run this query and tell me the result?"    │
│  User: *runs query manually, pastes result*                 │
│  Agent: "Based on that, it looks like..."                   │
│  → Agent jadi useless tanpa user intervention               │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                    WITH MCP                                 │
├─────────────────────────────────────────────────────────────┤
│  User: "Check if the migration works"                       │
│  Agent: *queries database directly*                         │
│  Agent: "Migration successful. 3 tables created,            │
│          152 rows migrated. No errors."                     │
│  → Agent bisa verify sendiri, user tinggal review           │
└─────────────────────────────────────────────────────────────┘
```

### MCP Servers by Use Case

| Use Case | MCP Server | What Agent Can Do |
|----------|------------|-------------------|
| **Database** | PostgreSQL, MySQL, SQLite | Query data, check migrations, verify schemas |
| **GitHub** | @modelcontextprotocol/server-github | Manage issues, PRs, search code, create releases |
| **Browser Testing** | @anthropic-ai/server-puppeteer | Screenshot pages, test UI, check rendering |
| **File System** | @modelcontextprotocol/server-filesystem | Controlled file access with boundaries |
| **Web Fetch** | @anthropic-ai/server-fetch | Fetch APIs, test endpoints, check responses |
| **Memory/Knowledge** | @modelcontextprotocol/server-memory | Persistent knowledge graph across sessions |
| **Slack** | @modelcontextprotocol/server-slack | Send messages, read channels, manage workspace |
| **Google Drive** | @anthropic-ai/server-gdrive | Search files, read docs, manage folders |
| **Notion** | @anthropic-ai/server-notion | Query databases, create pages, update records |
| **Custom API** | Build your own MCP server | Any internal API, monitoring, proprietary tools |

### Setup: Claude Code

```bash
# Database — PostgreSQL
claude mcp add postgres -- npx @anthropic-ai/server-postgres \
  --connection-string postgresql://user:pass@localhost:5432/mydb

# GitHub
claude mcp add github -- npx @modelcontextprotocol/server-github

# Browser Testing
claude mcp add puppeteer -- npx @anthropic-ai/server-puppeteer

# Web Fetching
claude mcp add fetch -- npx @anthropic-ai/server-fetch

# File System (with boundaries)
claude mcp add filesystem -- npx @modelcontextprotocol/server-filesystem \
  /path/to/allowed/dir

# Memory/Knowledge Graph
claude mcp add memory -- npx @modelcontextprotocol/server-memory

# Custom MCP (your own server)
claude mcp add myapi -- npx my-mcp-server --config ./mcp-config.json
```

### Setup: Hermes

```bash
# Add MCP server
hermes mcp add postgres --url postgresql://localhost:5432/mydb
hermes mcp add github --command "npx @modelcontextprotocol/server-github"

# List configured servers
hermes mcp list

# Test connection
hermes mcp test postgres

# Configure which tools to expose
hermes mcp configure postgres
```

### MCP Scopes (Claude Code)

| Flag | Scope | Storage | When to Use |
|------|-------|---------|-------------|
| `-s user` | Global (all projects) | `~/.claude.json` | Tools you always need (GitHub, Memory) |
| `-s local` | This project (personal) | `.claude/settings.local.json` | Your dev DB, personal tools |
| `-s project` | This project (team-shared) | `.claude/settings.json` | Team DB, shared CI tools |

### Auto-Detect: Which MCP Servers Does This Project Need?

```python
needs = []

if has_file("docker-compose.yml") and grep("postgres", "docker-compose.yml"):
    needs.append("postgresql")

if has_file(".github/"):
    needs.append("github")

if has_file("playwright.config.ts") or has_file("cypress.config.ts"):
    needs.append("puppeteer")

if has_file("requirements.txt") and grep("requests", "requirements.txt"):
    needs.append("fetch")
```

### MCP Limits & Tuning

| Setting | Default | Recommendation |
|---------|---------|----------------|
| Tool descriptions | 2KB cap per server | Keep descriptions concise |
| Result size | Capped | Use `maxResultSizeChars` annotation for large output |
| Output tokens | Varies | `export MAX_MCP_OUTPUT_TOKENS=50000` to cap |

---

## PILAR 3: Hooks — Automation on Events

Hooks adalah **otomatisasi yang berjalan tanpa user intervention**. Agent punya "guard rails" dan "auto-pilot" yang memastikan kualitas kode secara konsisten.

### Kenapa Hooks Penting?

```
┌─────────────────────────────────────────────────────────────┐
│                    WITHOUT HOOKS                            │
├─────────────────────────────────────────────────────────────┤
│  Agent: *writes code*                                       │
│  User: "Run the linter"                                     │
│  Agent: *runs linter, finds issues*                         │
│  Agent: *fixes issues*                                      │
│  User: "Run it again"                                       │
│  Agent: *runs linter, passes*                               │
│  User: "Now run tests"                                      │
│  → User jadi babysitter, bukan reviewer                     │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                    WITH HOOKS                               │
├─────────────────────────────────────────────────────────────┤
│  Agent: *writes code*                                       │
│  Hook (PostToolUse): *auto-runs linter, fixes issues*       │
│  Hook (PostToolUse): *auto-runs tests*                      │
│  Agent: "Code written, linted, and tested. All passing."    │
│  User: *reviews final result*                               │
│  → User jadi reviewer, bukan babysitter                     │
└─────────────────────────────────────────────────────────────┘
```

### 8 Hook Types

| Hook | Kapan Fire | Contoh Penggunaan |
|------|-----------|-------------------|
| `UserPromptSubmit` | Sebelum proses prompt | Input validation, logging |
| `PreToolUse` | Sebelum tool jalan | Security gates, block dangerous commands |
| `PostToolUse` | Setelah tool selesai | Auto-format, run linters, auto-test |
| `Notification` | Saat permission request | Desktop notifications, Slack alerts |
| `Stop` | Saat agent selesai respond | Completion logging, auto-commit |
| `SubagentStop` | Saat subagent selesai | Agent orchestration |
| `PreCompact` | Sebelum context di-compress | Backup session |
| `SessionStart` | Saat session mulai | Load dev context |

### Setup: Claude Code

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write(*.py)",
        "hooks": [{"type": "command", "command": "ruff check --fix $CLAUDE_FILE_PATHS"}]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [{
          "type": "command",
          "command": "if echo \"$CLAUDE_TOOL_INPUT\" | grep -qE 'rm -rf|DROP TABLE'; then echo 'BLOCKED!' && exit 2; fi"
        }]
      }
    ],
    "Stop": [
      {
        "hooks": [{"type": "command", "command": "echo \"[$(date)] Task completed\" >> ~/.claude/activity.log"}]
      }
    ]
  }
}
```

### Auto-Detect: Which Hooks Does This Project Need?

```bash
# Python linters
grep -l "ruff\|black\|flake8\|mypy" pyproject.toml 2>/dev/null

# JS/TS linters
ls .eslintrc* .prettierrc* eslint.config.* 2>/dev/null

# Go
ls .golangci.yml 2>/dev/null

# Rust
ls rustfmt.toml clippy.toml 2>/dev/null
```

### Per-Language Hook Examples

**Python:** `ruff check --fix $CLAUDE_FILE_PATHS && mypy $CLAUDE_FILE_PATHS`
**TypeScript:** `npx prettier --write $CLAUDE_FILE_PATHS && npx eslint --fix $CLAUDE_FILE_PATHS`
**Go:** `gofmt -w $CLAUDE_FILE_PATHS && go vet ./...`
**Rust:** `cargo fmt -- $CLAUDE_FILE_PATHS && cargo clippy -- -D warnings`

### Exit Codes

| Exit | Meaning |
|------|---------|
| 0 | Hook passed, continue |
| 1 | Hook failed, log warning, continue |
| 2 | Hook BLOCKS the action, stop execution |

---

## PILAR 4: Skills — Reusable Procedures

Skills adalah **prosedur yang bisa dipakai ulang** tanpa harus dijelaskan setiap session. Agent tidak reinvent the wheel — langsung tahu workflow yang harus diikuti.

### Kenapa Skills Penting?

```
┌─────────────────────────────────────────────────────────────┐
│                    WITHOUT SKILLS                           │
├─────────────────────────────────────────────────────────────┤
│  User: "Add a new API endpoint"                             │
│  Agent: *looks at existing code*                            │
│  Agent: *guesses pattern*                                   │
│  Agent: *creates file in wrong location*                    │
│  User: "No, put it in src/api/handlers/"                    │
│  Agent: *moves file*                                        │
│  User: "Also add to the router"                             │
│  Agent: *forgets test*                                      │
│  User: "Where's the test?"                                  │
│  → 5 rounds of correction for a repeatable task             │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                    WITH SKILLS                              │
├─────────────────────────────────────────────────────────────┤
│  User: "Add a new API endpoint"                             │
│  Agent: *loads add-api-endpoint skill*                      │
│  Agent: *creates handler in src/api/handlers/*              │
│  Agent: *adds route in router.py*                           │
│  Agent: *writes test in tests/api/*                         │
│  Agent: *runs `make test`*                                  │
│  Agent: "Done. New endpoint added with test."               │
│  → Zero correction rounds                                   │
└─────────────────────────────────────────────────────────────┘
```

### Skills vs Context Files

| Aspect | Context Files (AGENTS.md) | Skills |
|--------|--------------------------|--------|
| **When loaded** | Every session, always | On-demand, when relevant |
| **Size** | < 200 lines (keep lean) | Can be detailed |
| **Scope** | Project-wide rules | Specific workflow/procedure |
| **Use case** | "How we code in general" | "How to do X step by step" |
| **Token cost** | Always paid | Only paid when used |

### Skills by Platform

| Platform | Lokasi | Trigger |
|----------|--------|---------|
| Hermes | `~/.hermes/skills/` | Auto-detected, `/skill name`, `-s name` |
| Claude Code | `.claude/skills/` (project) or `~/.claude/skills/` (global) | Natural language match |
| Cursor | `.cursor/rules/*.mdc` (path-scoped) | File type match |

### Skills Every Project Should Have

#### 1. Add API Endpoint

```markdown
# .claude/skills/add-api-endpoint.md

When asked to add a new API endpoint:

1. Check existing endpoints in src/api/handlers/ for patterns
2. Create handler file: src/api/handlers/{resource}.py
3. Define route with proper HTTP method (GET/POST/PUT/DELETE)
4. Add request/response models in src/api/models/
5. Register route in src/api/router.py
6. Write unit test in tests/api/test_{resource}.py
7. Run `make test` to verify
8. Update API docs if they exist

## Patterns
- Use dependency injection for database sessions
- Return proper HTTP status codes (201 for create, 204 for delete)
- Always validate input with Pydantic models
- Use async handlers for I/O operations
```

#### 2. Database Migration

```markdown
# .claude/skills/database-migration.md

When asked to create or modify database migrations:

1. Check current schema in src/models/
2. Generate migration: `alembic revision --autogenerate -m "description"`
3. Review generated migration file in alembic/versions/
4. ALWAYS create a rollback: add `downgrade()` function
5. Test migration: `alembic upgrade head`
6. Test rollback: `alembic downgrade -1`
7. Test forward again: `alembic upgrade head`

## Rules
- Never modify existing migration files that are already committed
- Always include data migration when changing column types
- Add `op.execute()` for complex data transformations
- Name migrations descriptively: `add_user_avatar_column`
```

#### 3. Add New Feature

```markdown
# .claude/skills/add-feature.md

When asked to add a new feature:

## Phase 1: Explore
1. Understand existing code patterns (read similar features)
2. Identify affected files and dependencies
3. Check if feature needs database changes

## Phase 2: Plan
4. Create a branch: `git checkout -b feat/{feature-name}`
5. List files to create/modify

## Phase 3: Implement
6. Write code following existing patterns
7. Add tests (unit + integration)
8. Run `make test` and `make lint`

## Phase 4: Verify
9. Manual testing if UI changes
10. Check edge cases
11. Commit with descriptive message: `feat: {description}`

## Rules
- Follow existing code patterns, don't introduce new ones
- Write tests BEFORE or ALONGSIDE code, not after
- If feature touches 3+ files, create a plan first
```

#### 4. Debug Issue

```markdown
# .claude/skills/debug-issue.md

When asked to debug an issue:

## Phase 1: Reproduce
1. Understand the exact error message and stack trace
2. Try to reproduce the issue locally
3. Check if it's consistent or intermittent

## Phase 2: Investigate
4. Add logging at key points
5. Check recent changes: `git log --oneline -10`
6. Check related files for obvious issues
7. Use debugger if available

## Phase 3: Fix
8. Identify root cause (not just symptom)
9. Write a failing test that reproduces the issue
10. Fix the code
11. Verify test now passes

## Phase 4: Verify
12. Run full test suite
13. Check for similar issues elsewhere
14. Commit: `fix: {description}`

## Rules
- Never suppress errors without understanding them
- Always write a test that reproduces the bug first
- Fix root cause, not symptom
```

#### 5. Code Review

```markdown
# .claude/skills/code-review.md

When asked to review code:

## Checklist
1. **Correctness**: Does it do what it's supposed to?
2. **Edge cases**: What about null, empty, overflow, boundary?
3. **Security**: Any injection, auth bypass, data leak?
4. **Performance**: N+1 queries, unnecessary loops, large allocations?
5. **Tests**: Are there tests? Do they cover edge cases?
6. **Style**: Does it follow project conventions?
7. **Error handling**: Are errors handled properly?
8. **Documentation**: Are complex parts documented?

## Output Format
For each issue found:
- **File**: path/to/file.py:42
- **Severity**: critical/warning/info
- **Issue**: Description
- **Suggestion**: How to fix

## Rules
- Be specific, not vague ("This will cause X because Y")
- Suggest fixes, not just problems
- Praise good patterns when you see them
```

#### 6. Deploy

```markdown
# .claude/skills/deploy.md

When asked to deploy:

## Pre-Deploy
1. Run full test suite: `make test`
2. Run linter: `make lint`
3. Check for uncommitted changes: `git status`
4. Verify on correct branch: `git branch`

## Deploy Steps
5. Build: `make build`
6. Push to registry: `make push`
7. Deploy to staging: `make deploy-staging`
8. Run smoke tests on staging
9. Deploy to production: `make deploy-prod`

## Post-Deploy
10. Monitor logs for 5 minutes
11. Check error rates in monitoring dashboard
12. Tag release: `git tag v{version}`

## Rules
- NEVER deploy directly to production
- Always deploy to staging first
- If smoke tests fail, rollback immediately
- Notify team after successful deploy
```

### Auto-Detect: Which Skills Does This Project Need?

```python
skills_needed = []

# API project
if has_file("src/api/") or has_file("routes/"):
    skills_needed.append("add-api-endpoint")

# Has database
if has_file("alembic/") or has_file("migrations/"):
    skills_needed.append("database-migration")

# Has CI/CD
if has_file(".github/workflows/") or has_file("Jenkinsfile"):
    skills_needed.append("deploy")

# Always useful
skills_needed.append("add-feature")
skills_needed.append("debug-issue")
skills_needed.append("code-review")
```

### Creating Skills: Best Practices

```markdown
# Skill Structure

## Trigger Condition
"When asked to..." or "When working with..."

## Steps (numbered)
1. First step
2. Second step
3. ...

## Rules (bullet points)
- Rule 1
- Rule 2

## Examples (optional)
- Input → Output
```

### Skill Size Guidelines

| Size | When to Use |
|------|-------------|
| < 50 lines | Simple workflow (add endpoint, write test) |
| 50-150 lines | Medium workflow (database migration, deploy) |
| 150-300 lines | Complex workflow (full feature, architecture change) |
| > 300 lines | Split into multiple skills |

### Skill Loading Strategies

**Hermes:**
```bash
# Auto-load at session start
hermes -s add-api-endpoint,debug-issue

# Load during session
/skill add-api-endpoint

# In CLAUDE.md or AGENTS.md
When working on API tasks, load the add-api-endpoint skill.
```

**Claude Code:**
```markdown
# .claude/skills/ directory — loaded on demand via natural language
# "Add a new API endpoint" → automatically loads add-api-endpoint.md
```

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

## Testing
- Test framework: {pytest, jest, vitest, etc.}
- Run single test: `{command}`
- Coverage target: {percentage}

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
```

### .hermes.md (Hermes-Specific)

```markdown
# {PROJECT_NAME}

Hermes: when working in this repo, follow these rules.

## Build
- {build commands}

## Style
- {code style rules}

## Workflow
- {verification steps}
```

---

## Setup Procedure

### Step 0: SOUL.md (PILAR 1)

```bash
cat ~/.hermes/SOUL.md 2>/dev/null || echo "NEED TO CREATE SOUL.md"
```

### Step 1: Detect Project Type

```bash
ls package.json pyproject.toml Cargo.toml go.mod Makefile Dockerfile 2>/dev/null
```

### Step 2: Detect Existing Context Files

```bash
ls CLAUDE.md AGENTS.md .hermes.md .cursorrules .claude/ .cursor/ 2>/dev/null
```

### Step 3: Setup MCP Servers (PILAR 2)

```bash
grep -l "postgres\|mysql\|mongo" docker-compose.yml 2>/dev/null
ls .github/ playwright.config.* cypress.config.* 2>/dev/null
```

### Step 4: Setup Hooks (PILAR 3)

```bash
grep -l "ruff\|black\|flake8\|mypy" pyproject.toml 2>/dev/null
ls .eslintrc* .prettierrc* eslint.config.* 2>/dev/null
```

### Step 5: Create Skills (PILAR 4)

```bash
# Check for API structure
ls src/api/ routes/ app/api/ 2>/dev/null

# Check for database migrations
ls alembic/ migrations/ prisma/ 2>/dev/null

# Check for CI/CD
ls .github/workflows/ Jenkinsfile Makefile 2>/dev/null
```

Create skills based on detected project structure.

### Step 6: Generate Context Files

Fill templates with REAL data. Order: AGENTS.md → CLAUDE.md → .hermes.md

### Step 7: Verify

- Files under 200 lines each
- Build/test commands work
- MCP servers connect
- Hooks execute
- Skills load correctly

---

## How 4 PILAR Bekerja Bersama

```
┌─────────────────────────────────────────────────────────────┐
│  Agent + 4 PILAR = FULLY AUTONOMOUS                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  PILAR 1: SOUL.md        → Agent TAHU siapa dirinya        │
│  PILAR 2: MCP Servers    → Agent BISA aksi langsung         │
│  PILAR 3: Hooks          → Agent AUTO-jaga kualitas         │
│  PILAR 4: Skills         → Agent TAHU cara kerja            │
│                                                             │
│  Context Files (AGENTS.md, CLAUDE.md)                       │
│            → Agent TAHU projectnya                          │
│                                                             │
│  Result:                                                    │
│  ┌─────────────────────────────────────────────┐            │
│  │ User: "Add new API endpoint"                 │            │
│  │ Agent: *loads skill (PILAR 4)*               │            │
│  │ Agent: *follows SOUL.md style (PILAR 1)*     │            │
│  │ Agent: *codes based on AGENTS.md rules*      │            │
│  │ Hook: *auto-lints (PILAR 3)*                 │            │
│  │ Agent: *tests via MCP (PILAR 2)*             │            │
│  │ Hook: *auto-runs tests (PILAR 3)*            │            │
│  │ Agent: "Done. Endpoint added, tested, PR."   │            │
│  │ User: *reviews PR*                           │            │
│  └─────────────────────────────────────────────┘            │
│                                                             │
│  User = reviewer, bukan babysitter                          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## What Makes Context IMPACTFUL

### Tier 1 — Highest Impact
1. **SOUL.md (PILAR 1)** — Agent identity
2. **MCP Servers (PILAR 2)** — Agent can verify
3. **Hooks (PILAR 3)** — Auto-quality
4. **Skills (PILAR 4)** — Reusable procedures
5. **Build/test commands**
6. **Code style rules that DIFFER from defaults**

### Tier 2 — High Impact
7. **Repo etiquette** — branch naming, PR conventions
8. **Testing preferences**
9. **Environment quirks**

### NEVER Include
- ✗ Standard language conventions
- ✗ Detailed API docs (link instead)
- ✗ Self-evident practices

---

## Installing This Skill

```bash
hermes skills install zufarrizal/new-project-ai-context
```

---

## Pitfalls

1. **No SOUL.md = inconsistent personality**
2. **No MCP = agent can't verify**
3. **No hooks = user jadi babysitter**
4. **No skills = repeatable tasks need re-explaining**
5. **Skills too large = split into smaller skills**
6. **Skills in context files = wasted tokens — use on-demand loading**
7. **Bloated context files REDUCE adherence** — Keep < 200 lines
8. **Contradicting rules confuse agent**
9. **SOUL.md di wrong location** — Must be global
10. **`/compact` can lose context**
