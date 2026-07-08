# 7 PILAR AI AGENT FOCUS

> Faktor-faktor yang membuat AI coding agent benar-benar fokus dan efektif dalam sebuah project.
> Selain context files (CLAUDE.md, AGENTS.md, .hermes.md), ada 7 pilar lain yang saling melengkapi.

---

## 1. 🧠 SOUL.md — Agent Identity

File yang menentukan **siapa** agent ini, bukan soal project.

```markdown
# ~/.hermes/SOUL.md

You are a senior backend engineer.
You prefer Go over Python.
You write minimal, performant code.
You never use frameworks unless explicitly asked.
You explain trade-offs before making decisions.
```

**Impact:** Agent punya "karakter" yang konsisten lintas session.

| Platform | Lokasi |
|----------|--------|
| Hermes | `~/.hermes/SOUL.md` |
| Claude Code | `~/.claude/CLAUDE.md` (global) |
| Cursor | `~/.cursor/rules/*.mdc` (global) |

---

## 2. 🔧 MCP Servers — External Tools

Kasih agent akses ke tools external sehingga bisa verifikasi sendiri hasilnya.

```bash
# Database queries
claude mcp add postgres -- npx @anthropic-ai/server-postgres --connection-string postgresql://localhost/mydb

# GitHub operations
claude mcp add github -- npx @modelcontextprotocol/server-github

# Browser testing
claude mcp add puppeteer -- npx @anthropic-ai/server-puppeteer
```

**Impact:** Agent bisa query database, test API, cek browser tanpa minta user.

| MCP Server | Fungsi |
|------------|--------|
| PostgreSQL | Query & verify data |
| GitHub | Issues, PRs, code search |
| Puppeteer | Browser testing & screenshots |
| Filesystem | Controlled file access |
| Custom | API internal, monitoring, dll |

---

## 3. 🪝 Hooks — Automation on Events

Automate actions tanpa user intervention. Agent punya "guard rails" otomatis.

```json
// .claude/settings.json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Write(*.py)",
      "hooks": [{"type": "command", "command": "ruff check --fix $CLAUDE_FILE_PATHS"}]
    }],
    "PreToolUse": [{
      "matcher": "Bash",
      "hooks": [{"type": "command", "command": "if echo \"$CLAUDE_TOOL_INPUT\" | grep -q 'rm -rf'; then echo 'Blocked!' && exit 2; fi"}]
    }],
    "Stop": [{
      "hooks": [{"type": "command", "command": "echo 'Task completed at $(date)' >> ~/.claude/activity.log"}]
    }]
  }
}
```

**8 Hook Types:**

| Hook | Kapan Fire | Contoh Penggunaan |
|------|-----------|-------------------|
| `UserPromptSubmit` | Sebelum proses prompt | Input validation, logging |
| `PreToolUse` | Sebelum tool jalan | Security gates, block dangerous commands |
| `PostToolUse` | Setelah tool selesai | Auto-format, run linters |
| `Notification` | Saat permission request | Desktop notifications |
| `Stop` | Saat agent selesai respond | Completion logging |
| `SubagentStop` | Saat subagent selesai | Agent orchestration |
| `PreCompact` | Sebelum context di-compress | Backup session |
| `SessionStart` | Saat session mulai | Load dev context |

---

## 4. 📚 Skills — Reusable Procedures

Simpan workflow yang sering dipakai. Agent tidak reinvent the wheel setiap session.

```markdown
# .claude/skills/add-api-endpoint.md

When asked to add a new API endpoint:
1. Check existing endpoints in src/api/ for patterns
2. Create handler in src/api/handlers/
3. Add route in src/api/router.py
4. Write unit test in tests/api/
5. Run `make test` to verify
6. Update API docs in docs/api.md
```

| Platform | Lokasi Skills |
|----------|---------------|
| Hermes | `~/.hermes/skills/` |
| Claude Code | `.claude/skills/` (project) atau `~/.claude/skills/` (global) |
| Cursor | `.cursor/rules/*.mdc` (path-scoped) |

**Impact:** Agent langsung tahu workflow tanpa dijelaskan ulang.

---

## 5. 🎯 Goal-Oriented Prompting

Jangan kasih task, kasih **GOAL + VERIFICATION**.

### ❌ Prompt Lemah
```
Fix the login bug
```

### ✅ Prompt Kuat
```
Users report login fails after session timeout.
Check src/auth/, especially token refresh.
Write a failing test that reproduces the issue,
then fix it. Verify: run `make test` and ensure
all auth tests pass.
```

### Formula Prompt Efektif

```
[SITUATION] + [WHAT TO DO] + [WHERE TO LOOK] + [HOW TO VERIFY]
```

