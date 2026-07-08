---
name: project-context-setup
description: "Set up AI context files for new projects across all coding agents (Hermes, Claude Code, Cursor, Codex). Maximizes AI output quality through structured context. 7 PILAR: SOUL.md, MCP Servers, Hooks, Skills, Goal Prompting, Context Mgmt, Subagents."
version: 1.8.0
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

Agent knows WHO it is. Consistent personality across sessions.

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
- Always explain trade-offs before making architectural decisions
- Prefer {language/framework} over {alternative}
## Quality Standards
- Tests are not optional — {coverage_target}% minimum
```

Location: `~/.hermes/SOUL.md` (Hermes) | `~/.claude/CLAUDE.md` (Claude Code)

---

## PILAR 2: MCP Servers — External Tools

Agent CAN take action directly. Query database, manage GitHub, test browser.

```bash
# Claude Code
claude mcp add postgres -- npx @anthropic-ai/server-postgres --connection-string postgresql://localhost/mydb
claude mcp add github -- npx @modelcontextprotocol/server-github

# Hermes
hermes mcp add postgres --url postgresql://localhost:5432/mydb
```

---

## PILAR 3: Hooks — Automation on Events

Agent AUTO-enforces quality. Auto-lint, auto-test, block dangerous commands.

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

Agent KNOWS the workflow. No reinventing the wheel each session.

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

Agent KNOWS when it's done. Self-corrects without user intervention.

```
GOAL = SITUATION + WHAT + WHERE + VERIFY
```

```
BAD: "Fix the login bug"

GOOD: "Users report login fails after session timeout.
       Check src/auth/, especially token refresh logic.
       Write a failing test that reproduces the issue,
       then fix the root cause. Verify: run `make test`
       and ensure all auth tests pass."
```

4 Levels: Prompt → Goal → Hook → Adversarial

---

## PILAR 6: Context Window Management — Keep Agent Sharp

Agent STAYS SHARP. Performance drops when context > 70% full.

| Usage | Status | Action |
|-------|--------|--------|
| < 70% | OK Normal | No action needed |
| 70-85% | Warning | Use `/compact` |
| > 85% | DANGER | Start new session |

5 Strategies: Compact, Split, Delegate, Scope, Pipe

---

## PILAR 7: Subagents & Parallelism — Scale Beyond One Agent

Agent CAN SCALE. Delegates tasks to other agents with isolated context.

### 4 Parallel Patterns

```
1. Fan-Out (Independent Tasks)
   Main Agent -> Task A + Task B + Task C (parallel)

2. Pipeline (Sequential Dependencies)
   Main Agent -> Step 1 -> Step 2 -> Step 3 -> Step 4

3. Adversarial (Build + Review)
   Main Agent -> Builder + Reviewer (parallel)

4. Specialist (Domain Experts)
   Main Agent -> @security + @test-writer + @doc-writer
```

### When to Use Subagents

| Situation | Use Subagent? | Reason |
|-----------|--------------|--------|
| Simple task (< 5 files) | No | Overhead not worth it |
| Large codebase investigation | Yes | Fresh context, isolated |
| Parallel independent tasks | Yes | Concurrent execution |
| Adversarial review | Yes | Second opinion |
| Task needs different context | Yes | Isolated window |

### Hermes: delegate_task

```python
# Parallel batch
delegate_task(tasks=[
    {"goal": "Refactor auth module to use JWT"},
    {"goal": "Write integration tests for API endpoints"},
    {"goal": "Update API documentation"}
])
```

### Claude Code: Custom Subagents

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

Invoke: `@security-reviewer review the auth module`

---

## Context File Ecosystem

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

## How to Work (PILAR 5)
When given a task:
1. Understand the SITUATION
2. Identify the WHAT
3. Find the WHERE
4. Define the VERIFY
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

Add SITUATION + WHAT + WHERE + VERIFY rules to AGENTS.md.

### Step 6: Add Context Management Rules (PILAR 6)

Add context management rules to AGENTS.md.

### Step 7: Define Subagents (PILAR 7)

Create `.claude/agents/` for specialist agents.

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

## How 7 PILAR Work Together

```
User: "Build complete auth system"

Main Agent (orchestrator):
  1. Understand SITUATION (PILAR 5)
  2. Load skill (PILAR 4)
  3. Follow SOUL.md style (PILAR 1)
  4. Spawn subagents (PILAR 7):
     - @security: Review auth design
     - @backend: Implement JWT
     - @test-writer: Write tests
  5. Each subagent:
     - Fresh context (PILAR 6)
     - Query DB via MCP (PILAR 2)
     - Auto-lint via Hooks (PILAR 3)
  6. Main agent reviews results
  7. Verify all tests pass (PILAR 5)

Result: "Auth system built, reviewed, tested. PR ready."
User: *reviews PR*
```

---

## What Makes Context IMPACTFUL

### Tier 1 — Highest Impact
1. SOUL.md (PILAR 1) — Agent identity
2. MCP Servers (PILAR 2) — Agent can verify
3. Hooks (PILAR 3) — Auto-quality
4. Skills (PILAR 4) — Reusable procedures
5. Goal-Oriented Prompting (PILAR 5) — Clear success criteria
6. Context Window Management (PILAR 6) — Keep agent sharp
7. Subagents (PILAR 7) — Scale beyond one agent

### NEVER Include
- Standard language conventions (agent already knows)
- Detailed API docs (link instead)
- Self-evident practices ("write clean code")
- Info that changes frequently

---

## Installing This Skill

```bash
hermes skills install zufarrizal/new-project-ai-context
```

---

## Pitfalls

1. No SOUL.md = inconsistent personality
2. No MCP = agent cannot verify its own work
3. No hooks = user becomes babysitter
4. No skills = repeatable tasks need re-explaining
5. No goal-oriented prompting = agent does not know when done
6. No context management = agent degrades over time
7. No subagents = cannot scale, context fills fast
8. Subagent without context = subagent does not know what to do
9. Parallel tasks without isolation = conflicts
10. Not reviewing subagent results = trusting self-reports
