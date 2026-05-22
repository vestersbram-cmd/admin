---
name: present-options
description: Compact visual multiple-choice presenter for at-a-glance decision making
---
# Present Options Skill

## What It Does

Present-options renders structured multiple-choice questions in a compact, at-a-glance format. Unlike plain text questions, it formats options with icons, clear visual hierarchy, and structured layouts so users can see all choices instantly without scrolling.

**Key constraint:** All options must fit on one screen (max 4 options, ~20 lines total output).

---

## When to Use

**Use present-options when:**
- Presenting 2-4 distinct alternatives
- Visual comparison helps the decision
- Icons or structure improve clarity
- You want a recommended option highlighted

**Use clarify_tool instead when:**
- Simple yes/no or open-ended questions
- Quick confirmation only
- No visual hierarchy needed

---

## How It Works

### 1. Build Question Definition (JSON)

```json
{
  "version": "1.0",
  "title": "Deployment Strategy?",
  "context": "Release the update",
  "layout": "horizontal",
  "options": [
    {"id": "blue-green", "label": "Blue-Green", "short_desc": "Zero downtime",
     "icon": "🔵", "is_recommended": true},
    {"id": "canary", "label": "Canary", "short_desc": "Gradual rollout",
     "icon": "🐤"},
    {"id": "rolling", "label": "Rolling", "short_desc": "Sequential update",
     "icon": "🔄"},
    {"id": "research", "label": "Research", "short_desc": "Need more info",
     "icon": "🔍", "is_research_option": true}
  ]
}
```

### 2. Call the Tool

```python
result = present_options_tool(question_def_json=json.dumps(question))
# Returns: markdown, plaintext, and metadata
```

### 3. Display and Collect Input

Display the returned markdown to the user, then use `clarify_tool` or terminal input to collect their choice.

---

## Layouts

| Layout | Best For | Visual Style |
|--------|----------|--------------|
| `horizontal` | 2-4 options, side-by-side | Columns with headers |
| `table` | 3-4 options with attributes | Markdown table |
| `cards` | 3 options, icon-focused | Single row of cards |
| `vertical` | 4+ options (fallback) | Stacked list |

**Default:** `horizontal` — most compact, best for at-a-glance

---

## Field Constraints (Strict)

| Field | Max | Notes |
|-------|-----|-------|
| `title` | 40 chars | Ends with `?` |
| `context` | 60 chars | Optional situational info |
| `options` | 4 items | 3 choices + 1 research option |
| `label` | 20 chars | Per option |
| `short_desc` | 30 chars | ~5 words |
| `icon` | 2 chars | Single emoji or symbol |

---

## Option Fields

```json
{
  "id": "unique-id",
  "label": "Display Name",
  "short_desc": "Brief description",
  "icon": "🔧",
  "is_recommended": true,
  "is_research_option": false
}
```

- **id:** Unique identifier (returned in response)
- **label:** Visible text (max 20 chars)
- **short_desc:** Quick summary (max 30 chars)
- **icon:** Visual cue (emoji or 2-char symbol)
- **is_recommended:** Mark the suggested choice
- **is_research_option:** Escape hatch for "need more info"

---

## Example: Complete Flow

```python
import json

# 1. Define question
question = {
    "version": "1.0",
    "title": "Auth Method?",
    "context": "Secure the API",
    "layout": "horizontal",
    "options": [
        {"id": "jwt", "label": "JWT", "short_desc": "Stateless tokens",
         "icon": "🔐", "is_recommended": True},
        {"id": "oauth", "label": "OAuth2", "short_desc": "External auth",
         "icon": "🔑"},
        {"id": "session", "label": "Sessions", "short_desc": "Server-side",
         "icon": "🍪"},
        {"id": "research", "label": "Research", "short_desc": "Need more info",
         "icon": "🔍", "is_research_option": True}
    ]
}

# 2. Render
result_json = present_options_tool(question_def_json=json.dumps(question))
result = json.loads(result_json)

# 3. Display markdown
print(result["markdown"])

# 4. Collect response (using clarify_tool or terminal)
response = clarify_tool(
    question="Select option (1-4):",
    choices=["1", "2", "3", "4"]
)
```

---

## CLI Usage

Preview question definitions:

```bash
# Validate JSON
present-options validate question.json

# Render to see output
present-options render question.json

# Try different layout
present-options render question.json --layout table

# See all layouts in action
present-options demo
```

---

## Knowledge Base

- `kb/templates.json` — Pre-built question templates (auth, deployment, database, etc.)
- `kb/best-practices.md` — Guidelines for effective questions
- `kb/examples.md` — Real-world examples with outputs
- `schema/question-def.json` — JSON Schema for validation

---

## Comparison: clarify vs present-options

| Aspect | clarify_tool | present_options |
|--------|--------------|-----------------|
| **Best for** | Simple questions | Structured decisions |
| **Option count** | Up to 4 | 2-4 (compact) |
| **Icons** | ❌ No | ✅ Yes |
| **Layouts** | List only | horizontal/table/cards |
| **Recommended** | ❌ No | ✅ Yes |
| **Visual** | Plain | Structured markdown |
| **Output size** | Variable | ≤ 20 lines |

---

## Quick Reference

```json
{
  "version": "1.0",
  "title": "Question?",
  "context": "Situation",
  "layout": "horizontal",
  "options": [
    {"id": "a", "label": "Option A", "short_desc": "Description",
     "icon": "🔧", "is_recommended": true},
    {"id": "b", "label": "Option B", "short_desc": "Description",
     "icon": "⚙️"},
    {"id": "research", "label": "Research", "short_desc": "Need info",
     "icon": "🔍", "is_research_option": true}
  ]
}
```
