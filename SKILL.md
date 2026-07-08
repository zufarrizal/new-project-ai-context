---
name: project-context-setup
description: "Set up AI context files for new projects across all coding agents (Hermes, Claude Code, Cursor, Codex). Maximizes AI output quality through structured context. Includes 5 PILAR: SOUL.md, MCP Servers, Hooks, Skills, Goal-Oriented Prompting."
version: 1.5.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [ai-context, project-setup, claude-md, agents-md, cursorrules, soul-md, mcp-servers, hooks, skills, goal-prompting, coding-agents]
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

### Template: SOUL.md

```markdown
# Agent Identity

You are a {senior_role} specializing in {domain}.

## Expertise
- Primary: {main_tech_stack}
- Secondary: {supporting_tech}

## Coding Philosophy
- {principle_1}
- {principle_2}
- {principle_3}

## Behavior Rules
- Always explain trade-offs before making architectural decisions
- Prefer {language/framework} over {alternative} unless explicitly asked
- Never use {anti_patterns} unless the user insists
- When uncertain, ask — don't guess on critical decisions

## Communication Style
- {concise/detailed} responses
- {direct/diplomatic} when pointing out issues
- Show code first, explain after

## Quality Standards
- Every function must have {type hints/typed parameters}
- Tests are not optional — {coverage_target}% minimum
- No silent failures — always {log/throw} on errors
```

### Lokasi SOUL.md

| Platform | Lokasi | Scope |
|----------|--------|-------|
| Hermes | `~/.hermes/SOUL.md` | Global |
| Claude Code | `~/.claude/CLAUDE.md` (global) | Global |
| Cursor | `~/.cursor/rules/identity.mdc` | Global |

---

## PILAR 2: MCP Servers — External Tools

MCP Servers memberi agent akses ke tools external sehingga bisa **verifikasi sendiri hasilnya**.

### MCP Servers by Use Case

| Use Case | MCP Server | What Agent Can Do |
|----------|------------|-------------------|
| **Database** | PostgreSQL, MySQL, SQLite | Query data, verify schemas |
| **GitHub** | @modelcontextprotocol/server-github | Issues, PRs, code search |
| **Browser** | @anthropic-ai/server-puppeteer | Screenshot, test UI |
| **Web Fetch** | @anthropic-ai/server-fetch | Test APIs, check responses |
| **File System** | @modelcontextprotocol/server-filesystem | Controlled file access |
| **Memory** | @modelcontextprotocol/server-memory | Knowledge graph |

### Setup

```bash
# Claude Code
claude mcp add postgres -- npx @anthropic-ai/server-postgres --connection-string postgresql://localhost/mydb
claude mcp add github -- npx @modelcontextprotocol/server-github
claude mcp add puppeteer -- npx @anthropic-ai/server-puppeteer

# Hermes
hermes mcp add postgres --url postgresql://localhost:5432/mydb
hermes mcp list
hermes mcp test postgres
```

---

## PILAR 3: Hooks — Automation on Events

Hooks = **auto-quality enforcement** tanpa user intervention.

### 8 Hook Types

| Hook | Kapan Fire | Contoh |
|------|-----------|--------|
| `UserPromptSubmit` | Sebelum proses prompt | Input validation |
| `PreToolUse` | Sebelum tool jalan | Block dangerous commands |
| `PostToolUse` | Setelah tool selesai | Auto-format, auto-lint |
| `Notification` | Saat permission request | Desktop alerts |
| `Stop` | Saat selesai respond | Auto-commit, logging |
| `SubagentStop` | Saat subagent selesai | Orchestration |
| `PreCompact` | Sebelum compress | Backup session |
| `SessionStart` | Saat session mulai | Load dev context |

### Setup

```json
{
  "hooks": {
    "PostToolUse": [
      {"matcher": "Write(*.py)", "hooks": [{"type": "command", "command": "ruff check --fix $CLAUDE_FILE_PATHS"}]}
    ],
    "PreToolUse": [
      {"matcher": "Bash", "hooks": [{"type": "command", "command": "if echo \"$CLAUDE_TOOL_INPUT\" | grep -qE 'rm -rf|DROP TABLE'; then echo 'BLOCKED!' && exit 2; fi"}]}
    ]
  }
}
```

### Exit Codes

| Exit | Meaning |
|------|---------|
| 0 | Pass, continue |
| 1 | Warn, continue |
| 2 | BLOCK, stop execution |

