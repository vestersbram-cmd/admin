---
name: skills-maintenance
description: Maintain cleanliness and quality in the skills folder. Use when skills need consolidation, duplicates need removal, skill value needs assessment, or the skills directory requires organization and pruning.
---
# Skills Folder Maintenance

## Overview

AI agents have a tendency to create skills for every minor learning or temporary insight. Over time, this leads to:
- **Duplicate information** across multiple skills
- **Low-value skills** that duplicate general knowledge
- **Fragmented knowledge** scattered across many files
- **Skill bloat** making it hard to find useful skills

This skill provides a structured approach to maintaining a clean, high-quality skills folder by evaluating, consolidating, and pruning skills based on their actual value.

## When to Use

Use this skill when:
- The skills folder has grown large and unwieldy
- You suspect duplicate or overlapping skills exist
- New skills are being created frequently
- Skills need periodic review and cleanup
- Consolidating knowledge from multiple similar skills
- A user explicitly asks to "clean up skills" or "organize skills"

**Trigger phrases:**
- "Clean up the skills folder"
- "Organize skills"
- "Remove duplicate skills"
- "Consolidate these skills"
- "Review skills for quality"
- "Prune low-value skills"

## Core Principles

1. **Skills are for durable knowledge** — Temporary workarounds don't belong in skills
2. **One source of truth** — Each concept should exist in exactly one skill
3. **External references over duplication** — Link to official docs instead of copying
4. **Value test** — Would a future agent find this skill more useful than a web search?
5. **Minimalism** — Smaller, focused skills beat comprehensive but bloated ones

## The Evaluation Framework

### Step 1: Inventory and Categorize

First, get a complete picture of all skills:

```python
# List all skill directories and their SKILL.md files
import os
from pathlib import Path

skills_dir = Path("skills")
skill_files = list(skills_dir.rglob("SKILL.md"))

# Extract frontmatter and basic metadata from each
for skill_file in skill_files:
    content = skill_file.read_text()
    # Parse name, description, tags from frontmatter
    # Build inventory with path, name, description, tags
```

### Step 2: Identify Problem Categories

Look for these patterns:

| Pattern | Detection | Action |
|---------|-----------|--------|
| **Exact duplicates** | Same name/description in multiple locations | Delete duplicates, keep best version |
| **Overlap skills** | Similar descriptions, related tags | Consolidate into single skill |
| **Trivial skills** | "How to use X" where X is well-documented | Delete, replace with reference link |
| **One-off workarounds** | Specific to a temporary situation | Delete or archive |
| **Fragmented knowledge** | One topic split across many skills | Merge into comprehensive skill |
| **Orphan skills** | No related files, just SKILL.md | Evaluate if standalone value exists |
| **Overly broad skills** | Tries to cover too much | Split into focused skills |

### Step 3: Value Assessment Criteria

For each skill, evaluate:

**Must have at least ONE of these:**
- **Codebase-specific knowledge** — Project conventions, internal APIs, custom patterns
- **Proprietary process** — Unique workflow developed for this project
- **Curated external knowledge** — Distilled, organized info not easily searchable
- **Tool integration** — Specific setup/config for tools used in this codebase

**Red flags (consider deletion):**
- General programming knowledge easily found online
- Basic tool usage covered by official documentation
- Single session workarounds without lasting value
- Duplicates information from other skills
- Describes a bug that's been fixed
- "Learning notes" without actionable structure

### Step 4: External Verification

For borderline skills, verify actual value:

```python
# Search web for skill topic
search_web(query="{skill topic} tutorial documentation")

# Check if this is general knowledge
# If official docs are comprehensive → skill may be redundant

# Check codebase relevance
grep_search(query="{skill topic}", search_path="{codebase}")
# If no specific codebase integration → likely low value
```

## Consolidation Workflow

### Merging Similar Skills

When two skills cover overlapping ground:

```python
# 1. Read both skills completely
read_file("skills/topic-approach-a/SKILL.md")
read_file("skills/topic-approach-b/SKILL.md")

# 2. Identify unique value in each
# 3. Create consolidated skill with best of both
# 4. Update references if needed
# 5. Delete original skills
# 6. Verify no broken references remain
```

**Consolidation principles:**
- Keep the better-structured skill as base
- Merge unique insights from the other
- Preserve all relevant tags
- Update `related_skills` in affected skills

### Deleting Low-Value Skills

Before deleting, verify:

1. **Check references:** Is this skill referenced by others?
   ```bash
   grep -r "skill-name" skills/ --include="*.md"
   ```

2. **Check usage:** Has it been recently useful?
   - Look at git history for the skill
   - Check if it's in `related_skills` of active skills

3. **Archive vs Delete:**
   - **Delete:** Clear low value, no unique content
   - **Archive:** Move to `skills/archive/` if historical value

4. **Clean up:** After deletion, remove from any `related_skills` lists

## Directory Structure Standards

Maintain consistent organization:

```
skills/
├── skill-content-format/          # Meta-skill about skill format
├── software-development/          # Domain category
│   ├── add-tool/
│   ├── subagent-driven-development/
│   └── writing-plans/
├── devops/                        # Domain category
├── database/                      # Domain category
└── archive/                       # Retired skills (if keeping)
```

**Organization rules:**
- Use kebab-case for directory names
- Group related skills under domain folders
- Keep individual skills at top level if truly cross-cutting
- Maximum depth: 2 levels (domain/skill-name)

## Quality Checklist

When reviewing a skill, verify:

