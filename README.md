<div align="center">

# new-project-ai-context

### 7 PILAR Framework for AI Agent Effectiveness

**Stop repeating yourself to AI agents. Start every project with context that makes them 10x more effective.**

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](#contributing)

</div>

---

## Why This Exists

Every time you start a new project, AI coding agents begin with **zero context**. They don't know your build commands, your code style, your testing setup, or your project's gotchas.

You end up explaining the same things **over and over**.

This skill fixes that with the **7 PILAR Framework** — research-backed from Anthropic's internal data and community best practices.

---

## The 7 PILAR Framework

| PILAR | Name | Function | Agent... |
|-------|------|----------|----------|
| 1 | **SOUL.md** | Agent Identity | KNOWS who it is |
| 2 | **MCP Servers** | External Tools | CAN take action |
| 3 | **Hooks** | Automation | AUTO-enforces quality |
| 4 | **Skills** | Reusable Procedures | KNOWS the workflow |
| 5 | **Goal Prompting** | Clear Success Criteria | KNOWS when done |
| 6 | **Context Mgmt** | Keep Agent Sharp | STAYS SHARP |
| 7 | **Subagents** | Parallelism | CAN SCALE |

```
Agent + 7 PILAR = FULLY AUTONOMOUS

PILAR 1: SOUL.md        -> Agent KNOWS who it is
PILAR 2: MCP Servers    -> Agent CAN take action
PILAR 3: Hooks          -> Agent AUTO-enforces quality
PILAR 4: Skills         -> Agent KNOWS the workflow
PILAR 5: Goal Prompting -> Agent KNOWS when done
PILAR 6: Context Mgmt   -> Agent STAYS SHARP
PILAR 7: Subagents      -> Agent CAN SCALE

Result: User = reviewer, not babysitter
```

---

## Quick Start

### As a Hermes Skill
```bash
hermes skills install zufarrizal/new-project-ai-context
```

### Manual Usage
Just say to your AI agent:
> "Setup AI context for this project"

The agent will:
1. **Create SOUL.md** — Agent identity (PILAR 1)
2. **Setup MCP Servers** — Database, GitHub, browser (PILAR 2)
3. **Configure Hooks** — Auto-lint, auto-test (PILAR 3)
4. **Create Skills** — Reusable workflows (PILAR 4)
5. **Add Goal Rules** — Clear success criteria (PILAR 5)
6. **Add Context Rules** — Keep agent sharp (PILAR 6)
7. **Define Subagents** — Specialist agents (PILAR 7)
8. **Generate Context Files** — AGENTS.md, CLAUDE.md, .hermes.md

---

## Each PILAR Explained

### PILAR 1: SOUL.md — Agent Identity

```markdown
# Agent Identity
You are a senior backend engineer specializing in Python and Go APIs.
## Expertise
- Primary: Python (FastAPI, SQLAlchemy), Go (Gin, GORM)
## Coding Philosophy
- Solve the problem with the least code possible
- If it's not tested, it's broken
## Quality Standards
- Type hints on ALL functions
- 90% test coverage minimum
```

**Impact:** Agent has consistent personality across sessions.

### PILAR 2: MCP Servers — External Tools

```bash
claude mcp add postgres -- npx @anthropic-ai/server-postgres --connection-string postgresql://localhost/mydb
claude mcp add github -- npx @modelcontextprotocol/server-github
```

**Impact:** Agent can verify its own work.

### PILAR 3: Hooks — Automation

```json
{
  "PostToolUse": [
    {"matcher": "Write(*.py)", "hooks": [{"type": "command", "command": "ruff check --fix $CLAUDE_FILE_PATHS"}]}
  ]
}
```

**Impact:** Auto-quality enforcement without user intervention.

### PILAR 4: Skills — Reusable Procedures

```markdown
# .claude/skills/add-api-endpoint.md
When asked to add a new API endpoint:
1. Check existing endpoints for patterns
2. Create handler in src/api/handlers/
3. Register route in router.py
4. Write test in tests/api/
5. Run `make test`
```

**Impact:** Agent doesn't reinvent the wheel.

### PILAR 5: Goal-Oriented Prompting

```
BAD: "Fix the login bug"
GOOD: "Users report login fails after timeout. Check src/auth/token_refresh.py.
       Write failing test, fix root cause. Verify: all auth tests pass."
```

**Impact:** Agent can self-correct without user intervention.

### PILAR 6: Context Window Management

| Usage | Status | Action |
|-------|--------|--------|
| < 70% | OK | No action |
| 70-85% | Warning | `/compact` |
| > 85% | DANGER | New session |

**Impact:** Agent stays sharp throughout session.

### PILAR 7: Subagents & Parallelism

```python
delegate_task(tasks=[
    {"goal": "Refactor auth module to use JWT"},
    {"goal": "Write integration tests for API"},
    {"goal": "Update API documentation"}
])
```

**Impact:** Parallel execution, isolated contexts, full precision.

---

## Files Generated

| File | Agent | Purpose |
|------|-------|---------|
| `SOUL.md` | All | Agent identity (global) |
| `AGENTS.md` | All | Portable project context |
| `CLAUDE.md` | Claude Code | Imports AGENTS.md + additions |
| `.hermes.md` | Hermes | Hierarchical project rules |
| `.claude/agents/*.md` | Claude Code | Specialist subagents |
| `.claude/skills/*.md` | Claude Code | Reusable workflows |
| `.claude/settings.json` | Claude Code | Hooks configuration |

---

## The Science Behind It

### Context Window = Most Important Resource

> *"Claude's context window fills up fast, and performance degrades as it fills."*
> — [Anthropic Best Practices](https://code.claude.com/docs/en/best-practices)

The 7 PILAR framework addresses this:
- **PILAR 1-5** = Make agent effective when context is available
- **PILAR 6** = Keep context lean so quality doesn't degrade
- **PILAR 7** = Scale beyond single context window

---

## Compatible Agents

| Agent | AGENTS.md | CLAUDE.md | SOUL.md | MCP | Hooks | Skills | Subagents |
|-------|-----------|-----------|---------|-----|-------|--------|-----------|
| **Hermes** | Yes | Import | Yes | Yes | Yes | Yes | Yes |
| **Claude Code** | Import | Yes | Global | Yes | Yes | Yes | Yes |
| **Cursor** | — | Yes | Global | — | — | — | — |
| **Codex** | Yes | Yes | — | — | — | — | — |

---

## Best Practices

### DO
- Create SOUL.md FIRST (PILAR 1)
- Setup MCP for verification (PILAR 2)
- Use hooks for auto-quality (PILAR 3)
- Create skills for repeatable tasks (PILAR 4)
- Use goal-oriented prompts (PILAR 5)
- Monitor context usage (PILAR 6)
- Use subagents for parallel work (PILAR 7)
- Keep files under 200 lines
- Check into git for team sharing

### DON'T
- Skip SOUL.md (inconsistent personality)
- Skip MCP (agent can't verify)
- Skip hooks (user becomes babysitter)
- Skip skills (repeat tasks need re-explaining)
- Use vague prompts (multiple correction rounds)
- Ignore context > 85% (hallucination risk)
- Use subagents for simple tasks (overhead)

---

## Contributing

PRs welcome! Here's how:

1. Fork this repo
2. Create a branch: `git checkout -b feat/amazing-feature`
3. Commit: `git commit -m 'feat: add amazing feature'`
4. Push: `git push origin feat/amazing-feature`
5. Open a PR

### Ideas for Contributions
- Templates for more tech stacks (Rust, Go, Flutter, etc.)
- More specialist subagent definitions
- Integration with more AI agents
- Real-world examples from your projects

---

## License

MIT — use it however you want. See [LICENSE](LICENSE).

---

## Credits

- [Anthropic](https://docs.anthropic.com) — Claude Code best practices research
- [Hermes Agent](https://github.com/NousResearch/hermes-agent) — Skill system and context file hierarchy
- [Awesome Cursor Rules](https://github.com/PatrickJS/awesome-cursorrules) — Cursor .mdc patterns
- The AI coding community — for sharing what actually works

---

<div align="center">

**Made with by [zufarrizal](https://github.com/zufarrizal)**

*7 PILAR Framework — Because every project deserves an AI agent that actually knows what it's doing.*

</div>