---

## PILAR 4: Skills — Reusable Procedures

Skills = **prosedur yang dipakai ulang** tanpa re-explain setiap session.

### Skills vs Context Files

| Aspect | Context Files | Skills |
|--------|--------------|--------|
| When loaded | Every session | On-demand |
| Size | < 200 lines | Can be detailed |
| Scope | Project-wide rules | Specific workflow |

### 6 Essential Project Skills

1. **add-api-endpoint** — Standard workflow untuk tambah endpoint
2. **database-migration** — Cara buat/test/rollback migration
3. **add-feature** — Full feature development workflow
4. **debug-issue** — Systematic debugging procedure
5. **code-review** — Review checklist & output format
6. **deploy** — Pre-deploy, deploy, post-deploy steps

### Creating Skills

```markdown
# .claude/skills/{skill-name}.md

## Trigger
"When asked to..."

## Steps
1. First step
2. Second step
3. ...

## Rules
- Rule 1
- Rule 2
```

---

## PILAR 5: Goal-Oriented Prompting — Clear Success Criteria

Goal-Oriented Prompting adalah cara memberi task ke agent dengan **clear success criteria** sehingga agent bisa **self-correct** tanpa user intervention.

### Kenapa Goal-Oriented Prompting Penting?

```
┌─────────────────────────────────────────────────────────────┐
│                    VAGUE PROMPT                             │
├─────────────────────────────────────────────────────────────┤
│  User: "Fix the login bug"                                  │
│  Agent: *looks at login code*                               │
│  Agent: *adds try-catch around everything*                  │
│  Agent: "Fixed!"                                            │
│  User: "That's not a fix, that's just hiding the error"     │
│  Agent: *adds more try-catch*                               │
│  User: "No, find the ROOT CAUSE"                            │
│  Agent: *finally investigates properly*                     │
│  → 3 rounds of correction karena tidak ada success criteria │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                    GOAL-ORIENTED PROMPT                     │
├─────────────────────────────────────────────────────────────┤
│  User: "Users report login fails after session timeout.     │
│         Check src/auth/, especially token refresh.          │
│         Write a failing test that reproduces the issue,     │
│         then fix it. Verify: run `make test` and ensure     │
│         all auth tests pass."                               │
│                                                             │
│  Agent: *reads src/auth/token_refresh.py*                   │
│  Agent: *finds race condition in token refresh*             │
│  Agent: *writes failing test*                               │
│  Agent: *fixes the race condition*                          │
│  Agent: *runs `make test` — all pass*                       │
│  Agent: "Fixed. Root cause was race condition in            │
│          token refresh. Test added. All 47 auth tests pass."│
│  → Zero correction rounds                                   │
└─────────────────────────────────────────────────────────────┘
```

### The Formula

```
┌─────────────────────────────────────────────────────────────┐
│  GOAL = SITUATION + WHAT + WHERE + VERIFY                   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  SITUATION  → Context/masalah yang terjadi                  │
│  WHAT       → Apa yang harus dilakukan                      │
│  WHERE      → Di mana harus look/code                       │
│  VERIFY     → Bagaimana cara tahu sudah selesai             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Before/After Examples

#### Bug Fix

```
❌ Vague: "Fix the auth bug"

✅ Goal-Oriented:
"Users report login fails after session timeout.
Check src/auth/, especially token refresh logic.
Write a failing test that reproduces the issue,
then fix the root cause. Verify: run `make test`
and ensure all auth tests pass."
```

#### New Feature

```
❌ Vague: "Add user profile page"

✅ Goal-Oriented:
"Add a user profile page at /profile.
Look at src/pages/dashboard.tsx for the pattern.
Include: avatar, name, email, edit button.
Must be responsive (mobile-first).
Verify: take screenshot on mobile viewport,
compare with design at docs/figma/profile.png."
```

#### Refactor

```
❌ Vague: "Refactor the database layer"

✅ Goal-Oriented:
"The database layer in src/db/ uses raw SQL everywhere.
Refactor to use the repository pattern (see src/db/users.py for example).
Keep all existing tests passing — don't change public API.
Verify: run `make test` (0 failures), then `make lint` (0 warnings)."
```

#### Performance

```
❌ Vague: "Make the API faster"

