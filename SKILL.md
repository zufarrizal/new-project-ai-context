---
name: project-context-setup
description: "Set up AI context files for new projects across all coding agents (Hermes, Claude Code, Cursor, Codex). Maximizes AI output quality through structured context. Includes PILAR 1: SOUL.md, PILAR 2: MCP Servers, PILAR 3: Hooks."
version: 1.3.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [ai-context, project-setup, claude-md, agents-md, cursorrules, soul-md, mcp-servers, hooks, coding-agents]
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
# Detection logic based on project files
needs = []

if has_file("docker-compose.yml") and grep("postgres", "docker-compose.yml"):
    needs.append("postgresql")

if has_file(".github/"):
    needs.append("github")

if has_file("playwright.config.ts") or has_file("cypress.config.ts"):
    needs.append("puppeteer")

if has_file("requirements.txt") and grep("requests", "requirements.txt"):
    needs.append("fetch")  # API testing

if has_file("package.json") and grep("next", "package.json"):
    needs.append("puppeteer")  # Next.js usually needs browser testing
```

### MCP in Print/CI Mode

```bash
# Bare mode with specific MCP config
claude --bare -p 'Query database and verify migration' \
  --mcp-config mcp-servers.json \
  --strict-mcp-config
```

`--strict-mcp-config` = HANYA gunakan MCP dari config file, ignore yang lain.

### MCP Limits & Tuning

| Setting | Default | Recommendation |
|---------|---------|----------------|
| Tool descriptions | 2KB cap per server | Keep descriptions concise |
| Result size | Capped | Use `maxResultSizeChars` annotation for large output |
| Output tokens | Varies | `export MAX_MCP_OUTPUT_TOKENS=50000` to cap |

### Custom MCP Server Template

Untuk API internal atau proprietary tools, buat MCP server sendiri:

```javascript
// my-mcp-server.js
const { Server } = require("@modelcontextprotocol/sdk/server/index.js");
const { StdioServerTransport } = require("@modelcontextprotocol/sdk/server/stdio.js");

const server = new Server({ name: "my-api", version: "1.0.0" });

server.setRequestHandler("tools/list", async () => ({
  tools: [{
    name: "get_user",
    description: "Get user by ID from internal API",
    inputSchema: {
      type: "object",
      properties: { user_id: { type: "string" } },
      required: ["user_id"]
    }
  }]
}));

server.setRequestHandler("tools/call", async (request) => {
  if (request.params.name === "get_user") {
    const res = await fetch(`https://internal-api/users/${request.params.arguments.user_id}`);
    return { content: [{ type: "text", text: await res.text() }] };
  }
});

const transport = new StdioServerTransport();
server.connect(transport);
```

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
│  Agent: *runs tests, finds failures*                        │
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
| `UserPromptSubmit` | Sebelum proses prompt | Input validation, logging, context injection |
| `PreToolUse` | Sebelum tool jalan | Security gates, block dangerous commands |
| `PostToolUse` | Setelah tool selesai | Auto-format, run linters, auto-test |
| `Notification` | Saat permission request | Desktop notifications, Slack alerts |
| `Stop` | Saat agent selesai respond | Completion logging, auto-commit |
| `SubagentStop` | Saat subagent selesai | Agent orchestration, result aggregation |
| `PreCompact` | Sebelum context di-compress | Backup session, save state |
| `SessionStart` | Saat session mulai | Load dev context, check environment |

### Setup: Claude Code

Hooks dikonfigurasi di `.claude/settings.json` (project) atau `~/.claude/settings.json` (global).

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write(*.py)",
        "hooks": [
          {"type": "command", "command": "ruff check --fix $CLAUDE_FILE_PATHS"}
        ]
      },
      {
        "matcher": "Write(*.ts)",
        "hooks": [
          {"type": "command", "command": "npx prettier --write $CLAUDE_FILE_PATHS"}
        ]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "if echo \"$CLAUDE_TOOL_INPUT\" | grep -qE 'rm -rf|git push.*--force|DROP TABLE'; then echo 'BLOCKED: Dangerous command!' && exit 2; fi"
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {"type": "command", "command": "echo \"[$(date)] Task completed\" >> ~/.claude/activity.log"}
        ]
      }
    ],
    "Notification": [
      {
        "hooks": [
          {"type": "command", "command": "notify-send 'Claude Code' 'Waiting for your input'"}
        ]
      }
    ]
  }
}
```

### Setup: Hermes

