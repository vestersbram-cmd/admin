# Skill Library

A curated collection of **28 skills** for AI-assisted software development, synthesized from multiple agent frameworks and refined for broad applicability.

## What Is a Skill?

A skill is a procedural guide — a markdown file with YAML frontmatter — that tells an AI agent how to perform a specific task well. Skills live in subdirectories under `~/skillz/skills/`. Each skill has a `SKILL.md` file containing instructions, context, and heuristics.

## Skill Naming

- Directory name = skill name (kebab-case)
- Frontmatter `name:` matches the directory name
- Only `name` and `description` fields are allowed in frontmatter

## Using Skills

Load a skill by name when you need structured guidance for a task:

- **Before coding** → `plan`, `design-thinking`
- **While coding** → `execute`, `brownfield`, `greenfield`
- **After coding** → `review`, `cleanup`, `verify`
- **Debugging** → `debug`
- **Research** → `browsernotes`, `browserchat-ask`, `notebooklm`
- **Project hygiene** → `cleanz`, `fix-type-ignores`, `skills-maintenance`

## Sources

| Source | Count | Skills |
|--------|-------|--------|
| Codebuff | 12 | brownfield, browserchat-ask, browsernotes, cleanup, cleanz, debug, design-thinking, greenfield, meta, review, verify, workflow |
| `~/.agents/skills` | 10 | execute, fix-type-ignores, gmailz, infotree, llm-wiki, mermaid-master-architect, notebooklm, plan, skill-content-format, sql-admin |
| Third-party | 4 | handoff, present-options, skills-maintenance, todo-system |

## Merged Skills

Five skills were synthesized from multiple sources to produce the best of all worlds:

| Skill | Sources Merged | Key Improvements |
|-------|---------------|------------------|
| `plan` | `~/.agents/skills` + another source | Structured markdown template, decision-complete planning heuristics |
| `execute` | `~/.agents/skills` + another source | Evidence-based execution workflow, completion checklist |
| `verify` | codebuff + two other sources | Merged practical gate with structured verification process, quick-reference card |
| `review` | codebuff + two other sources | Merged bug-hunting focus with output format, severity levels, and guardrails |

> **Note:** `cleaner` was later consolidated into `cleanz` to create a single comprehensive code quality skill. `browsernotes-codebase` was consolidated into `browsernotes`.

## Skill Categories

### Development Lifecycle
- `plan` — Decision-complete implementation planning
- `execute` — Evidence-based feature/fix implementation
- `review` — Structured code review for correctness and quality
- `verify` — Fresh verification before completion claims
- `cleanup` — Simplify and clean uncommitted changes
- `workflow` — Full lifecycle: gather → plan → execute → review → learn

### Debugging & Quality
- `debug` — Systematic root-cause debugging
- `cleanz` — Run code quality checks and fix violations in Python packages
- `fix-type-ignores` — Remove unnecessary `# type: ignore` comments
- `test-driven-development` — TDD methodology with pytest

### Architecture & Design
- `design-thinking` — Tradeoff analysis and architectural decisions
- `greenfield` — Scaffold new projects from scratch
- `brownfield` — Work on existing codebases
- `meta` — Project-wide knowledge for the Codebuff codebase

### Research & External Tools
- `browser` — Chrome DevTools automation for GitHub sessions
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
- `mermaid-master-architect` — Mermaid diagram generation and refinement
- `skills-maintenance` — Evaluate, consolidate, and prune skills
- `skill-content-format` — Guidelines for creating new skills

## Quality Standards

Every skill in this library:

- Has valid YAML frontmatter with **only** `name` and `description`
- Has consistent markdown formatting
- Contains actionable instructions, not just vague advice
- Has no duplicate content with other skills in the library
- Uses the directory name as its canonical identifier

## Adding or Modifying Skills

See `skill-content-format/SKILL.md` for the full specification on creating new skills. When adding a skill:

1. Choose a unique kebab-case directory name
2. Write `SKILL.md` with proper YAML frontmatter (`name` and `description` only)
3. Ensure no overlap with existing skills
4. Run `skills-maintenance` periodically to prevent bloat

---

*Last updated: 2026*
