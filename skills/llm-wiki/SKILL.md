---
name: llm-wiki
description: Minimal LLM Wiki - lightweight knowledge base for markdown documentation
---
# llm-wiki - Knowledge Base Infrastructure

## Overview
The llm-wiki is a lightweight knowledge base infrastructure for markdown documentation, organized as subdirectories under `~/.llm-wiki/` with each wiki as a named subdirectory.

## Core Structure
```
~/.llm-wiki/
├── packages/           # Primary package knowledge base
│   ├── pages/          # Individual package documentation pages
│   ├── sources/        # Source tracking for documentation
│   └── AGENT.md        # Wiki agent configuration
├── skills/             # Reusable procedural skills
├── {custom}/           # Custom topic documentation
└── {wiki-name}/        # Named wiki instances
```

## Multi-Wiki Support
Each wiki is a subdirectory in `~/.llm-wiki/{wiki-name}/`. Use `--path` flag to specify which wiki to operate on.

## Package Documentation Pattern
Package pages follow a standard structure:
- YAML frontmatter with confidence, tags, type, timestamps
- Overview section with purpose and architecture
- Key features with examples
- Integration points with other packages
- Configuration and best practices
- Testing commands
- Success criteria and metrics

## Key Packages Documented

### gmailz - Core Foundation
- Multi-account OAuth2 Gmail API client
- PostgreSQL persistence via SQLAdmin
- Rate limiting (50ms interval)
- Browser-based credential automation

### gmail_finance - Detection Layer
- Keyword-based scoring (high/medium/low confidence)
- Sender domain and body pattern matching
- Rule-based filtering (whitelist/blacklist/ignore)
- Integrates with gmailz for API access

### orchestration layer - Orchestration Layer
- Deterministic two-stage pipeline
- Exclusion → Inclusion filter flow
- Batch processing with configurable depth
- Thread protection for conversation integrity

### inbox_cleaner - Cleanup Layer
- LLM-powered importance analysis
- Two-stage deletion (preview→approve→execute)
- Keyword-based importance detection
- Cross-account query verification

## CLI Commands
```bash
# List pages across all wikis
llm_wiki list

# Search all wiki pages
llm_wiki search "gmailz"

# Read a specific page
llm_wiki read packages/gmailz.md

# Create new wiki
llm_wiki init "Package Knowledge Base" --path ~/.llm-wiki/custom
```

## Update Workflow
1. Commit local changes before sync_upstream
2. Update package documentation pages in packages/pages/
3. Update skills with new patterns and fixes
4. Maintain CHANGELOG in relevant packages
5. Cross-reference related packages and patterns

## Best Practices
- Maintain backward compatibility when updating
- Document all integration points clearly
- Track configuration in JSON/YAML, not code
- Use deterministic patterns for testing
- Preserve custom patches across syncs