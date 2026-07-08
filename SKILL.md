---
name: project-context-setup
description: "Set up AI context files for new projects across all coding agents (Hermes, Claude Code, Cursor, Codex). Maximizes AI output quality through structured context. Includes 6 PILAR: SOUL.md, MCP Servers, Hooks, Skills, Goal Prompting, Context Window Management."
version: 1.6.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [ai-context, project-setup, claude-md, agents-md, cursorrules, soul-md, mcp-servers, hooks, skills, goal-prompting, context-management, coding-agents]
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
- When uncertain, ask — don't guess on critical decisions

## Quality Standards
- Every function must have {type hints/typed parameters}
- Tests are not optional — {coverage_target}% minimum
- No silent failures — always {log/throw} on errors
```

### Lokasi

| Platform | Lokasi | Scope |
|----------|--------|-------|
| Hermes | `~/.hermes/SOUL.md` | Global |
| Claude Code | `~/.claude/CLAUDE.md` (global) | Global |
| Cursor | `~/.cursor/rules/identity.mdc` | Global |

---

## PILAR 2: MCP Servers — External Tools

MCP Servers memberi agent akses ke tools external sehingga bisa **verifikasi sendiri hasilnya**.

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

---

## PILAR 4: Skills — Reusable Procedures

Skills = **prosedur yang dipakai ulang** tanpa re-explain setiap session.

### 6 Essential Project Skills

1. **add-api-endpoint** — Standard workflow tambah endpoint
2. **database-migration** — Cara buat/test/rollback migration
3. **add-feature** — Full feature development workflow
4. **debug-issue** — Systematic debugging procedure
5. **code-review** — Review checklist & output format
6. **deploy** — Pre-deploy, deploy, post-deploy steps

---

## PILAR 5: Goal-Oriented Prompting — Clear Success Criteria

### Formula

```
GOAL = SITUATION + WHAT + WHERE + VERIFY
```

### Before/After

```
❌ "Fix the login bug"

✅ "Users report login fails after session timeout.
    Check src/auth/, especially token refresh logic.
    Write a failing test that reproduces the issue,
    then fix the root cause. Verify: run `make test`
    and ensure all auth tests pass."
```

### 4 Level of Verification

| Level | Cara | Kapan Pakai |
|-------|------|-------------|
| Prompt | "Run tests after implementing" | Task sederhana |
| Goal | `/goal` condition | Task kompleks |
| Hook | Stop hook blocks until pass | Autonomous runs |
| Adversarial | Verification subagent | Critical changes |

---

## PILAR 6: Context Window Management — Keep Agent Sharp

Context window adalah **sumber daya paling penting** yang harus dikelola. Performance agent **DROPS** saat context > 70% full.

### Kenapa Context Window Penting?

```
┌─────────────────────────────────────────────────────────────┐
│                    CONTEXT WINDOW                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────────────────────────────────┐            │
│  │ CLAUDE.md / AGENTS.md (startup context)     │ ← Always   │
│  ├─────────────────────────────────────────────┤            │
│  │ SOUL.md (agent identity)                    │ ← Always   │
│  ├─────────────────────────────────────────────┤            │
│  │ Conversation history                        │ ← Growing  │
│  ├─────────────────────────────────────────────┤            │
│  │ File reads                                  │ ← Growing  │
│  ├─────────────────────────────────────────────┤            │
│  │ Command outputs                             │ ← Growing  │
│  ├─────────────────────────────────────────────┤            │
│  │ Tool results                                │ ← Growing  │
│  ├─────────────────────────────────────────────┤            │
│  │ Subagent summaries                          │ ← Growing  │
│  ├─────────────────────────────────────────────┤            │
│  │                                             │            │
│  │         AVAILABLE SPACE                     │ ← Shrinking│
│  │                                             │            │
│  └─────────────────────────────────────────────┘            │
│                                                             │
│  Saat AVAILABLE SPACE mengecil → quality DROPS              │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Thresholds

