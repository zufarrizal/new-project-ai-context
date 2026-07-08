---
name: project-context-setup
description: "Set up AI context files for new projects across all coding agents (Hermes, Claude Code, Cursor, Codex). Maximizes AI output quality through structured context. 7 PILAR: SOUL.md, MCP Servers, Hooks, Skills, Goal Prompting, Context Mgmt, Subagents."
version: 1.7.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [ai-context, project-setup, claude-md, agents-md, soul-md, mcp-servers, hooks, skills, goal-prompting, context-management, subagents, coding-agents]
    related_skills: [hermes-agent, claude-code, codex, opencode]
---

# Project Context Setup

Set up AI context files that maximize coding agent effectiveness. 7 PILAR framework for fully autonomous AI coding.

**Core principle:** Every line in context files should pass: "Would removing this cause the agent to make mistakes?"

## When to Use

- Starting a new project from scratch
- Cloning an existing repo that lacks AI context files
- User says "project baru", "new project", "setup project", or similar

---

## PILAR 1: SOUL.md — Agent Identity

Agent TAHU siapa dirinya. Karakter konsisten lintas session.

```markdown
# Agent Identity
You are a {senior_role} specializing in {domain}.
## Expertise
- Primary: {main_tech_stack}
- Secondary: {supporting_tech}
## Coding Philosophy
- {principle_1}
- {principle_2}
## Behavior Rules
- Always explain trade-offs before architectural decisions
- Prefer {language/framework} over {alternative}
## Quality Standards
- Tests are not optional — {coverage_target}% minimum
```

Lokasi: `~/.hermes/SOUL.md` (Hermes) | `~/.claude/CLAUDE.md` (Claude Code)

---

## PILAR 2: MCP Servers — External Tools

Agent BISA aksi langsung. Query database, kelola GitHub, test browser.

```bash
# Claude Code
claude mcp add postgres -- npx @anthropic-ai/server-postgres --connection-string postgresql://localhost/mydb
claude mcp add github -- npx @modelcontextprotocol/server-github

# Hermes
hermes mcp add postgres --url postgresql://localhost:5432/mydb
```

---

## PILAR 3: Hooks — Automation on Events

Agent AUTO-jaga kualitas. Auto-lint, auto-test, block dangerous commands.

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

---

## PILAR 4: Skills — Reusable Procedures

Agent TAHU cara kerja. Tidak reinvent the wheel setiap session.

6 Essential Skills: add-api-endpoint, database-migration, add-feature, debug-issue, code-review, deploy

```markdown
# .claude/skills/add-api-endpoint.md
When asked to add a new API endpoint:
1. Check existing endpoints for patterns
2. Create handler in src/api/handlers/
3. Register route in router.py
4. Write test in tests/api/
5. Run `make test`
```

---

## PILAR 5: Goal-Oriented Prompting — Clear Success Criteria

Agent TAHU kapan selesai. Self-correct tanpa user intervention.

```
GOAL = SITUATION + WHAT + WHERE + VERIFY
```

```
❌ "Fix the login bug"
✅ "Users report login fails after timeout. Check src/auth/token_refresh.py.
    Write failing test, fix root cause. Verify: all auth tests pass."
```

4 Level: Prompt → Goal → Hook → Adversarial

---

## PILAR 6: Context Window Management — Keep Agent Sharp

Agent TETAP SHARP. Performance drops saat context > 70%.

| Usage | Status | Action |
|-------|--------|--------|
| < 70% | ✅ Normal | No action |
| 70-85% | ⚠️ Warning | `/compact` |
| > 85% | 🔴 Danger | New session |

5 Strategies: Compact, Split, Delegate, Scope, Pipe

---

## PILAR 7: Subagents & Parallelism — Scale Beyond One Agent

Subagents memungkinkan agent **mendelegasikan task** ke agent lain dengan **isolated context** dan **parallel execution**.

### Kenapa Subagents Penting?