Hooks di Hermes menggunakan config.yaml:

```yaml
# ~/.hermes/config.yaml
hooks:
  post_tool_use:
    - matcher: "write_file(*.py)"
      command: "ruff check --fix {file_path}"
    - matcher: "write_file(*.ts)"
      command: "npx prettier --write {file_path}"
  pre_tool_use:
    - matcher: "terminal"
      command: "echo '{input}' | grep -qE 'rm -rf|DROP TABLE' && exit 1 || exit 0"
  stop:
    - command: "echo '[$(date)] Task completed' >> ~/.hermes/activity.log"
```

### Hook Environment Variables

| Variable | Content |
|----------|---------|
| `CLAUDE_PROJECT_DIR` | Current project path |
| `CLAUDE_FILE_PATHS` | Files being modified |
| `CLAUDE_TOOL_INPUT` | Tool parameters as JSON |

### Auto-Detect: Which Hooks Does This Project Need?

```python
# Detection logic
hooks_needed = []

# Linting
if has_file("pyproject.toml") and grep("ruff", "pyproject.toml"):
    hooks_needed.append({"matcher": "Write(*.py)", "command": "ruff check --fix {files}"})

if has_file(".eslintrc") or has_file("eslint.config.js"):
    hooks_needed.append({"matcher": "Write(*.ts)", "command": "npx eslint --fix {files}"})

if has_file(".prettierrc"):
    hooks_needed.append({"matcher": "Write(*.ts)", "command": "npx prettier --write {files}"})

# Testing
if has_file("pytest.ini") or has_file("pyproject.toml") and grep("pytest", "pyproject.toml"):
    hooks_needed.append({"matcher": "Write(*.py)", "command": "pytest --tb=short -q"})

if has_file("jest.config.js") or has_file("vitest.config.ts"):
    hooks_needed.append({"matcher": "Write(*.test.ts)", "command": "npx vitest run {file}"})

# Security
if has_file(".env"):
    hooks_needed.append({"matcher": "Bash", "command": "grep -q 'SECRET\\|KEY\\|PASSWORD' {input} && exit 1 || exit 0"})
```

### Security Hook Patterns

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "echo \"$CLAUDE_TOOL_INPUT\" | python3 -c \"import sys,json; cmd=json.load(sys.stdin).get('command',''); dangerous=['rm -rf','git push --force','chmod 777','curl|sh','wget|sh','DROP TABLE','DELETE FROM']; [print('BLOCKED:',d) or sys.exit(2) for d in dangerous if d in cmd]\""
          }
        ]
      }
    ]
  }
}
```

### Hook Execution Rules

| Rule | Detail |
|------|--------|
| Exit 0 | Hook passed, continue |
| Exit 1 | Hook failed, log warning, continue |
| Exit 2 | Hook BLOCKS the action, stop execution |
| Timeout | 60 seconds default, hook killed after |
| Output | stdout/stderr shown to agent |

### Hook Examples by Project Type

**Python Project:**
```json
{
  "PostToolUse": [
    {"matcher": "Write(*.py)", "hooks": [{"type": "command", "command": "ruff check --fix $CLAUDE_FILE_PATHS && mypy $CLAUDE_FILE_PATHS"}]}
  ]
}
```

**TypeScript Project:**
```json
{
  "PostToolUse": [
    {"matcher": "Write(*.ts)", "hooks": [{"type": "command", "command": "npx prettier --write $CLAUDE_FILE_PATHS && npx eslint --fix $CLAUDE_FILE_PATHS"}]}
  ]
}
```

**Go Project:**
```json
{
  "PostToolUse": [
    {"matcher": "Write(*.go)", "hooks": [{"type": "command", "command": "gofmt -w $CLAUDE_FILE_PATHS && go vet ./..."}]}
  ]
}
```

**Rust Project:**
```json
{
  "PostToolUse": [
    {"matcher": "Write(*.rs)", "hooks": [{"type": "command", "command": "cargo fmt -- $CLAUDE_FILE_PATHS && cargo clippy -- -D warnings"}]}
  ]
}
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
cat ~/.hermes/SOUL.md 2>/dev/null || echo "NEED TO CREATE SOUL.md"
```

Jika belum ada, deteksi tech stack lalu buat SOUL.md dari template di atas.

### Step 1: Detect Project Type

```bash
ls package.json pyproject.toml Cargo.toml go.mod Makefile Dockerfile 2>/dev/null
cat package.json | python3 -c "import json,sys; d=json.load(sys.stdin); print(d.get('name',''), d.get('scripts',{}))"
```

### Step 2: Detect Existing Context Files

```bash
ls CLAUDE.md AGENTS.md .hermes.md .cursorrules .claude/ .cursor/ 2>/dev/null
```

### Step 3: Setup MCP Servers (PILAR 2)

```bash
# Check docker-compose for databases
grep -l "postgres\|mysql\|mongo" docker-compose.yml 2>/dev/null

