---
name: new-project-ai-context
description: "Auto-setup AI context files (CLAUDE.md, AGENTS.md, .hermes.md) for new projects to maximize AI coding agent effectiveness."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [ai-context, project-setup, claude-md, agents-md, coding-agents, productivity]
    related_skills: [claude-code, hermes-agent, codex, opencode]
---

# New Project AI Context Setup

Automatically create AI context files when starting a new project. These files make AI coding agents (Hermes, Claude Code, Codex, Cursor, OpenCode) significantly more effective by providing persistent project knowledge.

## When to Use

- Starting a new project from scratch
- Cloning an existing repo that lacks AI context files
- User says "project baru", "new project", "setup project", or similar
- User asks to improve AI agent effectiveness on a project

## Files to Create

### Priority Order (create all that apply)

1. **CLAUDE.md** — Universal, works with Claude Code, importable by Hermes
2. **AGENTS.md** — Portable across all agents (Hermes, Claude Code, Codex, OpenCode)
3. **.hermes.md** — Hermes-specific, hierarchical (walks up to git root)
4. **.cursor/rules/*.mdc** — Cursor-specific path-scoped rules (if user uses Cursor)

### Cross-Agent Compatibility

| File | Hermes | Claude Code | Codex | Cursor | OpenCode |
|------|--------|-------------|-------|--------|----------|
| CLAUDE.md | via import | ✓ native | reads | reads | reads |
| AGENTS.md | ✓ native | via import | ✓ native | — | ✓ native |
| .hermes.md | ✓ native | — | — | — | — |
| .cursor/rules/*.mdc | — | — | — | ✓ native | — |

**Best strategy:** Create AGENTS.md as the portable base, then CLAUDE.md that imports it with Claude-specific additions.

## Template: AGENTS.md

```markdown
# {PROJECT_NAME}

## Overview
- {one-line description}
- Tech stack: {framework, language, database, etc.}

## Architecture
- {key architectural decisions}
- {directory structure highlights}

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
- {environment quirks}
- {required env vars}

## Key Directories
- `src/` — {purpose}
- `tests/` — {purpose}
- `docs/` — {purpose}
```

## Template: CLAUDE.md

```markdown
# {PROJECT_NAME}

@AGENTS.md

## Claude-Specific Instructions
- Use plan mode for changes affecting 3+ files
- Always run tests after modifications
- Prefer {specific patterns for this project}
- {any Claude-specific workflow rules}
```

## Template: .hermes.md

```markdown
# {PROJECT_NAME}

Hermes: when working in this repo, follow these rules.

## Build
- {build commands}
- {test commands}

## Style
- {code style rules}
- {conventions specific to this project}

## Workflow
- {workflow rules}
- {verification steps}
```

## Setup Procedure

### Step 1: Detect Project Type
```bash
# Check for existing project indicators
ls package.json pyproject.toml Cargo.toml go.mod Makefile Dockerfile 2>/dev/null
cat package.json | python3 -c "import json,sys; d=json.load(sys.stdin); print(d.get('name',''), d.get('scripts',{}))"
```

### Step 2: Detect Existing Context Files
```bash
ls CLAUDE.md AGENTS.md .hermes.md .cursorrules .claude/ .cursor/ 2>/dev/null
```

### Step 3: Generate Context Files
Based on detected tech stack, fill in the templates above with:
- Real build commands from package.json / Makefile / pyproject.toml
- Actual directory structure
- Real test commands
- Specific framework conventions

### Step 4: Verify
- Ensure files are under 200 lines each
- Check that build/test commands actually work
- Confirm no contradictions between files

## What Makes Context Files IMPACTFUL

Based on Anthropic's internal data and community research:

### HIGH IMPACT (always include)
1. **Build/test/lint commands** — Agent can't guess these
2. **Code style rules that DIFFER from defaults** — Saves correction cycles
3. **Verification commands** — Enables self-correcting loops
4. **Architecture decisions** — Prevents wrong approaches
5. **Common gotchas** — Prevents repeated mistakes
6. **Required env vars** — Prevents setup failures

### MEDIUM IMPACT (include when relevant)
7. **Path-scoped rules** — .claude/rules/ for monorepos
8. **Git workflow** — Branch naming, PR conventions
9. **Testing patterns** — How to write/run specific tests
10. **Import style** — ES modules vs CommonJS, etc.

### LOW IMPACT / SKIP
- Standard language conventions (agent already knows)
- Detailed API docs (link instead)
- Self-evident practices ("write clean code")
- Info that changes frequently
- Long explanations or tutorials

## Key Rules

1. **Keep it concise** — Each line: "Would removing this cause mistakes?" If not, cut it.
2. **Be specific** — "Use 2-space indentation" not "Format code properly"
3. **No contradictions** — Review periodically for conflicting rules
4. **Check into git** — Team shares the value
5. **Test changes** — Observe if agent behavior actually shifts
6. **Use imports** — CLAUDE.md can @import AGENTS.md to avoid duplication

## Pitfalls

1. **Bloated files cause agents to IGNORE instructions** — Keep under 200 lines
2. **Vague rules are useless** — "Be careful" → "Always validate user input"
3. **Contradicting rules confuse agents** — Review and prune regularly
4. **Forgetting to update** — Treat context files like code, maintain them
5. **Not using path-scoped rules in monorepos** — Loads irrelevant context