| Komponen | Contoh |
|----------|--------|
| Situation | "Users report login fails after timeout" |
| What | "Fix the auth bug" |
| Where | "Check src/auth/, especially token refresh" |
| Verify | "Run `make test`, ensure all auth tests pass" |

### Level Verification

| Level | Cara | Kapan Pakai |
|-------|------|-------------|
| Prompt | "Run tests after implementing" | Task sederhana |
| Goal | `/goal` condition + evaluator | Task kompleks |
| Hook | Stop hook blocks until check passes | Autonomous runs |
| Adversarial | Verification subagent checks work | Critical changes |

---

## 6. 📊 Context Window Management

Performance **DROPS** saat context > 70% full. Ini constraint paling penting.

### Thresholds

| Usage | Status | Action |
|-------|--------|--------|
| < 70% | Normal | Full precision |
| 70-85% | Warning | Consider `/compact` |
| > 85% | Danger | Hallucination risk spikes |

### Strategi

```
┌─────────────────────────────────────────────────┐
│         CONTEXT WINDOW MANAGEMENT               │
├─────────────────────────────────────────────────┤
│                                                 │
│  1. COMPACT  → /compact saat context 70%+       │
│                /compact focus on auth logic      │
│                                                 │
│  2. SPLIT    → Session baru untuk task berbeda   │
│                Jangan campur debug + feature     │
│                                                 │
│  3. DELEGATE → Subagents untuk investigasi       │
│                Isolated context, fresh window    │
│                                                 │
│  4. SCOPE    → Path-scoped rules                 │
│                Hanya load yang relevan           │
│                                                 │
│  5. PIPE     → cat file | claude -p "analyze"    │
│                Daripada suruh agent baca file    │
│                                                 │
└─────────────────────────────────────────────────┘
```

### Token Cost Reference

| Action | Approximate Tokens |
|--------|-------------------|
| CLAUDE.md startup | 1,000-3,000 |
| Reading a file | 200-2,000 per file |
| Command output | 100-5,000 per command |
| One conversation turn | 500-2,000 |

---

## 7. 🤖 Subagents & Parallelism

Untuk task besar, delegate ke subagents. Parallel execution dengan isolated contexts.

### Hermes: delegate_task

```python
delegate_task(tasks=[
    {"goal": "Refactor auth module to use JWT", "context": "src/auth/"},
    {"goal": "Write integration tests for API", "context": "tests/api/"},
    {"goal": "Update API documentation", "context": "docs/api.md"}
])
```

### Claude Code: Subagents

```markdown
# .claude/agents/security-reviewer.md
---
name: security-reviewer
description: Security-focused code review
model: opus
tools: [Read, Bash]
---
You are a senior security engineer.
Review code for injection vulnerabilities,
auth flaws, secrets in code, unsafe deserialization.
```

Invoke: `@security-reviewer review the auth module`

### Kapan Pakai Subagent

| Situasi | Pakai Subagent? |
|---------|----------------|
| Task sederhana (< 5 file) | ❌ Langsung saja |
| Investigasi codebase besar | ✅ Fresh context |
| Parallel independent tasks | ✅ Concurrent execution |
| Adversarial review | ✅ Second opinion |
| Task yang butuh context berbeda | ✅ Isolated window |

---

## RINGKASAN: 7 Pilar + Context Files

```
┌──────────────────────────────────────────────────────────────┐
│                    AI AGENT EFFECTIVENESS                     │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  FOUNDATION (sudah dibuat)                                   │
│  ├── Context Files (CLAUDE.md, AGENTS.md, .hermes.md)       │
│  └── Path-scoped rules (.claude/rules/)                      │
│                                                              │
│  7 PILAR TAMBAHAN                                            │
│  ├── 1. SOUL.md          → Agent identity & personality      │
│  ├── 2. MCP Servers      → External tool access              │
│  ├── 3. Hooks            → Automation on events              │
│  ├── 4. Skills           → Reusable procedures               │
│  ├── 5. Goal Prompting   → Clear success criteria            │
│  ├── 6. Context Mgmt     → Keep agent sharp                  │
│  └── 7. Subagents        → Parallel & isolated execution     │
│                                                              │
│  SEMUA SALING MELENGKAPI                                     │
│  Context files = TAHU project                                │
│  7 pilar = TAHU cara kerja yang efektif                      │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## Referensi

- [Anthropic: Best Practices for Claude Code](https://code.claude.com/docs/en/best-practices)
- [Anthropic: Store Instructions and Memories](https://code.claude.com/docs/en/memory)
- [Anthropic: Extend Claude Code](https://code.claude.com/docs/en/extend)
- [Hermes Agent](https://github.com/NousResearch/hermes-agent)
- [Awesome Cursor Rules](https://github.com/PatrickJS/awesome-cursorrules)

---

*Dibuat oleh [zufarrizal](https://github.com/zufarrizal) — Juli 2026*
