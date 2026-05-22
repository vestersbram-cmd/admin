---
name: skill-content-format
description: Guidelines for creating skills with proper YAML frontmatter and markdown content structure.
---

# Skill Content Format

How to write a skill file that an AI agent can load and follow.

## File Structure

Each skill lives in its own subdirectory under `skills/`:

```
skills/
  my-skill/
    SKILL.md
```

The directory name becomes the skill identifier. Use kebab-case.

## YAML Frontmatter (Required)

Every `SKILL.md` must start with:

```yaml
---
name: my-skill
description: What this skill does and when to use it
---
```

**Requirements:**
- Must start with `---` on its own line
- Must end with `---` on its own line
- Only two fields: `name` (matches directory) and `description`
- Keep the description under 200 characters
- Quote the description if it contains colons or special characters

## Content Body

After the frontmatter, write markdown content:

```markdown
# Skill Title

## When to Use
- Scenario 1
- Scenario 2

## Process
1. Step one
2. Step two

## Output Format
### Summary
What to include in the summary.

### Details
- Item one
- Item two

## Common Pitfalls
- Don't do X
- Avoid Y
```

## Recommended Sections

Include these sections when they add value:

- **When to Use** — clear trigger conditions
- **Process / Workflow** — numbered steps the agent should follow
- **Output Format** — structured reporting template
- **Guidelines** — rules and constraints
- **Common Pitfalls** — mistakes to avoid
- **Completion Criteria** — how to know when done

## What Not to Include

- Tool-specific API calls or code (skills are procedural guides, not scripts)
- References to external frameworks or platforms that aren't universal
- Frontmatter fields beyond `name` and `description`
- Auto-generated metadata blocks

## Example

```markdown
---
name: review
description: Review code changes for correctness, quality, and pattern alignment.
---

# Review

## When to Use
- After implementation, before merge
- When asked to check or validate code

## Process
1. Run git diff to see changes
2. Read each modified file
3. Check for correctness, simplicity, and pattern consistency
4. Report findings with severity

## Output Format
### Summary
Overall assessment.

### Issues
- [SEVERITY] File:line — description

## Common Pitfalls
- Don't suggest rewrites when a small fix would do
- Don't ignore existing patterns in the codebase
```

## Naming Conventions

- Directory: kebab-case (`my-skill`)
- Frontmatter `name`: same as directory (`my-skill`)
- Title: Title Case (`# My Skill`)
- Description: sentence case, no period at the end