```
┌─────────────────────────────────────────────────────────────┐
│                    WITHOUT SUBAGENTS                        │
├─────────────────────────────────────────────────────────────┤
│  User: "Build auth, API, and docs"                          │
│  Agent: *does auth (50% context)*                           │
│  Agent: *does API (75% context)*                            │
│  Agent: *forgets auth details (context full)*               │
│  Agent: *makes mistake in docs*                             │
│  User: "The docs are wrong!"                                │
│  → Single agent, degrading quality                          │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                    WITH SUBAGENTS                           │
├─────────────────────────────────────────────────────────────┤
│  User: "Build auth, API, and docs"                          │
│  Agent A: *auth module (fresh context)*                     │
│  Agent B: *API endpoints (fresh context)*                   │
│  Agent C: *documentation (fresh context)*                   │
│  → Parallel execution, each with full precision             │
│  → Main agent coordinates, reviews results                  │
└─────────────────────────────────────────────────────────────┘
```

### Subagents by Platform

| Platform | Method | Scope |
|----------|--------|-------|
| Hermes | `delegate_task()` | Full tool access, isolated context |
| Claude Code | Custom agents (`.claude/agents/`) | Defined tools, fresh context |
| Cursor | — | Not supported natively |

### Hermes: delegate_task

#### Single Task

```python
delegate_task(
    goal="Refactor auth module to use JWT tokens",
    context="Files: src/auth/, src/middleware/. Current: session-based auth. Target: JWT.",
    role="leaf"
)
```

#### Parallel Tasks (Batch)

```python
delegate_task(tasks=[
    {
        "goal": "Refactor auth module to use JWT",
        "context": "src/auth/ — current: session-based, target: JWT"
    },
    {
        "goal": "Write integration tests for API endpoints",
        "context": "tests/api/ — use pytest, follow existing patterns"
    },
    {
        "goal": "Update API documentation",
        "context": "docs/api.md — document new JWT flow"
    }
])
```

#### Background Execution

```python
delegate_task(
    goal="Run full test suite and fix any failures",
    context="Run `make test`, fix failures, re-run until green",
    background=True  # Returns immediately, result comes later
)
```

#### Orchestrator vs Leaf

| Role | Can Delegate? | Use Case |
|------|--------------|----------|
| `leaf` | ❌ No | Focused worker, single task |
| `orchestrator` | ✅ Yes | Coordinates multiple workers |

```python
# Orchestrator that spawns its own workers
delegate_task(
    goal="Build the entire auth system",
    context="Design JWT flow, implement, test, document",
    role="orchestrator"
)
```

### Claude Code: Custom Subagents

#### Define Agent

```markdown
# .claude/agents/security-reviewer.md
---
name: security-reviewer
description: Security-focused code review
model: opus
tools: [Read, Bash]
---
You are a senior security engineer. Review code for:
- Injection vulnerabilities (SQL, XSS, command injection)
- Authentication/authorization flaws
- Secrets in code
- Unsafe deserialization
```

```markdown
# .claude/agents/test-writer.md
---
name: test-writer
description: Write comprehensive tests
model: sonnet
tools: [Read, Write, Bash]
---
You are a test engineer. Write tests that:
- Cover happy path and edge cases
- Use existing test patterns in the project
- Achieve > 90% coverage for the target file
- Are readable and maintainable
```

```markdown
# .claude/agents/doc-writer.md
---
name: doc-writer
description: Write and update documentation
model: haiku
tools: [Read, Write]
---
You are a technical writer. Write docs that:
- Are concise and clear
- Include code examples
- Follow the existing doc style
- Cover API, setup, and common workflows
```

#### Invoke Agent

```
@security-reviewer review the auth module changes
@test-writer write tests for src/api/handlers/users.py
@doc-writer update API docs for the new JWT endpoints
```

#### Dynamic Agents via CLI

```bash
claude --agents '{
  "reviewer": {
    "description": "Reviews code for performance",
    "prompt": "You are a performance-focused code reviewer"
  }
}' -p "Use @reviewer to check auth.py"
```

### When to Use Subagents