| Usage | Status | Action |
|-------|--------|--------|
| < 70% | ✅ Normal | Full precision, no action needed |
| 70-85% | ⚠️ Warning | Consider `/compact` or start new session |
| > 85% | 🔴 Danger | Hallucination risk spikes, MUST take action |

### Token Cost Reference

| Action | Approximate Tokens |
|--------|-------------------|
| CLAUDE.md startup | 1,000-3,000 |
| SOUL.md startup | 500-1,500 |
| Reading a file | 200-2,000 per file |
| Command output | 100-5,000 per command |
| One conversation turn | 500-2,000 |
| Subagent summary | 500-3,000 |

### 5 Strategies for Context Management

#### Strategy 1: Compact — Compress When Needed

```bash
# Claude Code
/compact                          # Auto-compress
/compact focus on auth logic      # Compress with focus

# Hermes
/compress                         # Auto-compress
```

**When:** Context > 70%, agent starts "forgetting" things
**Impact:** Saves 30-50% context, but may lose some details
**Risk:** Important rules in conversation may be lost

#### Strategy 2: Split — New Session for New Task

```
Session 1: "Fix the login bug"    → 40% context used
Session 2: "Add user profile"     → Fresh 0% context
Session 3: "Refactor database"    → Fresh 0% context
```

**When:** Task is done, starting a different task
**Impact:** Each task gets full context precision
**Rule:** Don't campur debug + feature + refactor in one session

#### Strategy 3: Delegate — Subagents for Investigation

```python
# Hermes
delegate_task(goal="Investigate the auth module and find the bug")

# Claude Code
@security-reviewer review the auth module
```

**When:** Need to explore codebase without polluting main context
**Impact:** Subagent gets fresh context, returns only summary
**Rule:** Use for investigation, not implementation

#### Strategy 4: Scope — Path-Scoped Rules

```
.claude/rules/
├── frontend.md      # Only loads when editing frontend files
├── api.md           # Only loads when editing API files
├── database.md      # Only loads when editing DB files
└── testing.md       # Only loads when editing test files
```

**When:** Project has domain-specific rules
**Impact:** Only relevant rules load, saving context
**Rule:** Don't put critical global rules here

#### Strategy 5: Pipe — Input via Stdin

```bash
# Instead of having agent read a file:
cat large_file.py | claude -p "Analyze this code"

# Instead of having agent run command:
git diff HEAD~3 | claude -p "Summarize these changes"
```

**When:** You know exactly what content agent needs
**Impact:** Content goes directly to analysis, no intermediate steps
**Rule:** Use for known content, not exploration

### Context Window in Practice

```
┌─────────────────────────────────────────────────────────────┐
│  TYPICAL SESSION LIFECYCLE                                  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Turn 1-5:   Context 10-30%  → Agent is SHARP              │
│  Turn 6-10:  Context 30-50%  → Agent is GOOD               │
│  Turn 11-15: Context 50-70%  → Agent is OK                 │
│  Turn 16-20: Context 70-85%  → Agent STARTS FORGETTING     │
│  Turn 21+:   Context > 85%   → Agent MAKES MISTAKES        │
│                                                             │
│  SOLUTION: Compact at 70%, or start new session             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Context Window Rules in Context Files

Tambahkan ke AGENTS.md atau CLAUDE.md:

```markdown
## Context Management Rules

- When context is getting full, proactively use /compact
- Start a new session for unrelated tasks
- Use subagents for codebase investigation
- Keep file reads focused — don't read entire codebase
- Show evidence (test output, screenshots) instead of describing
```

### Monitoring Context Usage

**Claude Code:**
```bash
# In interactive session
/context          # Visual grid of context usage
/cost             # Token usage breakdown
```

**Hermes:**
```bash
# In interactive session
/usage            # Token usage
/insights         # Usage analytics
```

### Context-Efficient Prompting

```
❌ Context-WASTING prompt:
"Read all the files in src/ and understand the architecture,
then look at every test file, then check the CI config,
then look at the Docker setup, then tell me what you think
about the codebase quality."

