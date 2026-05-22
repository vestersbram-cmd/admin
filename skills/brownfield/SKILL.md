---
name: brownfield
description: Work on existing codebases — explore structure, understand architecture, delegate implementation subtasks, and verify results
---
# Brownfield Codebase Agent Skill

Use this skill when you need to work on an existing project: analyze its structure, understand the code, make changes, and verify everything passes.

## When to use

- "Add a new CLI command to my existing Python package mytool"
- "Refactor the authentication module in my-project"
- "Fix the bugs in the billing module of the invoice package"
- "Add tests for the database layer in my_project"
- Any "work on this existing project" request

## Workflow overview

```
Phase 1: Explore  →  Phase 2: Understand  →  Phase 3: Plan & Delegate  →  Phase 4: Verify
```

## Phase 1 — Explore the project

1. **Verify the project exists** — check that the project directory is accessible
2. **Read root configs** — `package.json`, `pyproject.toml`, `Cargo.toml`, `Makefile`, `tsconfig.json`, etc.
3. **Explore the directory tree** — list files to understand the structure:
   ```bash
   find <projectDir> -maxdepth 4 -type f | head -80
   ```
4. **Read relevant source files** — understand existing patterns, naming conventions, and architecture before making changes

## Phase 2 — Understand with browsernotes

If `browsernotes` is available and codebase docs are synced, use it to get high-level answers:

```bash
browsernotes query "What does the auth module do?"
browsernotes query "Which packages implement rate limiting?"
browsernotes query "How are database migrations handled?"
browsernotes query "What's the pattern for writing a new CLI command?"
```

This gives you architectural understanding faster than reading every file.

## Phase 3 — Plan & implement

### Direct implementation (do yourself)

- Quick edits and targeted fixes
- Cross-file refactoring that requires knowing the full project state
- Integrating code from multiple sources
- Fixing imports, type errors, and wiring things together
- Running verification commands

### Delegation with browserchat (for complex subtasks)

For well-scoped, self-contained tasks, delegate to an external AI provider:

```bash
browserchat ask "Generate a Python module for parsing YAML config at src/pkg/config.py" --config code-project --attach src/pkg/__init__.py --no-background --force
```

Good candidates:
- Generating new modules, classes, or functions
- Writing tests for existing modules (attach the source)
- Drafting documentation or README updates
- Researching library choices

Bad candidates (do yourself):
- Fixing import paths or type errors
- Integrating code from multiple browserchat outputs
- Small, targeted edits
- Any task requiring deep knowledge of the complete project state

## Phase 4 — Verify

After changes, run the project's verification commands:

- **Typecheck**: `bun x tsc --noEmit` (TypeScript), `mypy .` (Python), etc.
- **Tests**: `bun test`, `pytest`, `cargo test`, etc.
- **Lint**: `eslint`, `ruff`, `clippy`, etc.

Fix any issues found and confirm everything passes.

## Common pitfalls

- **"We don't have a browsernotes notebook"**: Run `browsernotes codebase` first, or skip Phase 2
- **"browserchat failed"**: Try `--new-tab --force` to start fresh, or increase timeout
- **"The project has no tests"**: Add basic tests yourself rather than spending time setting up a full test framework
- **"Don't know the project conventions"**: Read a few existing source files first — they'll show you the patterns
- **"Parallel browserchat tasks"**: Start 2-3 background tasks early, then work on your own tasks while they run