| Situation | Use Subagent? | Why |
|-----------|--------------|-----|
| Task sederhana (< 5 file) | ❌ | Overhead tidak worth it |
| Investigasi codebase besar | ✅ | Fresh context, isolated |
| Parallel independent tasks | ✅ | Concurrent execution |
| Adversarial review | ✅ | Second opinion |
| Task butuh context berbeda | ✅ | Isolated window |
| Long-running task | ✅ | Don't block main agent |
| Cross-cutting concern | ✅ | e.g., security review |

### Parallel Execution Patterns

#### Pattern 1: Fan-Out (Independent Tasks)

```
Main Agent
├── Task A: Backend (subagent 1)
├── Task B: Frontend (subagent 2)
└── Task C: Tests (subagent 3)
    → All run in parallel
    → Main agent collects results
```

```python
delegate_task(tasks=[
    {"goal": "Build REST API for user management"},
    {"goal": "Build React dashboard for user management"},
    {"goal": "Write integration tests for user API"}
])
```

#### Pattern 2: Pipeline (Sequential Dependencies)

```
Main Agent
├── Step 1: Design schema (subagent)
│   → Returns: schema design
├── Step 2: Implement models (subagent, uses Step 1 output)
│   → Returns: model code
├── Step 3: Build API (subagent, uses Step 2 output)
│   → Returns: API code
└── Step 4: Write tests (subagent, uses Step 3 output)
    → Returns: test suite
```

```python
# Sequential delegation with context passing
schema = delegate_task(goal="Design user schema", context="Requirements: name, email, role")
models = delegate_task(goal="Implement SQLAlchemy models", context=f"Schema: {schema}")
api = delegate_task(goal="Build FastAPI endpoints", context=f"Models: {models}")
tests = delegate_task(goal="Write tests", context=f"API: {api}")
```

#### Pattern 3: Adversarial (Build + Review)

```
Main Agent
├── Builder: Implement feature (subagent 1)
└── Reviewer: Find problems (subagent 2)
    → Reviewer checks Builder's work
    → Main agent decides what to fix
```

```python
builder = delegate_task(goal="Implement JWT auth in src/auth/")
reviewer = delegate_task(
    goal="Review JWT implementation for security issues",
    context=f"Code: {builder}"
)
```

#### Pattern 4: Specialist (Domain Experts)

```
Main Agent (orchestrator)
├── @security-reviewer: Check auth
├── @performance-expert: Check queries
├── @test-writer: Write tests
└── @doc-writer: Update docs
```

### Subagent Context Passing

Subagents don't share context with parent. Pass everything explicitly:

```python
# ❌ BAD: Subagent doesn't know what parent knows
delegate_task(goal="Fix the bug")

# ✅ GOOD: Pass all relevant context
delegate_task(
    goal="Fix race condition in token refresh",
    context="""
    File: src/auth/token_refresh.py:42
    Error: Token refresh fails when called concurrently
    Current code uses shared state without locking
    Fix: Use threading.Lock or asyncio.Lock
    Verify: Run `pytest tests/auth/ -v` — all must pass
    """
)
```

### Subagent Best Practices

1. **Be specific in goal** — "Fix race condition in token_refresh.py:42" not "Fix the bug"
2. **Pass all context** — Subagent knows nothing about parent conversation
3. **Define verification** — Tell subagent how to confirm success
4. **Set timeouts** — Complex tasks can take 5-10 minutes
5. **Use background for long tasks** — Don't block main agent
6. **Clean up** — Kill tmux sessions when done
7. **Review results** — Subagent self-reports may be wrong

### Auto-Detect: When to Use Subagents

```python
# Task complexity heuristic
if estimated_files > 10:
    strategy = "Use subagents for parallel work"
elif task_involves_multiple_domains:
    strategy = "Use specialist subagents"
elif task_is_investigation:
    strategy = "Use subagent for exploration"
elif task_is_review:
    strategy = "Use adversarial subagent"
else:
    strategy = "Do it directly"
```

---

## How 7 PILAR Bekerja Bersama

