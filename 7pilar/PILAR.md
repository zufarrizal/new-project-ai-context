# 7 PILAR AI AGENT FOCUS

> Complete framework for making AI coding agents truly focused and effective in any project.

---

## Summary

| PILAR | Name | Function | Agent... |
|-------|------|----------|----------|
| 1 | SOUL.md | Agent Identity | KNOWS who it is |
| 2 | MCP Servers | External Tools | CAN take action |
| 3 | Hooks | Automation | AUTO-enforces quality |
| 4 | Skills | Reusable Procedures | KNOWS the workflow |
| 5 | Goal Prompting | Clear Success Criteria | KNOWS when done |
| 6 | Context Mgmt | Keep Agent Sharp | STAYS SHARP |
| 7 | Subagents | Parallelism | CAN SCALE |

---

## PILAR 1: SOUL.md — Agent Identity

Agent KNOWS who it is. Consistent personality across sessions.

```markdown
# Agent Identity
You are a senior backend engineer specializing in Python and Go APIs.
## Expertise
- Primary: Python (FastAPI, SQLAlchemy), Go (Gin, GORM)
- Secondary: PostgreSQL, Redis, Docker, Kubernetes
## Coding Philosophy
- Solve the problem with the least code possible
- Measure before optimizing
- If it's not tested, it's broken
## Quality Standards
- Type hints on ALL functions
- 90% test coverage minimum
- No print() in production — use structlog
```

Location: `~/.hermes/SOUL.md` | `~/.claude/CLAUDE.md` (global)

---

## PILAR 2: MCP Servers — External Tools

Agent CAN take action directly. Query database, manage GitHub, test browser.

```bash
claude mcp add postgres -- npx @anthropic-ai/server-postgres --connection-string postgresql://localhost/mydb
claude mcp add github -- npx @modelcontextprotocol/server-github
claude mcp add puppeteer -- npx @anthropic-ai/server-puppeteer
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

Agent KNOWS the workflow. No reinventing the wheel.

6 Essential Skills: add-api-endpoint, database-migration, add-feature, debug-issue, code-review, deploy

---

## PILAR 5: Goal-Oriented Prompting — Clear Success Criteria

Agent KNOWS when it's done. Self-corrects without user intervention.

```
GOAL = SITUATION + WHAT + WHERE + VERIFY
```

```
BAD: "Fix the login bug"

GOOD: "Users report login fails after timeout. Check src/auth/token_refresh.py.
       Write failing test, fix root cause. Verify: all auth tests pass."
```

4 Levels: Prompt -> Goal -> Hook -> Adversarial

---

## PILAR 6: Context Window Management — Keep Agent Sharp

Agent STAYS SHARP. Performance drops when context > 70% full.

| Usage | Status | Action |
|-------|--------|--------|
| < 70% | OK | No action |
| 70-85% | Warning | `/compact` |
| > 85% | DANGER | New session |

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

| Situation | Use Subagent? |
|-----------|--------------|
| Simple task (< 5 files) | No |
| Large codebase investigation | Yes |
| Parallel independent tasks | Yes |
| Adversarial review | Yes |
| Task needs different context | Yes |

---

## Full Workflow

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

## References

- [Anthropic: Best Practices](https://code.claude.com/docs/en/best-practices)
- [Anthropic: Memory](https://code.claude.com/docs/en/memory)
- [Anthropic: Extend Claude Code](https://code.claude.com/docs/en/extend)
- [Hermes Agent](https://github.com/NousResearch/hermes-agent)
- [Awesome Cursor Rules](https://github.com/PatrickJS/awesome-cursorrules)

---

*Created by [zufarrizal](https://github.com/zufarrizal) — July 2026*