✅ Goal-Oriented:
"The /api/endpoint takes 3s (profile: see logs/perf.json).
Target: < 500ms p95.
Likely cause: N+1 query in src/api/handlers/users.py.
Verify: run `wrk -t4 -c100 -d10s http://localhost:8000/api/endpoint`
and confirm p95 < 500ms."
```

#### Code Review

```
❌ Vague: "Review this PR"

✅ Goal-Oriented:
"Review PR #42 for security issues and performance.
Focus on: SQL injection, auth bypass, N+1 queries.
Check: src/api/ (new endpoints), src/auth/ (changed logic).
Output: list of issues with severity (critical/warning/info)
and specific line references."
```

### Goal-Oriented Prompting in Context Files

Tambahkan goal-oriented prompting rules ke AGENTS.md atau CLAUDE.md:

```markdown
## How to Work on Tasks

When given a task, always:
1. Understand the SITUATION (what's the problem/context?)
2. Identify the WHAT (what needs to be done?)
3. Find the WHERE (where to look/code?)
4. Define the VERIFY (how to confirm it's done?)

If the user's request is vague, ask for clarification on missing parts.
Never assume — ask when uncertain.

## Verification Rules
- Always run tests after changes
- Show evidence of completion (test output, screenshots, command results)
- Don't claim "done" without verification
```

### Level of Verification

| Level | Cara | Kapan Pakai | Token Cost |
|-------|------|-------------|------------|
| **Prompt** | "Run tests after implementing" | Task sederhana | Rendah |
| **Goal** | Set check as /goal condition | Task kompleks | Sedang |
| **Hook** | Stop hook blocks until check passes | Autonomous runs | Rendah |
| **Adversarial** | Verification subagent checks work | Critical changes | Tinggi |

### Prompt-Level Verification (Simplest)

```
Write a validateEmail function.
Test cases: user@example.com → true, invalid → false, user@.com → false.
Run the tests after implementing.
```

Agent runs tests, sees results, iterates until pass.

### Goal-Level Verification (Persistent)

```bash
# Claude Code
/goal All auth tests pass and login bug is fixed

# Hermes
/goal API response time < 500ms p95
```

Goal persists across turns. Agent keeps working until goal is achieved.

### Hook-Level Verification (Automated)

```json
{
  "hooks": {
    "Stop": [{
      "hooks": [{
        "type": "command",
        "command": "make test || exit 2"
      }]
    }]
  }
}
```

Agent CANNOT finish until tests pass. Fully autonomous.

### Adversarial Verification (Most Thorough)

```markdown
# .claude/agents/verifier.md
---
name: verifier
description: Verifies work done by other agents
tools: [Read, Bash]
---
You are a skeptical code reviewer.
Your job is to find problems in the code.
Don't trust the author's claims — verify everything.
Run the tests yourself. Check edge cases.
If you find a problem, report it with evidence.
```

```
Use @verifier to check the auth module changes.
```

Second agent with fresh context reviews the work.

### Goal-Oriented Prompting Patterns

#### Pattern 1: Explore → Plan → Code → Verify

```
Phase 1 (Explore): "Read src/auth/ and understand the session flow"
Phase 2 (Plan): "Create a plan for adding JWT refresh tokens"
Phase 3 (Code): "Implement the refresh token flow from your plan"
Phase 4 (Verify): "Run tests, fix failures, confirm all pass"
```

#### Pattern 2: Failing Test First (TDD)

```
"Write a failing test for the login timeout bug.
Don't fix the bug yet — just the test.
Run it to confirm it fails.
Then fix the code so the test passes."
```

#### Pattern 3: Screenshot Comparison (UI)

```
"[paste screenshot] Implement this design.
Take a screenshot of the result.
Compare with the original.
List differences and fix them."
```

#### Pattern 4: Incremental Verification

```
"Add the user model. Run tests."
"Add the user API. Run tests."
"Add the user frontend. Run tests."
"Connect frontend to API. Run tests."
```

Each step verified before moving on.

### Auto-Detect: Which Prompting Pattern for This Task?

```python
def select_pattern(task_type):
    if task_type == "bug_fix":
        return "Failing test first → Fix → Verify all tests pass"
    elif task_type == "new_feature":
        return "Explore → Plan → Code → Verify"
    elif task_type == "ui_change":
        return "Screenshot comparison"
    elif task_type == "refactor":
        return "Keep all tests passing throughout"
    elif task_type == "performance":
        return "Benchmark before → Optimize → Benchmark after"
    else:
        return "Explore → Plan → Code → Verify"
```

---

## Context File Ecosystem

### Files by Agent Compatibility

| File | Agents | Scope |
|------|--------|-------|
| `AGENTS.md` | Hermes, Claude Code (import), Codex, OpenCode | cwd only |
| `CLAUDE.md` | Claude Code, Hermes (import) | Project root |
| `SOUL.md` | Hermes | Agent identity |
| `.hermes.md` | Hermes | Git root + subdirs |
| `.cursor/rules/*.mdc` | Cursor | Path-scoped |
| `.claude/rules/*.md` | Claude Code | Path-scoped |

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

## Code Standards
- {rules that differ from defaults}

## How to Work on Tasks (PILAR 5)
When given a task:
1. Understand the SITUATION
2. Identify the WHAT
3. Find the WHERE
4. Define the VERIFY

Always run tests after changes. Show evidence of completion.

## Common Gotchas
- {non-obvious behaviors}

## Key Directories
- `src/` — {purpose}
- `tests/` — {purpose}
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

### Step 2: Setup MCP Servers (PILAR 2)

```bash
grep -l "postgres\|mysql\|mongo" docker-compose.yml 2>/dev/null
ls .github/ playwright.config.* 2>/dev/null
```

### Step 3: Setup Hooks (PILAR 3)

```bash
grep -l "ruff\|black\|eslint\|prettier" pyproject.toml .eslintrc* 2>/dev/null
```

### Step 4: Create Skills (PILAR 4)

```bash
ls src/api/ routes/ alembic/ migrations/ .github/workflows/ 2>/dev/null
```

### Step 5: Add Goal-Oriented Prompting Rules (PILAR 5)

Tambahkan ke AGENTS.md:
```markdown
## How to Work on Tasks
When given a task:
1. Understand the SITUATION
2. Identify the WHAT
3. Find the WHERE
4. Define the VERIFY
Always run tests after changes.
```

### Step 6: Generate Context Files

Fill templates with REAL data.

### Step 7: Verify

- Files under 200 lines
- Build/test commands work
- MCP servers connect
- Hooks execute
- Skills load
- Goal-oriented rules included

---

## How 5 PILAR Bekerja Bersama

```
┌─────────────────────────────────────────────────────────────┐
│  Agent + 5 PILAR = FULLY AUTONOMOUS                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  PILAR 1: SOUL.md        → Agent TAHU siapa dirinya        │
│  PILAR 2: MCP Servers    → Agent BISA aksi langsung         │
│  PILAR 3: Hooks          → Agent AUTO-jaga kualitas         │
│  PILAR 4: Skills         → Agent TAHU cara kerja            │
│  PILAR 5: Goal Prompting → Agent TAHU kapan selesai         │
│                                                             │
│  Context Files (AGENTS.md, CLAUDE.md)                       │
│            → Agent TAHU projectnya                          │
│                                                             │
│  Result:                                                    │
│  ┌─────────────────────────────────────────────┐            │
│  │ User: "Users report login fails after timeout"           │
│  │ Agent: *understands SITUATION (PILAR 5)*     │            │
│  │ Agent: *loads debug skill (PILAR 4)*         │            │
│  │ Agent: *follows SOUL.md style (PILAR 1)*     │            │
│  │ Agent: *queries DB via MCP (PILAR 2)*        │            │
│  │ Agent: *writes failing test*                            │
│  │ Agent: *fixes root cause*                               │
│  │ Hook: *auto-lints (PILAR 3)*                 │            │
│  │ Agent: *runs tests — all pass (VERIFY)*       │            │
│  │ Agent: "Fixed. Race condition in token        │            │
│  │         refresh. Test added. 47/47 pass."     │            │
│  │ User: *reviews PR*                                      │
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
5. **Goal-Oriented Prompting (PILAR 5)** — Clear success criteria
6. **Build/test commands**
7. **Code style rules that DIFFER from defaults**

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
5. **No goal-oriented prompting = agent doesn't know when it's done**
6. **Vague prompts = multiple correction rounds**
7. **No verification = agent claims "done" without evidence**
8. **Bloated context files REDUCE adherence** — Keep < 200 lines
9. **SOUL.md di wrong location** — Must be global
10. **`/compact` can lose context**
