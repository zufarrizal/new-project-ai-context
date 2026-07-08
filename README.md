<div align="center">

# 🧠 new-project-ai-context

### AI Context Files That Actually Work

**Stop repeating yourself to AI agents. Start every project with context that makes them 10x more effective.**

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](#contributing)

</div>

---

## 🤔 Why This Exists

Every time you start a new project, AI coding agents (Claude Code, Hermes, Cursor, Codex, OpenCode) begin with **zero context**. They don't know your build commands, your code style, your testing setup, or your project's gotchas.

You end up explaining the same things **over and over**.

This skill fixes that. It auto-generates research-backed AI context files that make agents immediately productive on your project.

---

## 📊 What Makes Context Files Impactful?

Based on [Anthropic's internal data](https://code.claude.com/docs/en/best-practices) and community research:

| Impact | What to Include | Example |
|--------|----------------|---------|
| 🔴 **HIGH** | Build/test commands agent can't guess | `make test`, `npm run lint` |
| 🔴 **HIGH** | Code style rules that DIFFER from defaults | "Use 2-space indent, not 4" |
| 🔴 **HIGH** | Verification commands for self-correction | "Run `pytest` after changes" |
| 🔴 **HIGH** | Non-obvious gotchas | "Env var `DB_URL` required for tests" |
| 🟡 **MEDIUM** | Architecture decisions | "Uses CQRS pattern, see src/commands/" |
| 🟡 **MEDIUM** | Git workflow conventions | "Branch: feat/*, fix/*, chore/*" |
| ⚪ **SKIP** | Standard conventions agent already knows | "Write clean code" |
| ⚪ **SKIP** | Detailed API docs | Link instead |

> **Key insight:** Agents ignore bloated files. Keep each file **under 200 lines**. Every line should pass the test: *"Would removing this cause the agent to make mistakes?"*

---

## 📁 Files Generated

### `AGENTS.md` — Portable, works with ALL agents
```
Hermes ✅ | Claude Code ✅ | Codex ✅ | OpenCode ✅ | Cursor ✅
```
The universal base file. Contains build commands, code standards, testing setup, and project conventions.

### `CLAUDE.md` — Claude Code optimized
```
Claude Code ✅ | Cursor ✅ (via import)
```
Imports `AGENTS.md` and adds Claude-specific instructions (plan mode rules, verification workflows).

### `.hermes.md` — Hermes Agent specific
```
Hermes ✅
```
Hierarchical file that walks up to git root. Good for monorepo overrides.

### `.cursor/rules/*.mdc` — Cursor path-scoped rules
```
Cursor ✅
```
Rules that only load when editing specific file types (e.g., frontend rules load only when editing `.tsx`).

---

## 🚀 Quick Start

### As a Hermes Skill
```bash
hermes skills install zufarrizal/new-project-ai-context
```

### Manual Usage
Just say to your AI agent:
> "Setup AI context for this project"

The agent will:
1. **Detect** your tech stack (package.json, pyproject.toml, Cargo.toml, etc.)
2. **Generate** context files filled with real commands from your project
3. **Verify** that build/test commands actually work

---

## 📋 Template Preview

<details>
<summary><b>AGENTS.md</b> — Click to expand</summary>

```markdown
# My Project

## Overview
- REST API for user management
- Tech stack: FastAPI, SQLAlchemy, PostgreSQL, Redis

## Build & Run
- `make dev` — start dev server on :8000
- `make test` — run full test suite
- `make lint` — ruff + mypy

## Code Standards
- Type hints on ALL public functions
- Google-style docstrings
- 4-space indentation
- No wildcard imports

## Testing
- Framework: pytest with xdist
- Run single: `pytest tests/test_auth.py -v`
- Coverage target: 90%

## Common Gotchas
- `DATABASE_URL` env var required for tests
- Redis must be running for session tests
- Migrations use Alembic, not raw SQL

## Key Directories
- `src/api/` — FastAPI route handlers
- `src/models/` — SQLAlchemy models
- `src/services/` — Business logic
```
</details>

<details>
<summary><b>CLAUDE.md</b> — Click to expand</summary>

```markdown
# My Project

@AGENTS.md

## Claude-Specific Instructions
- Use plan mode for changes affecting 3+ files
- Always run `make test` after modifications
- Prefer async/await over sync in API handlers
- Write failing test first, then fix (TDD)
```
</details>

<details>
<summary><b>.hermes.md</b> — Click to expand</summary>

```markdown
# My Project

Hermes: when working in this repo, follow these rules.

## Build
- `make dev` — start dev server
- `make test` — run full test suite
- Use `uv run` for Python, not `pip install`

## Style
- Prefer `pathlib.Path` over `os.path`
- No `print()` in production code — use `logger`

## Workflow
- Always run tests before declaring done
- Create PR with descriptive title
```
</details>

---

## 🏗️ How It Works

```
┌─────────────────────────────────────────────────────────┐
│                    NEW PROJECT                           │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  1. DETECT     → Scan package.json, pyproject.toml,     │
│                  Makefile, directory structure            │
│                                                         │
│  2. GENERATE   → Fill templates with REAL commands       │
│                  Extract actual build/test/lint cmds     │
│                                                         │
│  3. VERIFY     → Test that commands actually work        │
│                  Confirm no contradictions                │
│                                                         │
│  4. DEPLOY     → AGENTS.md  (portable base)             │
│                  CLAUDE.md  (imports + Claude rules)     │
│                  .hermes.md (Hermes hierarchical)        │
│                  .cursor/rules/ (path-scoped, optional)  │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## 🎯 The Science Behind It

### Context Window = Most Important Resource

> *"Claude's context window fills up fast, and performance degrades as it fills."*
> — [Anthropic Best Practices](https://code.claude.com/docs/en/best-practices)

Effective context files:
- **Reduce token waste** — Agent doesn't explore to discover build commands
- **Enable self-correction** — Verification commands close the loop
- **Prevent repeated mistakes** — Gotchas are documented once
- **Save your time** — No more explaining the same things each session

### Path-Scoped Rules (Monorepo Power)

Instead of one massive file, split rules by directory:
```
.claude/rules/
├── frontend.md    → Only loads when editing src/web/
├── api.md         → Only loads when editing src/api/
├── database.md    → Only loads when editing src/db/
└── testing.md     → Only loads when editing tests/
```

Each file is **only injected when relevant**, keeping context lean.

---

## 🤝 Compatible Agents

| Agent | AGENTS.md | CLAUDE.md | .hermes.md | .cursor/rules |
|-------|-----------|-----------|------------|---------------|
| **Hermes** | ✅ Native | ✅ Import | ✅ Native | — |
| **Claude Code** | ✅ Import | ✅ Native | — | — |
| **Cursor** | — | ✅ Read | — | ✅ Native |
| **Codex** | ✅ Native | ✅ Read | — | — |
| **OpenCode** | ✅ Native | ✅ Read | — | — |
| **Cline** | ✅ Read | ✅ Read | — | — |
| **Windsurf** | ✅ Read | ✅ Read | — | — |

---

## 📖 Best Practices

### DO ✅
- Keep files under **200 lines**
- Be **specific**: "Use 2-space indent" not "Format properly"
- Include **verification commands**: `npm test`, `make lint`
- Document **non-default** style rules only
- Check into **git** for team sharing
- **Review and prune** regularly

### DON'T ❌
- Write obvious things ("write clean code")
- Include standard language conventions
- Paste detailed API docs (link instead)
- Add info that changes frequently
- Exceed 200 lines (agents start ignoring rules)
- Contradict rules between files

---

## 🛠️ Contributing

PRs welcome! Here's how:

1. Fork this repo
2. Create a branch: `git checkout -b feat/amazing-feature`
3. Commit: `git commit -m 'feat: add amazing feature'`
4. Push: `git push origin feat/amazing-feature`
5. Open a PR

### Ideas for Contributions
- Templates for more tech stacks (Rust, Go, Flutter, etc.)
- Path-scoped rule templates for common architectures
- Integration with more AI agents
- Real-world examples from your projects

---

## 📜 License

MIT — use it however you want. See [LICENSE](LICENSE).

---

## 🙏 Credits

- [Anthropic](https://docs.anthropic.com) — Claude Code best practices research
- [Hermes Agent](https://github.com/NousResearch/hermes-agent) — Skill system & context file hierarchy
- [Awesome Cursor Rules](https://github.com/PatrickJS/awesome-cursorrules) — Cursor .mdc patterns
- The AI coding community — for sharing what actually works

---

<div align="center">

**Made with 🧠 by [zufarrizal](https://github.com/zufarrizal)**

*Because every project deserves an AI agent that actually knows what it's doing.*

</div>