✅ Context-EFFICIENT prompt:
"Look at src/api/router.py and src/api/handlers/ to understand
the API pattern. Then check tests/api/ for test patterns.
Tell me: is the router following REST conventions?"
```

### Auto-Detect: Context Management Needs

```python
# Large project = more context management needed
if count_files("src/**/*.py") > 100:
    rules.append("Use path-scoped rules for domain-specific instructions")
    rules.append("Use subagents for codebase investigation")
    rules.append("Start new sessions for unrelated tasks")

# Monorepo = definitely need path-scoped rules
if has_file("packages/") or has_file("apps/"):
    rules.append("Create .claude/rules/ for each package")
    rules.append("Use claudeMdExcludes to skip other teams' CLAUDE.md")
```

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

## How to Work on Tasks (PILAR 5)
When given a task:
1. Understand the SITUATION
2. Identify the WHAT
3. Find the WHERE
4. Define the VERIFY
Always run tests after changes.

## Context Management (PILAR 6)
- When context > 70%, use /compact
- Start new session for unrelated tasks
- Use subagents for investigation
- Keep file reads focused

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

Tambahkan SITUATION + WHAT + WHERE + VERIFY rules ke AGENTS.md.

### Step 6: Add Context Management Rules (PILAR 6)

Tambahkan context management rules ke AGENTS.md:
- Compact at 70%
- New session for new task
- Subagents for investigation
- Path-scoped rules for monorepo

### Step 7: Generate Context Files

Fill templates with REAL data.

### Step 8: Verify

- Files under 200 lines
- Build/test commands work
- MCP servers connect
- Hooks execute
- Skills load
- Goal-oriented rules included
- Context management rules included

---

## How 6 PILAR Bekerja Bersama

```
┌─────────────────────────────────────────────────────────────┐
│  Agent + 6 PILAR = FULLY AUTONOMOUS                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  PILAR 1: SOUL.md        → Agent TAHU siapa dirinya        │
│  PILAR 2: MCP Servers    → Agent BISA aksi langsung         │
│  PILAR 3: Hooks          → Agent AUTO-jaga kualitas         │
│  PILAR 4: Skills         → Agent TAHU cara kerja            │
│  PILAR 5: Goal Prompting → Agent TAHU kapan selesai         │
│  PILAR 6: Context Mgmt   → Agent TETAP SHARP                │
│                                                             │
│  Context Files (AGENTS.md, CLAUDE.md)                       │
│            → Agent TAHU projectnya                          │
│                                                             │
│  Result:                                                    │
│  ┌─────────────────────────────────────────────┐            │
│  │ Session 1: Fix login bug                     │            │
│  │   Agent: *loads debug skill (PILAR 4)*       │            │
│  │   Agent: *follows SOUL.md style (PILAR 1)*   │            │
│  │   Agent: *queries DB via MCP (PILAR 2)*      │            │
│  │   Agent: *writes test, fixes code*           │            │
│  │   Hook: *auto-lints (PILAR 3)*               │            │
│  │   Agent: *runs tests (PILAR 5 VERIFY)*       │            │
│  │   Context: 45% — still sharp (PILAR 6)       │            │
│  │   Agent: "Done. All tests pass."             │            │
│  │                                              │            │
│  │ Session 2: Add user profile (FRESH)          │            │
│  │   Agent: *full context precision*            │            │
│  │   Agent: *loads add-feature skill*           │            │
│  │   ...                                        │            │
│  └─────────────────────────────────────────────┘            │
│                                                             │
│  Each session = fresh context, full precision               │
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
6. **Context Window Management (PILAR 6)** — Keep agent sharp
7. **Build/test commands**
8. **Code style rules that DIFFER from defaults**

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
6. **No context management = agent degrades over time**
7. **Context > 85% = hallucination risk**
8. **Bloated context files REDUCE adherence** — Keep < 200 lines
9. **SOUL.md di wrong location** — Must be global
10. **`/compact` can lose context** — Important rules in context files, not conversation
