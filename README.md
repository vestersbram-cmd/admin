# skillz

Centralized, public repository of agent skills for Buffy, Hermes, and related AI systems.

## Structure

```
skills/
  <skill-name>/      # each skill is a directory with a single SKILL.md file
    SKILL.md
```

All 28 skills live flat under `skills/` — no category nesting.

## Format

Each `SKILL.md` follows this convention:

```yaml
---
name: <skill-name>
description: <one-line description of what this skill does>
---
```

Only two frontmatter fields: `name` (matches the directory) and `description`.

## Skills

| Directory | Description |
|-----------|-------------|
| `plan` | Create decision-complete implementation plans before writing code |
| `execute` | Implement features with disciplined, evidence-based execution |
| `review` | Review code changes for correctness, quality, and pattern alignment |
| `verify` | Verify work before claiming completion — evidence before claims |
| `debug` | Systematic debugging workflow — build feedback loop, hypothesize, fix |
| `design-thinking` | Architectural and design decisions, tradeoff analysis, critique |
| `workflow` | Full development lifecycle: context → plan → execute → review → lessons |
| `cleanup` | Simplify and clean uncommitted code changes |
| `cleanz` | Run code quality checks and fix violations in Python packages |
| `fix-type-ignores` | Find and fix unnecessary `# type: ignore` comments in Python |
| `test-driven-development` | Red-green-refactor methodology for writing tests first |
| `greenfield` | Scaffold new projects from scratch with boilerplate generation |
| `brownfield` | Work on existing codebases — explore, understand, delegate, verify |
| `handoff` | Prepare structured handoff summaries when switching tasks |
| `browser` | Chrome DevTools browser automation via persistent session |
| `browserchat-ask` | Delegate tasks to external AI providers via browser automation |
| `browsernotes` | Sync docs into and query codebase via Google NotebookLM |
| `notebooklm` | Programmatic API for Google NotebookLM (sources, artifacts, downloads) |
| `gmailz` | Multi-account Gmail Manager with search, sync, and caching |
| `sql-admin` | PostgreSQL database operations via sql_admin package |
| `infotree` | Interactive decision-tree system for clarifying user intent |
| `present-options` | Compact visual multiple-choice presenter for quick decisions |
| `todo-system` | Persistent todo system with infotree clarification and wiki storage |
| `llm-wiki` | Minimal LLM Wiki — lightweight markdown knowledge base |
| `skill-content-format` | Guidelines for creating skills with proper frontmatter and structure |
| `skills-maintenance` | Maintain cleanliness and quality in the skills folder |
| `meta` | Project-level knowledge about the Codebuff codebase |
| `mermaid-master-architect` | Generate error-free Mermaid diagrams for architecture and flows |

## Goals

1. **Centralization** — one source of truth for all agent skills
2. **Portability** — skills work across machines via `git clone`
3. **Quality** — curated subset, no duplication, clean frontmatter only

## Syncing

```bash
cd ~/skillz && git pull origin main
```

## Adding Skills

1. Fork the repo
2. Add your skill directory under `skills/<skill-name>/SKILL.md`
3. Frontmatter must contain only `name` and `description`
4. PR against `main`