- [ ] Has proper YAML frontmatter with name, description, version
- [ ] Description clearly states when to use the skill
- [ ] Content is actionable (not just informational)
- [ ] No duplication with other skills
- [ ] Code examples use project conventions
- [ ] References are up-to-date and valid
- [ ] Tags are accurate and specific
- [ ] `related_skills` lists actually related skills

## Common Cleanup Patterns

### Pattern: "Learning Journey" Skills

**Problem:** Skills created while learning a topic, full of exploration notes.

**Solution:**
1. Identify the core durable knowledge
2. Rewrite as focused skill
3. Delete exploration noise

### Pattern: "Tool Usage" Skills

**Problem:** Skills that just explain how to use a common tool.

**Solution:**
- If tool has good official docs → Delete skill, add reference link instead
- If tool needs project-specific configuration → Keep, focus on config
- If tool has custom scripts/helpers → Keep, focus on custom code

### Pattern: "Temporary Workaround" Skills

**Problem:** Skills for bugs that are now fixed or temporary situations.

**Solution:**
- Verify if issue still exists
- If fixed → Delete skill
- If workaround still needed → Keep with clear "temporary" notice

### Pattern: "Mega Skills"

**Problem:** One skill tries to cover everything about a broad topic.

**Solution:**
- Split into focused sub-skills
- Create index skill that references them
- Keep related_skills updated

## Decision Tree

```
Evaluate a skill:
│
├─ Does it contain codebase-specific knowledge?
│  └─ YES → Keep, ensure it's well-organized
│
├─ Does it duplicate general documentation?
│  └─ YES → Delete, replace with reference link
│
├─ Is it a temporary workaround?
│  ├─ Issue still exists? → Keep with temporary notice
│  └─ Issue fixed? → Delete
│
├─ Does it overlap with another skill?
│  └─ YES → Consolidate both into best version
│
├─ Is it a "learning journey" skill?
│  └─ YES → Extract value, rewrite focused, delete original
│
└─ Is it actively used (in related_skills, recent git activity)?
   ├─ YES → Keep
   └─ NO → Archive or delete
```

## Execution Workflow

### Phase 1: Discovery (Read-Only)

1. **Inventory all skills**
   - List every SKILL.md file
   - Extract metadata (name, description, tags)
   - Build skills database

2. **Identify duplicates and overlaps**
   - Group by similar names
   - Compare descriptions for similarity
   - Check for shared tags with similar topics

3. **Flag candidates for review**
   - Mark obvious duplicates
   - Identify potential consolidations
   - List skills with red flags

### Phase 2: Planning

1. **Create cleanup plan**
   ```markdown
   # Skills Cleanup Plan

   ## Deletions
   - skill-a: Reason for deletion
   - skill-b: Reason for deletion

   ## Consolidations
   - Merge skill-c into skill-d (reason)

   ## Rewrites
   - skill-e: Extract core knowledge, remove exploration
   ```

2. **Get user approval** for major changes
   - Present plan with reasoning
   - Highlight skills to be deleted
   - Wait for explicit confirmation

### Phase 3: Execution

1. **Start with deletions** (safest)
   - Remove from filesystem
   - Update any related_skills references

2. **Perform consolidations**
   - Create merged skill
   - Delete originals
   - Update references

3. **Clean up directory structure**
   - Move skills to appropriate categories
   - Ensure consistent naming

4. **Verify no broken references**
   ```bash
   # Check for dangling references
   grep -r "related_skills:" skills/ --include="*.md" | grep {deleted-skill-name}
   ```

### Phase 4: Validation

1. **Test skill discovery**
   ```python
   from skills_manager.resolver import resolve_skill
   # Verify all remaining skills resolve correctly
   ```

2. **Check frontmatter validity**
   ```python
   from tools.skill_manager_tool import _validate_frontmatter
   # Validate all SKILL.md files
   ```

3. **Final review**
   - Look at cleaned skills folder
   - Confirm structure makes sense
   - Document what was changed

## Red Flags — Never Do These

- **Don't delete without checking references** — Always verify nothing links to a skill before deleting
- **Don't consolidate without reading fully** — Partial consolidation loses valuable details
- **Don't keep skills "just in case"** — If it doesn't pass the value test now, delete it
- **Don't move skills without updating references** — Broken related_skills links are confusing
- **Don't split mega-skills without creating the index** — Users need to find the pieces
- **Don't skip the planning phase** — Bulk cleanup without a plan causes chaos

## Examples

### Example 1: Finding Duplicate Skills

```python
# Find skills with similar names/descriptions
skills = [
    {"name": "git-workflow", "desc": "Git branching and commit conventions"},
    {"name": "github-workflow", "desc": "How we use git and github"},
    {"name": "version-control", "desc": "Git workflow for the team"},
]

# These three likely overlap significantly
# Action: Consolidate into single "git-workflow" skill
```

### Example 2: Deleting a Low-Value Skill

**Skill to evaluate:** `how-to-use-pip`

**Assessment:**
- Content: Basic pip install/uninstall commands
- Value test: Would an agent find this more useful than `pip --help`?
- Result: No — pip documentation is comprehensive
- Action: Delete skill

### Example 3: Consolidating Overlapping Skills

**Skills:** `database-postgres-setup`, `postgresql-local-dev`, `postgres-docker`

**Assessment:**
- All three cover PostgreSQL setup
- Each has unique value (local install, docker, cloud)
- Action: Consolidate into `postgresql-setup` with sections for each approach

## Remember

```
Skills are for durable, codebase-specific knowledge
When in doubt, delete — web search beats outdated skill
One source of truth beats scattered fragments
Quality over quantity — 10 great skills beat 50 mediocre ones
```

**A clean skills folder is a useful skills folder.**
