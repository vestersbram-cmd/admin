---
name: mermaid-master-architect
description: Generate, refine, and visually optimize error-free Mermaid diagrams for system architecture, process flows, data models, and state transitions using default colors only.
---
# Mermaid Master Architect

Expert at creating, refining, and optimizing error-free Mermaid diagrams. Understands that Mermaid relies on specific syntax rules and that underlying layout algorithms (Dagre/Sugiyama) require careful node declaration to prevent tangled, unreadable outputs.

## Purpose

Generate, refine, and visually optimize error-free Mermaid diagrams for system architecture, process flows, data models, and state transitions.

## When to Use

- User requests to "create a diagram", "generate mermaid", or "visualize architecture"
- Documenting system flows, APIs, or component interactions
- Drawing flowcharts, sequence diagrams, or state machines
- Mapping out code structure or data relationships
- Resolving Mermaid syntax errors or layout issues

## Intent Analysis & Routing Workflow

Analyze intent before writing code to choose the correct diagram type:

| Intent | Diagram Type |
|--------|--------------|
| Process, Logic, or Algorithms | `flowchart TD` or `flowchart LR` |
| API Flow, Interactions, or Time | `sequenceDiagram` |
| Infrastructure or Cloud Components | Architecture Flowchart with subgraphs or `C4Context` |
| Data Schemas | `erDiagram` or `classDiagram` |
| State Changes | `stateDiagram-v2` |

## Golden Rules of Syntax & Layout

To prevent syntax errors and minimize cognitive load (crossed lines):

1. **Declare First, Connect Later:** Always declare elements (nodes, participants, containers) in a block *before* drawing connections.
2. **Left-to-Right / Top-to-Bottom Declaration:** Mermaid places elements based on declaration order. Declare nodes in logical sequence to minimize crossed relationship lines.
3. **Explicit Quoting:** Always quote labels to prevent parser issues with special characters or spaces: `NodeID["Label Text"]`.
4. **Node Limit:** Keep diagrams between 5–15 nodes. For larger systems, break into multiple diagrams or use nested subgraphs.
5. **Escape Characters:** Do not use `>` or `<` inside node labels without proper escaping, as they break HTML parsing.

## Aesthetic & Styling Principles

**IMPORTANT: Always use default colors only. No custom colors or explicit styling should be applied.**

Diagrams should be production-ready and readable using Mermaid's built-in default theming.

### Color Policy

- **NEVER** use custom colors, hex codes, or explicit styling
- **ALWAYS** rely on Mermaid's default color schemes
- **NO** `classDef` styling with custom colors
- **NO** explicit fill, stroke, or color properties
- **NO** theme customization

### Subgraph Usage

Use subgraphs for organization without any styling:

```mermaid
subgraph SystemName["System Label"]
```

### Icon Restrictions

Do not use Font Awesome (`fa:fa-*`) icons unless explicitly requested. Many renderers (including GitHub) do not support them natively.

## Specific Diagram Constraints

### Flowcharts (`flowchart TD` / `LR`)

- Use rounded shapes `([Text])` for start/end states
- Use `((Text))` for circles/events
- Use `[Text]` for standard processes
- Add descriptive text to complex lines: `A -->|Validates Token| B`

### Sequence Diagrams (`sequenceDiagram`)

- Declare participants in exact left-to-right visual order
- Use `autonumber` at the top of the diagram
- Use `box` to group related participants (newer Mermaid versions)
- Use `opt`, `alt`, and `loop` blocks for conditional logic
- Keep nested blocks to maximum 2 deep

## Iterative Refinement Workflow

If MCP tools like `mermaid_preview` or `mermaid_save` are available:

1. **Draft:** Create initial raw diagram syntax
2. **Preview:** Run `mermaid_preview` to render SVG
3. **Troubleshoot:** Check for:
 - Unquoted labels containing spaces
 - Reserved characters (`{}`, `()`, `[]`) used inside node IDs
 - Missing commas or semicolons in class definitions
4. **Save:** Output final markdown block or use `mermaid_save` once error-free

## Pre-Flight Quality Checklist

Verify before delivering final Mermaid block:

- [ ] Diagram starts with correct directive (e.g., `flowchart TD`)
- [ ] All node IDs are alphanumeric with no spaces
- [ ] All visible text labels wrapped in quotes `["Like This"]`
- [ ] Elements declared in layout-order before relationships defined
- [ ] No circular logic or infinite loops (unless documenting literal loop)
- [ ] No custom colors or explicit styling applied

## Chat Output Format

If the mermaid diagram should be returned in the chat and not in a file, encapsulate it with triple back-tick + `plaintext` instead of triple back-tick + `mermaid`.

## Example Usage

```python
# Example: Creating a system architecture diagram
# 1. Analyze intent: Infrastructure visualization → flowchart with subgraphs
# 2. Use default colors only - no custom styling
# 3. Declare nodes in visual order before connecting
# 4. Run pre-flight checklist
# 5. Deliver final Mermaid block
```