```
┌─────────────────────────────────────────────────────────────┐
│  Agent + 7 PILAR = FULLY AUTONOMOUS                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  PILAR 1: SOUL.md        → Agent TAHU siapa dirinya        │
│  PILAR 2: MCP Servers    → Agent BISA aksi langsung         │
│  PILAR 3: Hooks          → Agent AUTO-jaga kualitas         │
│  PILAR 4: Skills         → Agent TAHU cara kerja            │
│  PILAR 5: Goal Prompting → Agent TAHU kapan selesai         │
│  PILAR 6: Context Mgmt   → Agent TETAP SHARP                │
│  PILAR 7: Subagents      → Agent BISA SCALE                 │
│                                                             │
│  Context Files (AGENTS.md, CLAUDE.md)                       │
│            → Agent TAHU projectnya                          │
│                                                             │
│  FULL WORKFLOW:                                             │
│  ┌─────────────────────────────────────────────┐            │
│  │ User: "Build complete auth system"           │            │
│  │                                              │            │
│  │ Main Agent (orchestrator):                   │            │
│  │   1. Understand SITUATION (PILAR 5)          │            │
│  │   2. Load skill (PILAR 4)                    │            │
│  │   3. Follow SOUL.md style (PILAR 1)          │            │
│  │   4. Spawn subagents (PILAR 7):              │            │
│  │      ├─ @security: Review auth design        │            │
│  │      ├─ @backend: Implement JWT              │            │
│  │      └─ @test-writer: Write tests            │            │
│  │   5. Each subagent:                          │            │
│  │      ├─ Fresh context (PILAR 6)              │            │
│  │      ├─ Query DB via MCP (PILAR 2)           │            │
│  │      └─ Auto-lint via Hooks (PILAR 3)        │            │
│  │   6. Main agent reviews results              │            │
│  │   7. Verify all tests pass (PILAR 5)         │            │
│  │                                              │            │
│  │ Result: "Auth system built, reviewed,        │            │
│  │          tested. PR ready."                  │            │
│  │                                              │            │
│  │ User: *reviews PR*                           │            │
│  └─────────────────────────────────────────────┘            │
│                                                             │
└─────────────────────────────────────────────────────────────┘
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

Tambahkan SITUATION + WHAT + WHERE + VERIFY rules ke AGENTS.md.

### Step 6: Add Context Management Rules (PILAR 6)

Tambahkan context management rules ke AGENTS.md.

### Step 7: Define Subagents (PILAR 7)

Buat `.claude/agents/` untuk specialist agents:
- security-reviewer
- test-writer
- doc-writer
- performance-expert

### Step 8: Generate Context Files

Fill templates with REAL data.

### Step 9: Verify

- Files under 200 lines
- All commands work
- MCP servers connect
- Hooks execute
- Skills load
- Subagents defined

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

## How to Work (PILAR 5)
1. Understand SITUATION
2. Identify WHAT
3. Find WHERE
4. Define VERIFY
Always run tests after changes.

## Context Management (PILAR 6)
- Compact at 70%
- New session for new task
- Subagents for investigation

## Subagents (PILAR 7)
- Use @security-reviewer for auth changes
- Use @test-writer for new features
- Use parallel subagents for independent tasks

## Common Gotchas
- {non-obvious behaviors}
```

---

## What Makes Context IMPACTFUL

### Tier 1 — Highest Impact
1. **SOUL.md (PILAR 1)** — Agent identity
2. **MCP Servers (PILAR 2)** — Agent can verify
3. **Hooks (PILAR 3)** — Auto-quality
4. **Skills (PILAR 4)** — Reusable procedures
5. **Goal-Oriented Prompting (PILAR 5)** — Clear success criteria
6. **Context Window Management (PILAR 6)** — Keep agent sharp
7. **Subagents (PILAR 7)** — Scale beyond one agent

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
5. **No goal-oriented prompting = agent doesn't know when done**
6. **No context management = agent degrades over time**
7. **No subagents = can't scale, context fills fast**
8. **Subagent without context = subagent doesn't know what to do**
9. **Parallel tasks without isolation = conflicts**
10. **Not reviewing subagent results = trusting self-reports**