# Check for GitHub Actions
ls .github/ 2>/dev/null

# Check for browser testing
ls playwright.config.* cypress.config.* 2>/dev/null
```

Setup MCP yang relevan.

### Step 4: Setup Hooks (PILAR 3)

Deteksi linter/formatter dari project:

```bash
# Python
grep -l "ruff\|black\|flake8\|mypy" pyproject.toml setup.cfg .flake8 2>/dev/null

# JavaScript/TypeScript
ls .eslintrc* .prettierrc* eslint.config.* prettier.config.* 2>/dev/null

# Go
ls .golangci.yml 2>/dev/null

# Rust
ls rustfmt.toml clippy.toml 2>/dev/null
```

Generate `.claude/settings.json` dengan hooks yang relevan.

### Step 5: Generate Context Files

Fill templates with REAL data from project. Order: AGENTS.md → CLAUDE.md → .hermes.md

### Step 6: Verify

- Files under 200 lines each
- Build/test commands actually work
- MCP servers connect successfully
- Hooks execute without errors
- No contradictions between files

---

## How 3 PILAR Bekerja Bersama

```
┌─────────────────────────────────────────────────────────────┐
│  Agent + 3 PILAR = FULLY AUTONOMOUS                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  PILAR 1: SOUL.md        → Agent TAHU siapa dirinya        │
│  PILAR 2: MCP Servers    → Agent BISA aksi langsung         │
│  PILAR 3: Hooks          → Agent AUTO-jaga kualitas         │
│                                                             │
│  Context Files (AGENTS.md, CLAUDE.md)                       │
│            → Agent TAHU projectnya                          │
│                                                             │
│  Result:                                                    │
│  ┌─────────────────────────────────────────────┐            │
│  │ User: "Build feature X"                      │            │
│  │ Agent: *plans based on SOUL.md personality*  │            │
│  │ Agent: *codes based on AGENTS.md rules*      │            │
│  │ Hook: *auto-lints, auto-formats*             │            │
│  │ Agent: *tests via MCP database*              │            │
│  │ Hook: *auto-runs test suite*                 │            │
│  │ Agent: "Done. All tests passing. PR ready."  │            │
│  │ User: *reviews PR*                           │            │
│  └─────────────────────────────────────────────┘            │
│                                                             │
│  User = reviewer, bukan babysitter                          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## What Makes Context IMPACTFUL

### Tier 1 — Highest Impact (always include)
1. **SOUL.md (PILAR 1)** — Agent identity foundation
2. **MCP Servers (PILAR 2)** — Agent can verify its own work
3. **Hooks (PILAR 3)** — Auto-quality enforcement
4. **Build/test commands** — Agent can't guess these
5. **Verification commands** — Enables self-correcting loops
6. **Code style rules that DIFFER from defaults**

### Tier 2 — High Impact (include when relevant)
7. **Repo etiquette** — branch naming, PR conventions
8. **Testing preferences** — test runner, coverage targets
9. **Environment quirks** — required env vars

### NEVER Include
- ✗ Standard language conventions
- ✗ Detailed API documentation (link instead)
- ✗ Self-evident practices ("write clean code")

---

## Installing This Skill

```bash
hermes skills install zufarrizal/new-project-ai-context
```

Or just tell your AI agent: "Setup AI context for this project"

---

## Pitfalls

1. **No SOUL.md = inconsistent personality** — Always create SOUL.md first
2. **No MCP = agent can't verify** — Setup relevant MCP servers
3. **No hooks = user jadi babysitter** — Setup auto-quality hooks
4. **Exit 2 in PreToolUse blocks action** — Use carefully
5. **Hooks timeout = 60s** — Long-running hooks get killed
6. **Bloated context files REDUCE adherence** — Keep < 200 lines
7. **Contradicting rules confuse agent** — Review periodically
8. **SOUL.md di wrong location** — Must be global, not project-level
9. **Path-scoped rules are on-demand** — Don't put critical global rules there
10. **`/compact` can lose context** — Important rules in context files, not conversation
