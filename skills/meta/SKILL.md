---
name: meta
description: Broad project-level implementation and validation heuristics for this codebase
---
# Meta: Project-Wide Knowledge

This skill captures general knowledge about the Codebuff codebase that applies across all development work. It's the default skill to consult when no topic-specific skill fits.

## Repository Architecture

### Monorepo Structure

This is a TypeScript monorepo using Bun workspaces. The package architecture:

```
cli/        → TUI client (OpenTUI + React). Entry: src/index.tsx → app.tsx → chat.tsx
sdk/        → Public JS/TS SDK (@codebuff/sdk). Entry: src/client.ts → src/run.ts
agents/     → Built-in agents shipped with Codebuff (base2, editor, thinker, reviewer, etc.)
common/     → Shared types, tools, schemas, constants. No dependencies (leaf package).
packages/
  agent-runtime/ → Core agent loop: LLM inference, tool execution, handleSteps generators
  code-map/      → Tree-sitter code parser for read_subtree
freebuff/   → Free tier product (separate CLI binary)
scripts/    → Dev tooling, analytics, service management
buffz/      → Python infrastructure scripts (chat logs, instances, fingerprinting)
```

### Key Entry Points

- **CLI start**: `cli/src/index.tsx`
- **SDK run**: `sdk/src/run.ts` → `run()` → `runOnce()` → `callMainPrompt()`
- **Agent loop**: `packages/agent-runtime/src/run-agent-step.ts` → `loopAgentSteps()`
- **Main prompt**: `packages/agent-runtime/src/main-prompt.ts` → `mainPrompt()`
- **Model selection**: `sdk/src/impl/model-provider.ts` (ChatGPT OAuth or Codebuff backend/OpenRouter)

### Development Commands

- `bun start-cli` — Start the CLI (`bun --cwd cli dev`)
- `bun typecheck` — Run all type checks (via `bun --filter='*' run typecheck`)
- `bun test` — Run all tests
- `bun run --cwd <workspace> <script>` — Run a script in a specific workspace. If Bun prints global run help instead, re-check flag order or command shape.
- `infisical run --env=prod -- bun scripts/<name>.ts` — Run against production

### CLI Validation Heuristics

- When validating CLI changes, always run a non-effectful command path first (e.g. `--help`) before any command that could trigger external side effects.
- For tightly scoped edits, pair runtime smoke-checks with `git diff -- <file>` to verify no unintended spillover.
- For SDK-driven agent evaluation, persist both structured run artifacts and raw tmux capture paths so you can compare event-level behavior against what the CLI actually displayed.
- For SDK-driven before/after comparisons, keep prompts, logging granularity, and timeout conditions fixed; otherwise event-count, cost, and duration deltas are too noisy to trust.

### Logs

CLI-related logs go to `debug/browser-agent-traces/`.

### Debugging Approach

When static code analysis and tracing through the codebase isn't enough to find a bug:

1. **Add targeted logging** to the suspected code path — log all key decision variables (inputs, intermediate booleans, outputs) in a single structured log line so you can see exactly why a code path was taken.
2. **Reproduce the issue live** — e.g. via the codebuff-local-cli tmux agent for CLI bugs.
3. **Clean up debug logging** after the issue is found — don't leave it in the codebase.


## Agent & Skill System

### Agent Definitions

Agents live in two places:
- `agents/` — Built-in agents shipped with Codebuff (loaded by the runtime)
- `.agents/` — Project-local agents and skills

Agent definitions use `AgentDefinition` or `SecretAgentDefinition` types from `agents/types/`. 
Each has: `id`, `publisher`, `displayName`, `model`, and optionally `handleSteps` (generator function for programmatic agents).

### Skill System

Skills are SKILL.md files in `.agents/skills/<name>/SKILL.md` with YAML frontmatter:
```yaml
---
name: <skill-name>
description: <one-line description>
---
```

Key rules:
- **Directory name must match `name` field** in frontmatter — the loader validates this
- Skills are loaded from `{cwd}/.agents/skills/` and `~/.agents/skills/`
- Skills provide **instructions for the base agent to follow**, not executable code
- Use the `skill` tool to load a skill by name

### Free Mode Restrictions

Some agents are restricted in free mode via `FREE_MODE_AGENT_MODELS` allowlist. The available agents in free mode are:
- `base2-free`, `base2-free-kimi`, `base2-free-deepseek`, `base2-free-deepseek-flash`
- `file-picker`, `file-picker-max`, `file-lister`
- `researcher-web`, `researcher-docs`
- `browser-use`, `basher`, `tmux-cli`
- Select code reviewers and `thinker-gemini`

Most orchestrators (base2, editor, thinker, etc.) and expensive models are NOT in free mode.
The `@AgentName` mention workflow only works for agents in the allowlist.

## Code Editing Conventions

### Tool Usage

- **Prefer `str_replace` over `write_file`** — more efficient for targeted changes, better feedback
- **Use `write_file` only for new files** or when rewriting the entire file
- **Read before edit** — always read relevant files before modifying them
- **Use `list_directory` and `glob` directly** for searching and exploring the codebase
- **Spawn multiple agents in parallel** for independent context-gathering tasks

### Editing Principles

- **Surgical changes**: Modify as little code as possible. Each change should do exactly one thing.
- **Pattern matching**: Follow existing code patterns (style, naming, typing, architecture)
- **No speculative features**: Don't add parameters or hooks "for future use"
- **Verify library availability**: Check `package.json` before assuming a library is available
- **Refactoring awareness**: When modifying exported symbols, update all references
- **Cleanup**: Remove debug artifacts, unused imports, and dead code after changes

### Agent Spawning

- Spawn context-gathering agents before making edits (file pickers, code searchers, researchers)
- Sequence dependent agents properly (don't spawn editors before context is gathered)
- Each spawn should have a clear purpose — don't spawn agents unnecessarily

## Documentation & Knowledge

### Where to Find Documentation

- `docs/` directory — Detailed architecture, request flow, error schema, testing docs
- `AGENTS.md` — Project overview, conventions, recommended reading
- `.agents/skills/` — Reusable skill files with project-specific knowledge
- `knowledge.md` files — Scattered through the codebase for directory-specific context

### Request Flow Overview

```
CLI prompt → SDK run() → agent-runtime loopAgentSteps() → LLM call → tool execution (local) → response
```

Tool calls execute **locally on the user's machine** (file edits, terminal, code search).

### Testing Patterns

- **DI over mocking** — Use contract types (`common/src/types/contracts/`) for testable dependencies
- **tmux CLI testing** — Interactive CLI testing via tmux sessions
- E2E tests at `agents/e2e/` and `freebuff/e2e/`
- Unit tests co-located with source files or in `__tests__/` directories

## Common Pitfalls

| Situation | Likely issue | What to do |
|---|---|---|
| `bun run` shows global help | Wrong flag order or command shape | Re-check the command syntax |
| Typecheck fails after changes | Exported symbol renamed without updating references | Search for all references to the symbol |
| Agent not responding to `@name` | Agent not in free mode allowlist | Use the skill tool instead of @mention |
| Skill not loading | Directory name != name field in frontmatter | Check the YAML name matches the dir name |
| Test fails after config change | Config changed behavior of unrelated tests | Re-run full test suite, not just your tests |
| Session state lost | Not passing `previousRun` to next `client.run()` | The CLI handles this automatically; manual invocations must pass it |
