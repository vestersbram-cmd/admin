---
name: handoff
description: Prepare a structured handoff summary when switching tasks, agents, or pausing long-running work.
---

# Handoff

Create a handoff summary so the next session can pick up exactly where you left off without losing context.

## When to Use

- Switching from one task to another within the same project
- Pausing work that will resume later
- Transferring work to a different agent or team member
- Ending a session with incomplete work
- Before a break in a multi-step implementation

## What to Capture

### 1. Current State (one sentence)
A single-line description of exactly where things stand right now.

### 2. Completed
- [x] Items that are fully done
- [x] Files that were created or modified
- [x] Decisions that were made

### 3. In Progress
- [ ] Items that are partially done with current status
- [ ] Files with uncommitted or unsaved changes
- [ ] Commands that were run and their output

### 4. Next Steps (priority order)
1. The very next thing to do
2. What comes after that
3. Any follow-up tasks

### 5. Blockers
- Any errors, failures, or open questions preventing progress
- Missing information or decisions needed from others

### 6. Context
- Relevant file paths and line numbers
- Design decisions and their rationale
- Gotchas or non-obvious behaviors discovered
- Links to related issues, PRs, or documentation

## Example Handoff

```
Current State: Implementing user authentication — login flow is complete,
registration is partially implemented with validation pending.

Completed:
- [x] Login endpoint with JWT token generation
- [x] Password hashing with bcrypt
- [x] Auth middleware for protected routes
- [x] Created src/auth/login.ts and src/auth/middleware.ts

In Progress:
- [ ] Registration endpoint — validation schema defined but not wired up
- [ ] src/auth/register.ts has uncommitted changes (git diff shows ~40 lines)

Next Steps:
1. Complete registration validation in src/auth/register.ts
2. Add registration tests in src/auth/__tests__/register.test.ts
3. Update API documentation with auth endpoints
4. Run full test suite before committing

Blockers:
- None currently

Context:
- Password requirements: min 8 chars, 1 uppercase, 1 number
- JWT expires in 24h — configurable via AUTH_TOKEN_EXPIRY env var
- Auth middleware attaches user object to request context
- See src/auth/README.md for full auth architecture notes
```

## Format Rules

- Start with the current state in one sentence
- Use checkboxes for completed/in-progress items
- List next steps in strict priority order
- Flag blockers prominently (use ⚠️ or bold)
- Include exact file paths and line numbers
- Keep it scannable — use headings, lists, and whitespace
