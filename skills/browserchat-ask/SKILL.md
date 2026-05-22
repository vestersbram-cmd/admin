---
name: browserchat-ask
description: Delegate coding, analysis, or research tasks to external AI providers (ChatGPT, Claude, Gemini, etc.) via browser automation
---
# BrowserChat Ask Skill

Use this skill when you need to delegate a task to an external AI provider — for a second opinion, a more capable model, or parallel work.

## When to use

- Delegating complex coding tasks to a more capable AI provider (GPT-5, Claude Sonnet, etc.)
- Getting a second opinion or review from a different AI model
- Researching codebases or generating documentation
- Generating tests, specs, or implementation plans
- When you need an AI to work on something in the background while you continue

## How it works

The `browserchat ask` command sends a prompt to an AI provider (ChatGPT, Claude, Gemini, Grok, DeepSeek, Kimi, Qwen, Meta AI, AIStudio) via browser automation.

### Key options

| Option | Description |
|---|---|
| `--config <preset>` | Preset mapping: `fast`, `high-intelligence`, `code-project`, `deep-research`, `default` |
| `--model <model>` | Override model (overrides config preset) |
| `--attach <paths...>` | Attach files/directories as context for the AI |
| `--continue-conversation` | Reuse the last conversation for the provider |
| `--conversation-id <id>` | Resume a specific conversation by ID |
| `--new-tab` | Start a fresh browser tab |
| `--no-background` | Run in foreground (response streams to stdout) |
| `--force` | Override an existing lock |

### Modes

**Foreground mode** (`--no-background`): The AI streams its response directly to stdout. Preferred for direct response capture.

**Background mode** (default): The command starts a browser tab and exits immediately, printing a status file path like:
```
chat_history/ask_logs/<timestamp>.status.json
```
Poll this file for completion:
```json
{"status": "running"}
{"status": "completed", "outputFile": "chat_history/ask_logs/<timestamp>.output.md"}
```

### Providers

`chatgpt`, `claude`, `gemini`, `grok`, `deepseek`, `kimi`, `meta`, `qwen`, `aistudio`

## Usage

### Basic delegation (foreground)

```bash
browserchat ask "Refactor this function to use async/await" --config code-project --no-background --force
```

### With file attachments

```bash
browserchat ask "Add unit tests for the database module" --config code-project --attach src/db.ts --no-background --force
```

### Specify a provider and model

```bash
browserchat ask "Write a Python script to parse CSV files" claude --model claude-sonnet-4-20250514 --no-background --force
```

### Background mode (fire-and-forget multiple tasks)

```bash
browserchat ask "Generate API client module" --config code-project --force
browserchat ask "Write README documentation" --config default --force
```

Then poll the status files later:
```bash
cat chat_history/ask_logs/1700000000000.status.json
```

### Continue an existing conversation

```bash
browserchat ask "Fix the issues from the last review" --continue-conversation --no-background --force
browserchat ask "Continue from a specific session" --conversation-id abc123 --no-background --force
```

## Prompt construction

- Make prompts self-contained — the AI provider has zero prior context about your project
- Attach relevant source files, tests, configs, and docs with `--attach`
- Include concrete acceptance criteria in the prompt
- Keep attachments focused — don't overwhelm the provider with too much context

## Common pitfalls

- Long-running tasks may time out — use `timeout_seconds: 600` or higher for complex tasks
- Background mode is default — use `--no-background` if you need the response immediately
- If the browser session is stuck, use `--new-tab --force` to start fresh
- The AI has no context about your project — attach relevant files
