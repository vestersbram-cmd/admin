---
name: browsernotes
description: Sync package documentation into Google NotebookLM and query it for architectural understanding and pattern discovery.
---

# BrowserNotes

Sync your project's source code documentation into Google NotebookLM, then ask questions about architecture, APIs, dependencies, and design patterns.

## When to Use

- Before making changes to an unfamiliar part of the codebase — understand the architecture first
- To find which package implements a specific feature
- To understand how parts of the codebase relate to each other
- To discover design patterns, conventions, or API usage
- After scaffolding a new project — to understand what was generated
- After significant code changes — to refresh stale documentation

## Setup: Sync Documentation to NotebookLM

Before querying, documentation must be synced. If no notebook exists yet, the sync will create one automatically.

### Sync all packages

```bash
browsernotes codebase
```

### Sync specific packages only

```bash
browsernotes codebase auth billing api
```

### Full refresh (delete and recreate notebook)

```bash
browsernotes codebase --refresh
```

Use this when:
- The notebook is out of sync with the codebase
- A previous sync failed halfway, leaving an inconsistent state
- Packages have been deleted or renamed since the last sync

### Test with a limited set

```bash
browsernotes codebase --limit 5 --keep
```

The `--keep` flag preserves generated markdown files for debugging.

### Sync notes

- **Browser automation required** — the command opens NotebookLM in a real browser
- **Slow operation** — takes 30-120+ seconds depending on package count
- **Partial syncs**: if a sync fails halfway, run with `--refresh` to start clean

## Usage: Querying

Once documentation is synced, ask questions about your codebase:

### Basic query

```bash
browsernotes query "How does the authentication module work?"
```

### Architecture understanding

```bash
browsernotes query "What is the overall architecture of the project and how do the main packages interconnect?"
```

### Pattern discovery

```bash
browsernotes query "What's the pattern for writing a new CLI command in this codebase?"
```

### Dependency analysis

```bash
browsernotes query "Which packages depend on the auth module?"
```

## Response

The tool returns an AI-generated answer citing relevant sources from your codebase documentation. Results typically include:

- A natural language answer to your question
- References to specific files, functions, or packages
- Code snippets where relevant

## Common Pitfalls

- **Missing notebook**: Run `browsernotes codebase` first to sync documentation
- **Stale answers**: Run `browsernotes codebase --refresh` to update the notebook after significant changes
- **Slow queries**: Browser automation takes 10-60 seconds per query — be patient
- **Vague questions**: Be specific — "How does the auth flow work?" is better than "Tell me about the project"
- **Auth errors**: The CLI has built-in retry but may need manual browser login to NotebookLM if it persists
- **Timeouts during sync**: For large codebases, set `timeout_seconds: 600` or higher. Use `--limit N` to test with fewer packages first
