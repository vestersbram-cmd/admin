# Skill Library

A curated collection of **22 skills** for AI-assisted software development, synthesized from multiple agent frameworks and refined for broad applicability.

## What Is a Skill?

A skill is a procedural guide — a markdown file with YAML frontmatter — that tells an AI agent how to perform a specific task well. Skills live in subdirectories under `.admin/skills/`. Each skill has a `SKILL.md` file containing instructions, context, and heuristics.

## Skill Naming

- Directory name = skill name (kebab-case)
- Frontmatter `name:` matches the directory name
- Only `name` and `description` fields are allowed in frontmatter

## Using Skills

Load a skill by name when you need structured guidance for a task:

- **Before coding** → `plan`
- **While coding** → `execute`, `brownfield`, `greenfield`
- **After coding** → `review`, `cleanup`
- **Debugging** → `debug`
- **Research** → `browsernotes`, `browserchat-ask`, `notebooklm`
- **Project hygiene** → `cleanz`, `skills-maintenance`

## Sources

| Source | Count | Skills |
|--------|-------|--------|
| Codebuff | 10 | brownfield, browserchat-ask, browser-github, browsernotes, cleanup, cleanz, debug, greenfield, review, workflow |
| `~/.agents/skills` | 8 | execute, gmailz, infotree, llm-wiki, mermaid-create, notebooklm, plan, sql-admin |
| Third-party | 4 | handoff, present-options, skills-maintenance, todo-system |

## Merged Skills

Four skills were synthesized from multiple sources to produce the best of all worlds:

| Skill | Sources Merged | Key Improvements |
|-------|---------------|------------------|
| `plan` | `~/.agents/skills` + `design-thinking` | Structured markdown template, decision-complete planning heuristics, design thinking phases |
| `execute` | `~/.agents/skills` + another source | Evidence-based execution workflow, completion checklist |
| `review` | codebuff + `verify` | Merged verification gate, structured review output, severity levels, guardrails, and completion protocol |

> **Note:** `cleaner` was later consolidated into `cleanz`. `browsernotes-codebase` was consolidated into `browsernotes`. `design-thinking` was absorbed into `plan`. `verify` was absorbed into `review`.

## Skill Categories

### Development Lifecycle
- `plan` — Decision-complete implementation planning
- `execute` — Evidence-based feature/fix implementation
- `review` — Structured code review for correctness and quality
- `cleanup` — Simplify and clean uncommitted changes
- `workflow` — Full lifecycle: gather → plan → execute → review → learn

### Debugging & Quality
- `debug` — Systematic root-cause debugging
- `cleanz` — Run code quality checks and fix violations in Python packages

### Architecture & Design
- `greenfield` — Scaffold new projects from scratch
- `brownfield` — Work on existing codebases

### Research & External Tools
- `browser-github` — Chrome DevTools browser automation via persistent session for GitHub
- `browserchat-ask` — Delegate tasks to external AI providers
- `browsernotes` — Sync docs into and query codebase via NotebookLM
- `notebooklm` — Programmatic NotebookLM API
- `sql-admin` — PostgreSQL operations via sql_admin

### Communication & Organization
- `handoff` — Prepare context summaries for next agent
- `infotree` — Structured multiple-choice intent clarification
- `present-options` — Compact visual choice presenter
- `todo-system` — Persistent task management

### Specialized
- `gmailz` — Multi-account Gmail manager
- `llm-wiki` — Lightweight markdown knowledge base
- `mermaid-create` — Mermaid diagram generation and refinement
- `skills-maintenance` — Evaluate, consolidate, and prune skills

## Quality Standards

Every skill in this library:

- Has valid YAML frontmatter with **only** `name` and `description`
- Has consistent markdown formatting
- Contains actionable instructions, not just vague advice
- Has no duplicate content with other skills in the library
- Uses the directory name as its canonical identifier

## Adding or Modifying Skills

When adding a skill:

1. Choose a unique kebab-case directory name
2. Write `SKILL.md` with proper YAML frontmatter (`name` and `description` only) — see `skills-maintenance` → **Skill Format Specification** section
3. Ensure no overlap with existing skills
4. Run `skills-maintenance` periodically to prevent bloat

---

*Last updated: 2026*
