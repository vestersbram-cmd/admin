---
name: greenfield
description: Scaffold new projects from scratch — generate boilerplate, install dependencies, delegate implementation subtasks to AI providers
---
# Greenfield Project Scaffolder Skill

Use this skill when you need to create a new project from scratch: scaffold the boilerplate, install dependencies, and build out the implementation using AI providers for parallel code generation.

## When to use

- "Create a new Python CLI tool called mytool that fetches weather data"
- "Build a FastAPI backend for a todo app in a new package"
- "Generate a TypeScript library with the following API..."
- Any "create a new project" request where scaffolding + AI generation is the right approach
- Starting a new package with `boilerz` and then filling in the implementation

## Workflow

```
Phase 1: Scaffold  →  Phase 2: Understand  →  Phase 3: Delegate & Build  →  Phase 4: Finalize
```

## Phase 1 — Scaffold the project

1. **Determine package name** — from params or extract from the prompt
2. **Check if package exists** — `ls ~/dev/packages/<name>`
3. **Scaffold with boilerz** (if missing):
   ```bash
   cd ~/dev/packages && boilerz <name> -y
   ```
4. **Install dependencies**:
   ```bash
   cd ~/dev/packages/<name> && installz -y
   ```
5. **Explore what was generated** — read the key files:
   - Python: `pyproject.toml`, `src/<name>/__init__.py`, tests
   - TypeScript: `package.json`, `src/index.ts`, `tsconfig.json`
6. **Consult project standards** — check `.admin/standards/` for relevant conventions:
   - Python project: read `.admin/standards/PYTHON.md`, `.admin/standards/FILE_STRUCTURE.md`, and `.admin/standards/pyproject.toml` for structure reference
   - FastAPI project: also read `.admin/standards/FASTAPI.md`
   - Testing: read `.admin/standards/TESTING.md` for expected test patterns
   - README: read `.admin/standards/README_STYLE.md` for README structure

## Phase 2 — Understand with browsernotes

If `browsernotes` is available, query the codebase to understand the scaffolded structure:

```bash
browsernotes query "What is the structure of the <name> package?"
browsernotes query "How is the entry point defined?"
browsernotes query "What testing framework and conventions are set up?"
```

## Phase 3 — Build in parallel

### 1. Start background browserchat tasks early

Fire off well-scoped implementation subtasks to AI providers:

```bash
browserchat ask "Implement a module for interacting with the GitHub API at src/pkg/api.py" --config code-project --force
browserchat ask "Write comprehensive tests for the database module" --config code-project --attach src/pkg/db.py --force
```

These run in background browser tabs. Save the status file paths from the output.

### 2. Work on your own tasks in parallel

While AI providers think:

- Read and adjust scaffolded files to match the requirements
- Create additional project files (entry points, configs, Makefile, CI configs)
- Set up any additional dependencies
- Write the project-level glue code that ties everything together

### 3. Poll and integrate

Check status files periodically:
```bash
cat chat_history/ask_logs/<timestamp>.status.json
```

When `"status": "completed"`, read the output file and integrate the generated code into the project files. Fix imports, adjust to project conventions, and wire things together.

### 4. Good vs bad delegation

**Good candidates for browserchat:**
- "Generate a Python module for parsing CSV data at src/pkg/parser.py"
- "Write pytest tests for the database module at src/pkg/db.py" (attach the source)
- "Draft a README with installation, usage, and example sections"
- "Implement a data validation module with Pydantic schemas"

**Do yourself:**
- Running boilerz or installz
- Fixing import paths or type errors
- Integrating code from multiple sources
- Making cross-file refactoring changes
- Any task requiring complete knowledge of the project state

## Phase 4 — Finalize

1. **Verify** — run a quick sanity check:
   ```bash
   cd ~/dev/packages/<name> && python -c "import <name>"
   # or for TS:
   cd ~/dev/packages/<name> && bun x tsc --noEmit
   ```
2. **Re-install if dependencies changed** — `cd ~/dev/packages/<name> && installz -y`

## Common pitfalls

- **"boilerz failed"**: Check that boilerz is installed (it's typically at `~/.local/bin/boilerz`)
- **"installz had issues"**: Check the output for missing system dependencies
- **"browserchat background task stuck"**: Try `--new-tab --force` to start a fresh session
- **"AI-generated code doesn't fit"**: That's expected — the AI has no context about the scaffolded structure. You'll need to adapt and integrate
- **"Can't read status file"**: Status files are relative to your home directory or `chat_history/` in the project root — use `cat` with the full path
- **"Not sure what to delegate"**: Start with just 1-2 tasks. You can always spawn more. Quality over quantity
