# .admin

Centralized meta-level configuration directory for AI agent skills, tools, and MCP (Model Context Protocol) setup.

This directory lives at the root of the project workspace (`~/.../.admin`) and serves as the single source of truth for agent capabilities — one level above the code itself.

## Structure

```
.admin/
  README.md              # This file
  mcp/                   # MCP tool configurations
    chrome-devtools.json
  skills/                # Agent skills library (22 skills)
    README.md            # Internal skill documentation
    EXCLUSIONS.md        # Rationale for excluded source skills
    <skill-name>/
      SKILL.md           # Skill instructions with YAML frontmatter
  standards/             # Reference standards and conventions (10 files)
    PYTHON.md            # Python coding conventions
    FASTAPI.md           # FastAPI project structure
    ...
```

### `skills/`

A curated collection of **22 procedural guides** that tell AI agents how to perform specific development tasks well. Each skill is a `SKILL.md` file with YAML frontmatter (only `name` and `description` fields).

| Directory | Description |
|-----------|-------------|
| `plan` | Create decision-complete implementation plans before writing code |
| `execute` | Implement features with disciplined, evidence-based execution |
| `review` | Review code changes for correctness, quality, and pattern alignment |
| `debug` | Systematic debugging workflow — build feedback loop, hypothesize, fix |
| `workflow` | Full development lifecycle: context → plan → execute → review → lessons |
| `cleanup` | Simplify and clean uncommitted code changes |
| `cleanz` | Run code quality checks and fix violations in Python packages |
| `greenfield` | Scaffold new projects from scratch with boilerplate generation |
| `brownfield` | Work on existing codebases — explore, understand, delegate, verify |
| `handoff` | Prepare structured handoff summaries when switching tasks |
| `browser-github` | Chrome DevTools browser automation via persistent session for GitHub |
| `browserchat-ask` | Delegate tasks to external AI providers via browser automation |
| `browsernotes` | Sync docs into and query codebase via Google NotebookLM |
| `notebooklm` | Programmatic API for Google NotebookLM (sources, artifacts, downloads) |
| `gmailz` | Multi-account Gmail Manager with search, sync, and caching |
| `sql-admin` | PostgreSQL database operations via sql_admin package |
| `infotree` | Interactive decision-tree system for clarifying user intent |
| `present-options` | Compact visual multiple-choice presenter for quick decisions |
| `todo-system` | Persistent todo system with infotree clarification and wiki storage |
| `llm-wiki` | Minimal LLM Wiki — lightweight markdown knowledge base |
| `skills-maintenance` | Maintain cleanliness and quality in the skills folder |
| `mermaid-create` | Generate error-free Mermaid diagrams for architecture and flows |

See `skills/README.md` for detailed categorization, quality standards, and contributing guidelines.

### `mcp/`

Model Context Protocol configurations for tool integrations used by AI agents.

| File | Description |
|------|-------------|
| `chrome-devtools.json` | Chrome DevTools MCP server config for browser automation |

## Usage

The `.admin` directory is placed at the project root and is automatically discovered by the AI agent system. Skills can be loaded by name to get structured guidance for any development task.

### Quick reference

- **Before coding** → `plan`
- **While coding** → `execute`, `brownfield`, `greenfield`
- **After coding** → `review`, `cleanup`
- **Debugging** → `debug`
- **Research** → `browsernotes`, `browserchat-ask`, `notebooklm`
- **Project hygiene** → `cleanz`, `skills-maintenance`

### For reference standards

Reference standards and conventions for development work. Unlike skills (procedural guides), these are **reference documents** — coding conventions, project structures, and style guidelines to consult when working on specific technologies.

| File | Topic | Applies to |
|------|-------|------------|
| `PYTHON.md` | Python coding conventions and project structure | Python projects |
| `FASTAPI.md` | FastAPI application structure and patterns | FastAPI web apps |
| `TESTING.md` | Testing methodology and conventions | Python test suites |
| `README_STYLE.md` | README formatting and structure | Any project with a README |
| `FILE_STRUCTURE.md` | Standard project layout conventions | Python packages |
| `ANDROID.md` | Android app development conventions | Android/Kotlin projects |
| `ESP-IDF.md` | ESP-IDF embedded development standards | IoT/embedded projects |
| `EXTENSIONS.md` | Extension development guidelines | VS Code or similar extensions |
| `TKINTER_STYLE.md` | Tkinter UI style guide and patterns | Desktop GUI apps |
| `pyproject.toml` | Python project configuration template | Python packages |

**Usage:** Read the relevant file via `read_files` when starting work in a particular domain (e.g., `read_files` with path `.admin/standards/PYTHON.md`). Follow the conventions defined; when a standard conflicts with project-local conventions, project conventions win.

**Standards vs Skills:**

| | Skills (`skills/`) | Standards (`standards/`) |
|---|---|---|
| **Purpose** | Procedural guides for AI agents | Reference docs for humans and AI |
| **Format** | SKILL.md with YAML frontmatter | Plain markdown, no frontmatter |
| **Loading** | Loaded via `skill` tool | Read directly via `read_files` |
| **Example** | "How to debug a bug" | "Python naming conventions" |

*Sourced from `~/dev/standards/` — updated May 2026*

## Syncing

```bash
cd /path/to/your/project/.admin && git pull origin main
```

## Adding Skills

1. Add your skill directory under `skills/<skill-name>/SKILL.md`
2. Frontmatter must contain only `name` and `description`
3. See `skills/skills-maintenance/SKILL.md` → **Skill Format Specification** section for the full specification

## Goals

1. **Centralization** — one source of truth for all agent skills and configurations
2. **Portability** — works across machines via `git clone`
3. **Meta-level** — lives one level above project code, keeping agent config separate
