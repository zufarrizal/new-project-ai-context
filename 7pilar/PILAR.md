# 7 PILAR AI AGENT FOCUS

> Framework lengkap untuk membuat AI coding agent benar-benar fokus dan efektif dalam sebuah project.

---

## Ringkasan

| PILAR | Nama | Fungsi | Agent... |
|-------|------|--------|----------|
| 1 | SOUL.md | Agent Identity | TAHU siapa dirinya |
| 2 | MCP Servers | External Tools | BISA aksi langsung |
| 3 | Hooks | Automation | AUTO-jaga kualitas |
| 4 | Skills | Reusable Procedures | TAHU cara kerja |
| 5 | Goal Prompting | Clear Success Criteria | TAHU kapan selesai |
| 6 | Context Mgmt | Keep Agent Sharp | TETAP SHARP |
| 7 | Subagents | Parallelism | BISA SCALE |

---

## PILAR 1: SOUL.md — Agent Identity

Agent TAHU siapa dirinya. Karakter konsisten lintas session.

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

Lokasi: `~/.hermes/SOUL.md` | `~/.claude/CLAUDE.md` (global)

---

## PILAR 2: MCP Servers — External Tools

Agent BISA aksi langsung. Query database, kelola GitHub, test browser.

```bash
claude mcp add postgres -- npx @anthropic-ai/server-postgres --connection-string postgresql://localhost/mydb
claude mcp add github -- npx @modelcontextprotocol/server-github
claude mcp add puppeteer -- npx @anthropic-ai/server-puppeteer
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

Agent TAHU cara kerja. Tidak reinvent the wheel.

6 Essential Skills: add-api-endpoint, database-migration, add-feature, debug-issue, code-review, deploy

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

Agent BISA SCALE. Mendelegasikan task ke agent lain dengan isolated context.

### 4 Parallel Patterns

```
1. Fan-Out (Independent Tasks)
   Main Agent → Task A + Task B + Task C (parallel)

2. Pipeline (Sequential Dependencies)
   Main Agent → Step 1 → Step 2 → Step 3 → Step 4

3. Adversarial (Build + Review)
   Main Agent → Builder + Reviewer (parallel)

4. Specialist (Domain Experts)
   Main Agent → @security + @test-writer + @doc-writer
```

### When to Use

| Situation | Use Subagent? |
|-----------|--------------|
| Task sederhana (< 5 file) | ❌ |
| Investigasi codebase besar | ✅ |
| Parallel independent tasks | ✅ |
| Adversarial review | ✅ |
| Task butuh context berbeda | ✅ |

---

## Full Workflow

```
User: "Build complete auth system"

Main Agent (orchestrator):
  1. Understand SITUATION (PILAR 5)
  2. Load skill (PILAR 4)
  3. Follow SOUL.md style (PILAR 1)
  4. Spawn subagents (PILAR 7):
     ├─ @security: Review auth design
     ├─ @backend: Implement JWT
     └─ @test-writer: Write tests
  5. Each subagent:
     ├─ Fresh context (PILAR 6)
     ├─ Query DB via MCP (PILAR 2)
     └─ Auto-lint via Hooks (PILAR 3)
  6. Main agent reviews results
  7. Verify all tests pass (PILAR 5)

Result: "Auth system built, reviewed, tested. PR ready."
User: *reviews PR*
```

---

## Referensi

- [Anthropic: Best Practices](https://code.claude.com/docs/en/best-practices)
- [Anthropic: Memory](https://code.claude.com/docs/en/memory)
- [Anthropic: Extend Claude Code](https://code.claude.com/docs/en/extend)
- [Hermes Agent](https://github.com/NousResearch/hermes-agent)
- [Awesome Cursor Rules](https://github.com/PatrickJS/awesome-cursorrules)

---

*Dibuat oleh [zufarrizal](https://github.com/zufarrizal) — Juli 2026*
