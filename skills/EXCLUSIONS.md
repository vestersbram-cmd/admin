# Excluded Skills: Why Some Source Skills Were Not Included

This document explains which skills from the three source directories were **not** copied into `~/skillz/skills/`, and why.

## Sources Consulted

1. **Codebuff** — `/home/tom260513/codebuff/.agents/skills/` (10 skills copied)
2. **`~/.agents/skills`** — User's local agent skills (13 skills copied)
3. **Third-party** — `~/dev/packages/_CLAW/third-party/skills/` (4 skills copied)

---

## Excluded from Third-party Source

### `github`
**Why excluded:** This directory contained sub-skills (`clone`, `fork`, `issue`, `pr`, `repo`) rather than a single top-level `SKILL.md`. The individual sub-skills were too fragmented and overlapped with general git knowledge already present in `workflow` and `brownfield`.

### `research`
**Why excluded:** No top-level `SKILL.md` file existed in this directory. The research use case is already well-covered by `browserchat-ask` (delegate to external AI providers) and `browsernotes` (query codebase docs).

### `software-development`
**Why excluded:** No top-level `SKILL.md` file existed. This directory may have been a category folder rather than an actual skill. The software development lifecycle is comprehensively covered by `plan`, `execute`, `review`, `workflow`, `brownfield`, and `greenfield`.

### `domain`
**Why excluded:** No top-level `SKILL.md` file existed. Domain-specific knowledge is better captured per-project in a project's own documentation.

---

## Merged Instead of Duplicated

Several skills existed in multiple sources with overlapping content. Rather than include duplicates, the best version was selected and enhanced:

| Skill | Primary Source | Enhancements from Other Sources |
|-------|---------------|--------------------------------|
| `plan` | `~/.agents/skills` + `design-thinking` | Added structured markdown template, decision-complete planning heuristics, design thinking phases (territory, framing, interface, tradeoffs, critique) |
| `execute` | `~/.agents/skills` | Added evidence-based execution workflow, completion checklist |
| `cleaner` | `~/.agents/skills` | Added structured process, focus areas, output format (since consolidated into `cleanz`) |
| `review` | codebuff + `verify` | Merged with verification gate, structured output format, severity levels, guardrails, and completion protocol (absorbed `verify`) |

The alternate versions of these skills were **not** copied separately because their content was fully absorbed into the merged versions above. `cleaner` was later consolidated into `cleanz` to create a single comprehensive code quality skill.

---

## Intentionally Removed for Scope

### Platform-specific infrastructure skills
Five skills were removed because they were tightly coupled to a specific platform (Windsurf, VSAPI, Discord, custom scripts) and could not be made generically useful:

- `change-report` — Referenced platform-specific sync paths and terminology
- `forget-everything` — Destructive actions targeted at a specific agent's home directory
- `gmail-creditor` — Entirely about a package whose name and internals were platform-branded
- `orchestrate` — Managed Windsurf sessions, VSAPI, and Discord notifications
- `testing-standards` — Every command referenced platform-specific scripts (`test_planner.py`, `pytest_scaffold.py`)

---

## Summary

| Reason for Exclusion | Count | Names |
|----------------------|-------|-------|
| No `SKILL.md` file present | 3 | `research`, `software-development`, `domain` |
| Fragmented sub-skills (no single skill file) | 1 | `github` |
| Too platform-specific to generalize | 5 | `change-report`, `forget-everything`, `gmail-creditor`, `orchestrate`, `testing-standards` |
| **Total excluded** | **9** | — |

> **Note:** 4 additional skills (`plan`, `execute`, `cleaner`, `review`) existed in multiple sources and were **merged** rather than duplicated. Their content is represented in the merged versions — they are counted among the 22 skills, not excluded.

**Result:** 22 high-quality, non-overlapping, platform-agnostic skills covering the full development lifecycle.

## Post-Creation Consolidations

After the initial curation, two additional consolidations were performed:

| Skill | Action | Reason |
|-------|--------|--------|
| `cleaner` | Consolidated into `cleanz` | `cleaner` was a thin process wrapper around the `cleanz` tool — merged to create a single comprehensive skill |
| `browsernotes-codebase` | Consolidated into `browsernotes` | Both were about the same NotebookLM codebase integration (sync + query) — merged into one skill with Setup and Query sections |
| `design-thinking` | Absorbed into `plan` | Design thinking phases (territory, framing, interface, tradeoffs, critique) merged as preamble to existing plan-writing content |
| `verify` | Absorbed into `review` | Iron law, verification gate, self-verification, completion protocol, and code review protocols merged into `review` |
| `browser` | Renamed to `browser-github`; PROCEDURE.md → SKILL.md | Restructured from multi-file procedure into single SKILL.md with clearer scope (GitHub-specific) |
| `mermaid-master-architect` | Renamed to `mermaid-create` | Streamlined name; content preserved |
| `meta` | Removed | Codebuff-specific internals, not portable; conventions already covered by `workflow` and `plan` |

---

*Last updated: 2026*
