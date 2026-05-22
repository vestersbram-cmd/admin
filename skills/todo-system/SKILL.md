---
name: todo-system
description: Persistent todo system for managing tasks. Create, list, and complete todos that persist across sessions. Uses infotree for intent clarification when requests are vague, and llm_wiki for durable storage. Execute todos via natural language or /queue integration.
---
# Todo System Skill

## Purpose

A persistent todo system that captures user requests, clarifies vague intent via infotree when needed, stores todos durably in llm_wiki, and executes them one at a time without rushing.

## When to Use

Use this skill when the user:
- Says "I want to...", "I need to...", "Can you...", "Please..." (task-like language)
- Asks "what todos are open/pending?", "what do I need to do?"
- Wants to "mark that done", "complete that todo"
- Says "show me my todos", "list my tasks"
- Asks "what did we complete since last time?"
- Wants to "create a todo for...", "add a task..."

## Natural Language Triggers

The skill automatically detects these patterns:

**Creating todos:**
- "I want to add auth to my API"
- "Fix the bug in the auth module"
- "Can you help me refactor the user model?"
- "Create a todo to update the README"

**Querying todos:**
- "What todos do I have?"
- "Show pending tasks"
- "What's on my list?"
- "Have we completed anything?"

**Completing todos:**
- "Mark that todo done"
- "That auth task is complete"
- "I've finished the refactoring"

## Workflow

### 1. Detect Intent

When user input matches task-like patterns, the skill:
- Extracts the todo content
- Determines if clarification is needed (vague vs specific)

### 2. Clarify if Vague

If the request lacks specifics (no file paths, function names, etc.):
- Start an infotree session: `infotree.api.start(f"Todo: {content}")`
- Ask clarifying questions about scope, approach, files
- On APPROVED: create todo with clarified content
- On DENIED: don't create todo

### 3. Create Todo

```python
from todo_system.api import create_todo

todo = create_todo(
    content="Add JWT authentication to the auth module",
    tags=["auth", "backend"],
    source_infotree_session="it_abc123"  # if clarified
)
```

### 4. Store in Wiki

Todos are stored as llm_wiki journal pages with:
- YAML frontmatter: todo_id, status, timestamps, tags
- Content: task description
- Tags: `todo`, status, user tags

Location: `~/.llm-wiki/todos/pages/todos/`

### 5. Execute When Asked

When user asks to work on a todo ("let's do the next todo", "work on the auth task"):
- Skill calls `get_next_todo()` or finds specific todo by content
- Marks it `in_progress`
- Agent executes the todo content
- On completion, marks it `completed`

## Tools Available

The todo system registers these tools:

- `create_todo(content, tags)` - Create new todo
- `list_todos(status, since_days)` - List todos with filters
- `get_next_todo()` - Get oldest pending todo
- `update_todo_status(todo_id, new_status)` - Mark done/cancelled/in_progress
- `delete_todo(todo_id)` - Remove todo permanently

## Example Workflows

### Workflow 1: Specific Request → Direct Todo

**User:** "Fix the bug on line 42 of auth.py"

**Skill detects:** specific (has file + line number)

**Action:**
```python
create_todo("Fix the bug on line 42 of auth.py", tags=["bugfix"])
```

**Response:** "Created todo todo_abc123: Fix the bug on line 42 of auth.py"

### Workflow 2: Vague Request → Infotree → Todo

**User:** "I want to add auth"

**Skill detects:** vague (no specifics)

**Action:**
```python
from infotree import api as it_api
session = it_api.start("Add auth to the project")
# ... infotree drives clarification ...
# User answers questions about JWT vs session, which routes, etc.
# On APPROVED:
create_todo(
    "Add JWT authentication to API routes with refresh tokens",
    source_infotree_session=session.session_id
)
```

### Workflow 3: Query Todos

**User:** "What todos are open?"

**Action:**
```python
from todo_system.api import get_active_todos, format_todos
todos = get_active_todos()
formatted = format_todos(todos)
```

**Response:** Shows list with status markers:
```
[ ] todo_abc123: Fix bug in auth.py (2025-01-18)
[ ] todo_def456: Add JWT authentication... (2025-01-18)
```

### Workflow 4: Execute a Todo

**User:** "Let's work on the auth todo"

**Skill:**
```python
from todo_system.api import get_todo_by_id, mark_in_progress
# Find the auth todo
todos = list_todos(status="pending")
auth_todo = [t for t in todos if "auth" in t.content.lower()][0]
mark_in_progress(auth_todo.todo_id)
# Agent now works on auth_todo.content
```

**Agent executes the todo content**

**On success:** User or agent marks it completed via `update_todo_status`

## Status Lifecycle

```
user request → [infotree if vague] → PENDING → [user asks to work on it] → IN_PROGRESS → [agent work] → COMPLETED
                                    ↓
                              cancelled by user
```

## Storage Format

Wiki page at `~/.llm-wiki/todos/pages/todos/todo-2025-01-18-abc123.md`:

```yaml
---
title: "Todo: Fix bug in auth.py"
type: journal
tags: [todo, pending, bugfix]
created: 2025-01-18T10:30:00
updated: 2025-01-18T10:30:00
todo_id: todo_abc123
status: pending
confidence: medium
---

## Todo: Fix bug in auth.py

**Status:** pending
**Created:** 2025-01-18 10:30
```

## Implementation Notes

- Todo package: `todo_system/` (models, storage, api, intent)
- Tools: `tools/todo_system_tools.py` (registered via tools.registry)
- Wiki domain: `~/.llm-wiki/todos/`
- No CLI modifications needed - pure skill-based

## References

- `todo_system/__init__.py` - Models (Todo, TodoStatus)
- `todo_system/storage.py` - Wiki persistence layer
- `todo_system/api.py` - Public API (create_todo, list_todos, etc.)
- `todo_system/intent.py` - Intent detection and infotree integration
- `tools/todo_system_tools.py` - Tool definitions for agent
