---
name: infotree
description: Interactive decision-tree system for clarifying user intent through structured multiple-choice questions
---
# Infotree Skill

## Layer 1: The Big Picture (What infotree does)

**Simple explanation:** infotree is a structured conversation system that helps understand what the user wants by asking multiple-choice questions.

**Metaphor:** "It's like a doctor's diagnosis — keep asking targeted questions until you understand the problem, then summarize your findings for confirmation."

**Key concept:** The user starts with a vague idea ("I want to refactor the auth module"), and infotree helps narrow down exactly what they mean through a series of guided questions with recommendations.

**How you access it:** The `infotree` package is pip-installed. Use the **terminal tool** to execute all CLI commands.

---

## Layer 2: When This Skill Activates

The user types `/it` or `/infotree`. There are two modes:

### Mode A: No prompt after `/it`
- **What happens:** List existing infotree sessions for the user to choose from
- **Your action:** Run `infotree list` and present results

### Mode B: User provides a prompt after `/it <prompt>`
- **What happens:** Interpret the user's intent and act accordingly
- **Your action:** Analyze what the user wants, then do one of:

**Examples of what users might say:**
- `"find the infotree about honey making"` → **Search** existing sessions by keyword: `infotree search honey`
- `"start up the last infotree we worked on"` → **Resume** most recent session (use `infotree list` to find it)
- `"what infotrees have we got for ESP32 devices?"` → **Search by topic**: `infotree topic ESP32`
- `"Show me all completed and uncompleted infotrees in a table"` → **List and format** sessions from `infotree list`
- `"I want to refactor the auth module"` → **Start new** session: `infotree start "refactor the auth module"`

**Decision rule:**
1. If the prompt mentions finding/searching/listing → Use `infotree search` or `infotree list`
2. If the prompt mentions resuming/continuing/starting up → Find session ID, then `infotree resume <id>`
3. If the prompt is a new topic/question/note → `infotree start "<prompt>"`
4. If ambiguous → Ask the user a clarifying question, preferably multiple choice.

---

## Layer 3: How infotree Works (The Flow)

### Phase 1: Session Management (Entry Point)
1. **Receive user's `/it` command** → Read what they typed after the command
2. **Decide what to do** based on the prompt (see Layer 2 examples)
3. **Execute CLI commands** to perform the action (list, search, resume, or start)

### Phase 2: Active Session (Once Inside a Session)
The infotree system manages a **decision tree** where each node is a multiple-choice question:

1. **Present multiple-choice questions** to the user
2. **User can answer with:**
   - Choice selections (e.g., "1, 3")
   - Open text (e.g., "Actually I want something else...")
   - Both at once
3. **Every question has:**
   - 2-5 specific choices
   - Exactly ONE "do more research" option
   - At least ONE recommended choice with reasoning
4. **When you understand enough:**
   - Summarize intent in 1-4 lines
   - Ask user to approve, retry, research more, or stop
5. **User can revise earlier answers:**
   - Creates a new branch (old path preserved)
   - Can navigate between branches

### Advanced Integration Patterns

**Pattern: Full Pipeline Integration**
**When to use:** Multiple related packages need to work together as a unified system

**Example - Email + Finance Automation:**
```bash
# 1. Start infotree to plan the integration
infotree start "Integrate gmailz, gmail_finance, and inbox_cleaner"

# 2. Through iterative questions, arrive at:
#    - Choice: Full pipeline (gmailz→gmail_finance→inbox_cleaner)
#    - Justification: User needs both research and automated cleanup

# 3. Implementation phases:
#    Phase 1: Foundation (gmailz setup)
#    Phase 2: Integration (gmail_finance detection)
#    Phase 3: Automation (inbox_cleaner workflow)
```

**Key decisions from this pattern:**
- Always choose the full pipeline when user needs both setup AND ongoing automation
- Research existing setup BEFORE implementation (avoids breaking changes)
- Use phased approach: foundation → integration → automation

**Pattern: Research-First Implementation**
**When to use:** Uncertain about existing configuration or user wants to understand before changing

**Approach:**
1. `infotree search <keywords>` - Find related sessions
2. `infotree list` - Review all sessions
3. `infotree start "research current setup"` - Create research session
4. Only then formulate implementation plan

---

## Layer 4: Detailed Step-by-Step Instructions

### STEP 1: Interpret the User's Intent (Entry Handler)

**If user said nothing after `/it`:**
```bash
infotree list
# Present to user: show titles, status, dates, system names
```

**If user wants to find/search:**
```bash
# Extract keywords from their prompt
infotree search <keyword>
# Or for broader topic search:
infotree topic <topic>
# Filter by keywords in: title, tags, original_prompt, intent_summary
# Present matching sessions with metadata
```

**If user wants to resume:**
```bash
# First identify which session from their description
infotree list
# Or search: infotree search <keyword>
# Then resume:
infotree resume <session_id>
# Or resume at specific node:
infotree resume <session_id> --node <node_id>
```

**If user wants to start new:**
```bash
infotree start "<prompt>" [--type <prompt_type>]
# Then immediately begin Phase 2 (formulate first question)
```

### STEP 2: Active Session Management (Once in a Session)

**For each node in the tree, follow this exact workflow:**

#### 2.1 Check Current Node
```bash
infotree get <session_id>
# Parse the JSON to find: current_node_id, node status, tree structure
```

#### 2.2 If Node Status is PENDING → You MUST Formulate Choices

**First, do research (see Layer 5 for detailed methodology):**
```bash
# Use Layer 5 Research Methodology:
# 1. Search llm-wiki for prior knowledge
# 2. Search ~/dev/packages for implementation
# 3. Synthesize findings using semantic groups

# Then record your research:
infotree research <session_id> <node_id> <source> "<query>" "<findings>"
# source: codebase | llm_wiki | online | context
```

**Then, create choices (JSON format):**
```bash
infotree formulate <session_id> <node_id> '[
  {
    "id": "c1",
    "label": "JWT Token Auth",
    "description": "Standard JWT implementation with refresh tokens",
    "is_recommended": true,
    "recommendation_reason": "Best for API backends, industry standard"
  },
  {
    "id": "c2",
    "label": "OAuth2 with External Provider",
    "description": "Use Google/GitHub as identity provider"
  },
  {
    "id": "c3",
    "label": "Session-based Auth",
    "description": "Traditional server-side sessions"
  },
  {
    "id": "c4",
    "label": "Research more options",
    "description": "Do more research before deciding",
    "is_research_option": true
  }
]'
```

#### 2.3 Present Choices to User

Format clearly:
```
**Question:** [The question text]

**Options:**
**1)** [label] (recommended): [description]
   → Why: [recommendation_reason]

**2)** [label]: [description]

**3)** [label]: [description]

**4)** [label] (research option): [description]

You can also type any open response.
```

**IMPORTANT:** Multiple choice options must be numbered (1, 2, 3, etc.), not lettered (A, B, C, etc.).

#### 2.4 Receive User's Answer

User responds with:
- Just choice IDs: "1" or "1, 2"
- Just open text: "I actually need something else..."
- Both: "1, but also I want to mention..."

#### 2.5 Record the Answer
```bash
infotree answer <session_id> <node_id> "<choice_ids>" [--open "<open_answer>"]
# Example:
infotree answer it_abc123 n1 "c1" --open "I also need it to work with legacy systems"
# This creates a new child node automatically
```

**NOTE:** The CLI command uses internal choice IDs (c1, c2, c3, etc.) which correspond to the user-facing numbers (1, 2, 3, etc.). When the user selects "1", you record it as "c1"; when they select "2", you record it as "c2", and so on.

#### 2.5.5 Set the Next Question (CRITICAL!)

**After recording an answer, the new child node has placeholder text ("Awaiting formulation..."). You MUST set the real question before formulating choices:**

```bash
infotree context <session_id> <new_node_id> "<question>" "<context_summary>"
# Example:
infotree context it_abc123 n2 "What specific auth mechanism do you prefer?" "User wants auth refactor, need to know: JWT, OAuth, or session-based?"
```

**This step is REQUIRED because:**
- `infotree answer` creates the child node with placeholder text
- The question must be set before you can formulate meaningful choices
- Without this step, the tree will show "Awaiting formulation..." instead of your actual question

#### 2.6 Contemplate: Do You Have Enough Information?

**Check if ready to summarize:**
- Do you know exactly what the user wants?
- Can you state it clearly in 1-4 lines?
- Is the scope clear and actionable?

**If YES → Summarize:**
```bash
infotree summarize <session_id> "<summary>" <confidence>
# Example:
infotree summarize it_abc123 "Add JWT token authentication with refresh tokens, integrated with the existing PostgreSQL users table. Support for both new and legacy API endpoints." 0.85
# Then present to user and ask for approval
```

**If NO → Formulate Next Question:**
- Based on user's answer, what don't you know yet?
- What would clarify the remaining ambiguity?
- **Set the question via step 2.5.5** (if not already done)
- Research if needed, then formulate new choices
- Go back to step 2.2

#### 2.7 If Summarizing → Present Summary and Ask

Format:
```
**Intent Summary** (confidence: 85%)

"Add JWT token authentication with refresh tokens, integrated with the existing PostgreSQL users table. Support for both new and legacy API endpoints."

**Please choose:**
1. **Approve** - Proceed with this understanding
2. **Retry** - Let's go back and ask different questions
3. **Research** - Do more research before finalizing
4. **Stop** - End this infotree session
```

#### 2.8 Handle Approval/Denial

**If approve:**
```bash
infotree approve <session_id>
# Session status becomes APPROVED
```

**If retry:**
```bash
infotree deny <session_id> "<reason>" retry
# Ask user which node to revisit, then:
infotree revise <session_id> <node_id>
```

**If research:**
```bash
infotree deny <session_id> "<reason>" research
# Continue with step 2.2 (do more research before next question)
```

### STEP 3: Revisions and Branching

When user wants to change an earlier answer:
```bash
infotree revise <session_id> <node_id>
# This:
# 1. Marks old descendants as SUPERSEDED
# 2. Marks node as REVISING
# 3. Creates new child node for revised path
# 4. Sets current_node_id to new child
```

The old branch remains in history — user can view both paths.

---

## Layer 5: Research Methodology (How to Search Effectively)

Research is the foundation of good infotree questions. This layer teaches you HOW to search efficiently across all knowledge sources.

### 5.1 System Knowledge Architecture

Know where information lives:

| Source | Location | What You'll Find |
|--------|----------|------------------|
| **LLM-Wiki** | `~/.llm-wiki/` | Prior research, exported infotrees, knowledge bases |
| **Codebases** | `~/dev/packages/` | All source code, recursively searchable |
| **Package Registry** | `~/.llm-wiki/packages/` | Tracked/known packages |
| **System** | `~/` | Everything else (configs, notes, etc.) |

### 5.2 Search Strategy: Semantic Groups

Don't search for individual keywords. Search for **concepts** using regex alternation:

```bash
# BAD: Multiple searches for related terms
search_files(pattern="auth", ...)
search_files(pattern="oauth", ...)
search_files(pattern="jwt", ...)

# GOOD: Single search with semantic group
search_files(pattern="auth|oauth|jwt|token|login|credential", ...)
```

**Standard Semantic Groups:**

| Topic | Pattern | Use When |
|-------|---------|----------|
| Authentication | `auth|oauth|jwt|token|login|credential|session` | User mentions auth, login, security |
| Database | `database|db|sql|postgres|sqlite|orm|model` | Data storage questions |
| API | `api|endpoint|route|rest|graphql|http` | API design or integration |
| Frontend | `frontend|ui|react|vue|html|css|component` | UI/UX questions |
| Async | `async|await|concurrent|thread|parallel|celery` | Concurrency topics |
| Testing | `test|pytest|unittest|mock|fixture|coverage` | Testing strategy |

### 5.3 Multi-Source Research Workflow

**Step 1: Search Prior Knowledge (fast, informative)**
```bash
# Check llm-wiki first — it contains distilled knowledge
# (Use your file search capability)
```

**Step 2: Search Codebase (authoritative)**
```bash
# Search implementation in all packages
# (Use your file search capability with patterns like)
# search_files(pattern="auth|oauth|jwt", path="~/dev/packages", file_glob="*.py")
```

**Step 3: Synthesize Findings**
Analyze what you found, then record:
```bash
infotree research <session_id> <node_id> codebase "auth patterns in codebase" "Found 3 JWT files, 2 OAuth implementations. JWT appears to be already partially implemented in auth/jwt_handler.py"
```

### 5.4 Search Optimization Guidelines

| Approach | When to Use | Speed | Detail |
|----------|-------------|-------|--------|
| `output_mode="files_only"` | Quick reconnaissance | Fastest | Just file paths |
| `output_mode="content"` | Understanding implementations | Fast | Lines with context |
| `file_glob="*.py"` | Language-specific search | Fast | Filters noise |
| `limit=10` | Initial exploration | Fast | Top matches only |
| `limit=50` | Deep research | Slower | Comprehensive |

### 5.5 Common Research Patterns

**Pattern: Package Reference by Name**
```bash
# User says "the infotree package"
# → Search in ~/dev/packages/infotree
```

**Pattern: Find Configuration**
```bash
# Search for config files across all packages
```

**Pattern: Cross-Reference**
```bash
# Search both wiki and codebase, compare findings
# If wiki has "async guide" and code has no async/await → gap identified
```

---

## Layer 6: CLI Command Reference

### Session Management
| Command | Purpose | When to Use |
|---------|---------|-------------|
| `infotree start "<prompt>" [--type <type>]` | Create new session | Starting a new infotree |
| `infotree list` | List all sessions | `/it` with no args, or listing |
| `infotree search <keyword>` | Search by keyword | Finding specific sessions |
| `infotree topic <topic>` | Search by topic | Flexible topic matching |
| `infotree get <session_id>` | Load session data | Checking current state |
| `infotree resume <session_id> [--node <node_id>]` | Resume session | Continuing existing session |
| `infotree title <session_id> "<title>"` | Update session title | Better organization |
| `infotree tags <session_id> <tag1> [tag2...]` | Add tags | Categorizing sessions |
| `infotree delete <session_id>` | Delete session | User wants to remove a session |

### Tree Operations
| Command | Purpose | When to Use |
|---------|---------|-------------|
| `infotree formulate <session_id> <node_id> '<choices_json>'` | Add choices to node | Ready to ask user a question |
| `infotree answer <session_id> <node_id> "<choice_ids>" [--open "<text>"]` | Record user answer | User has responded |
| `infotree research <session_id> <node_id> <source> "<query>" "<findings>"` | Log research | After searching codebase/wiki/web |
| `infotree revise <session_id> <node_id>` | Return to earlier node | User wants to change an answer |
| `infotree context <session_id> <node_id> "<question>" "<summary>"` | Update node context | Setting question context |

### Lifecycle Commands
| Command | Purpose | When to Use |
|---------|---------|-------------|
| `infotree summarize <session_id> "<summary>" <confidence>` | Set intent summary | Ready to present final understanding |
| `infotree approve <session_id>` | Mark approved | User confirms summary |
| `infotree deny <session_id> "<reason>" <retry\|research>` | Mark denied | User rejects (mode: retry or research) |

### Documentation Commands
| Command | Purpose |
|---------|---------|
| `infotree readme` | Show README.md content |
| `infotree mermaid` | Show MERMAID.md content |

---

## Layer 7: Cross-Infotree Knowledge Building (IMPORTANT!)

Infotrees are NOT isolated islands. They form a **knowledge network**.

### Always Consider Other Infotrees

**Before starting a new infotree:**
```bash
infotree list
# Or: infotree search <keyword>
# Search for related topics in: title, original_prompt, tags, intent_summary
```

**When formulating choices:**
```bash
# Look at similar past infotrees for context and patterns
infotree search <keyword>
# Mention relevant findings: "Building on what we learned in [previous session]..."
```

**Build explicit connections:**
```bash
infotree research <session_id> <node_id> context "Related infotrees on this topic" "Related to 'ESP32-sensor-setup' infotree (it_abc123). Key insight: [summary]"
```

### Knowledge Synthesis Examples

**When you see similar topics:**
> "This seems related to the 'ESP32 sensor setup' infotree from last week. Key insight from that session: [copy relevant summary]"

**When you see contradictions:**
> "Note: This differs from what we decided in [other infotree] — there we chose [X] because [reason], but here the context suggests [Y]"

**When you see evolution:**
> "Building on the OAuth2 research from [previous infotree], we know that [insight], which suggests [current recommendation]"

### How to Search Across Infotrees

```bash
# Get all sessions
infotree list
# Parse and filter by keyword in metadata
```

**Remember:** The value is not in individual infotrees, but in the **connected knowledge graph** they form.

---

## Layer 8: Meta-Cognitive Reflection & Knowledge Maintenance

This layer teaches you to learn from your infotree interactions and accumulate meta-knowledge in `~/.llm-wiki/infotree-meta/`.

### 8.1 Automatic Basic Capture

On every session approval, denial, or revision, basic metrics are automatically logged:
- Session duration, depth reached, revision count
- Final status and confidence
- Research activity summary

You don't need to do anything for this — it happens automatically.

### 8.2 When to Manually Reflect

Ask yourself after significant interactions: **"Is there a pattern here worth remembering?"**

**Trigger moments for meta-reflection:**
- User redirects you away from a line of questioning
- Same question type caused revision in previous sessions
- User expresses frustration or enthusiasm about approach
- Divergence between your recommended choice and user's selection
- Topic-specific insight emerges ("when discussing API design, first clarify scope")
- You needed to revise because you asked the "wrong" question

### 8.3 CRUD Guidance for Meta-Knowledge

**CREATE** when:
- New pattern detected ("User often redirects from implementation details")
- Topic insight emerges ("For 'refactor' prompts, start with motivation not code")
- User preference noted ("Prefers concrete examples over abstract questions")

**UPDATE** when:
- Pattern strengthens or weakens with new evidence
- Refinement needed ("actually, this applies more to DB topics than all refactoring")
- Confidence changes based on accumulated data

**DELETE** when:
- Pattern proven wrong or irrelevant
- Superseded by better understanding
- Initially thought meaningful but wasn't

> **Important:** Deleting outdated insights is POSITIVE. It shows learning and correction.

**LOG** when:
- Something observed but not yet pattern-worthy
- You want a chronological trace for later synthesis
- Quick note without full insight structure

### When to Evolve Skills (Beyond Infotree)

**Create NEW skills when:**
- New integration pattern discovered (like full pipeline pattern)
- Repeated user requests for similar complex setup
- Workflow needs documentation beyond what infotree sessions capture

**UPDATE existing skills when:**
- New patterns emerge from conversations (add to relevant layers)
- User feedback improves the approach
- Anti-patterns are identified and corrected (add to Layer 9 Anti-Patterns)
- Research methodology needs new semantic groups (add to Layer 5)

### 8.4 Recording Meta-Knowledge

Meta-knowledge is stored as markdown files in `~/.llm-wiki/infotree-meta/`:
- `patterns/` - Questioning patterns you observe
- `topics/` - Domain-specific strategies
- `user/` - User preferences

Create structured markdown with this format:
```markdown
## Pattern: [Name]

**Observation:** [What you noticed]

**Trigger:** [When does this apply]

**Evidence:**
- Session [id]: [what happened]

**Recommended Action:** [What to do differently]

**Applies to:** [topic categories]
**Confidence:** LOW|MEDIUM|HIGH
```

### 8.5 Pre-Session Briefing

**BEFORE starting any new infotree:**

1. Check `~/.llm-wiki/infotree-meta/patterns/` for relevant questioning patterns
2. Check `~/.llm-wiki/infotree-meta/topics/` for domain-specific strategies
3. Check `~/.llm-wiki/infotree-meta/user/` for user preferences

Apply what you learn to formulate better opening questions.

### 8.6 Meta-Knowledge Maintenance Philosophy

- **Quality over quantity** — Fewer, high-value insights beat many weak ones
- **Delete freely** — Outdated insights should be removed, not deprecated
- **Update actively** — Patterns evolve; keep insights current
- **Log everything** — Use the log for chronological traces
- **Apply learnings** — Actually use meta-knowledge to improve questions

---

## Layer 9: Rules & Constraints (Don't Forget These!)

| Rule | Why It Matters |
|------|----------------|
| **ALWAYS include exactly one research option** in every choice set | User must always be able to say "tell me more" |
| **ALWAYS mark at least one choice as recommended** | User wants guidance on the best path |
| **ALWAYS put the recommended choice FIRST** in the list (position 1) | User sees the best option immediately |
| **ALWAYS explain the recommendation reason** | User needs to understand WHY you recommend it |
| **ALWAYS allow open text answers** | User may have nuances not captured by choices |
| **NEVER delete nodes** — revisions create new branches | History is preserved for reference |
| **Summaries must be 1-4 lines maximum** | Brevity forces clarity |
| **Confidence is 0.0-1.0** | Be honest about your certainty |
| **ALWAYS consider other infotrees** when searching/formulating | Build knowledge connections |
| **Record ALL research** via `infotree research` command | Transparency and future reference |
| **Update `updated_at` on every change** | Accurate session tracking (automatic) |

### Anti-Patterns (What NOT to Do)

| ❌ Don't | ✅ Do Instead |
|----------|---------------|
| Skip the planning phase and dive straight into code | Use multiple-choice questions to narrow ambiguity |
| Assume user wants the same approach as last time | Search for related sessions first, check context |
| Forget to set context before formulating choices | Always use `infotree context` after `infotree answer` |
| Don't record research findings in the session | Log all research via `infotree research` command |

---

## Layer 10: Common Patterns

### Pattern: Iterative Clarification
```
Initial vague request → Multiple-choice questions → Deeper questioning → Clear understanding
     "Fix my emails"          → "Which approach?"         → "Full pipeline"      → Ready to implement
```

### Pattern: Phased Rollout
```
Foundation (week 1) → Integration (week 2) → Automation (week 3+)
     ↓                   ↓                      ↓
  gmailz setup     gmail_finance rules    inbox_cleaner workflow
```

### Pattern: Safety-First
- Always research before implementing
- Use two-stage cleanup (preview → execute)
- Keep thread protection mechanisms
- Rate limit API calls

### Pattern: Quick Start
```bash
# User: /it I want to add auth to my API
infotree start "I want to add auth to my API"
# Returns session data with session_id and current_node_id
# Immediately formulate first question
```

### Pattern: Resume and Continue
```bash
# User: /it resume the auth session
infotree search auth
# Identify the right one from output, then:
infotree resume <session_id>
# Continue where it left off
```

### Pattern: Branch and Compare
```bash
# User: Actually, let me reconsider question 3
infotree revise <session_id> n2
# Creates new branch from n2
# Now you can formulate different choices
# User can later compare both branches
```

### Pattern: Search and Connect
```bash
# User: /it what did we decide about OAuth2?
infotree search OAuth2
# If found: present summary and ask if they want to resume
# If not found: offer to start new infotree on OAuth2
```

---

## Example Complete Session

```bash
# === PHASE 1: Start ===
infotree start "I want to refactor the auth module"
# Response: {"session_id":"it_abc123","current_node_id":"n1",...}

# === PHASE 2: First Question ===
# Do research first...
infotree research it_abc123 n1 codebase "auth patterns" "Found JWT in auth.py, OAuth handler in oauth.py"

# Formulate choices
infotree formulate it_abc123 n1 '[
  {"id":"c1","label":"Security focus","description":"Fix vulnerabilities","is_recommended":true,"recommendation_reason":"Security is priority for auth"},
  {"id":"c2","label":"Performance focus","description":"Optimize token validation speed"},
  {"id":"c3","label":"Code clarity","description":"Improve readability and structure"},
  {"id":"cR","label":"Research more","description":"Study current implementation first","is_research_option":true}
]'

# User responds: "1"
infotree answer it_abc123 n1 "c1"

# Set the question for the new child node (n2) BEFORE formulating choices
infotree context it_abc123 n2 "What specific auth mechanism do you prefer?" "User wants security focus, need to choose: JWT, OAuth2, or session-based?"

# Now formulate choices for n2
infotree formulate it_abc123 n2 '[
  {"id":"c1","label":"JWT tokens","description":"Stateless token-based auth","is_recommended":true,"recommendation_reason":"Already partially implemented"},
  {"id":"c2","label":"OAuth2","description":"Integration with external providers"},
  {"id":"c3","label":"Session-based","description":"Traditional server-side sessions"},
  {"id":"cR","label":"Research more","description":"Study current auth implementation","is_research_option":true}
]'

# === Continue with more questions until ready to summarize ===

# === PHASE 3: Summarize ===
infotree summarize it_abc123 "Refactor auth module to fix JWT token validation vulnerability and improve error handling" 0.9

# Present summary to user, they approve:
infotree approve it_abc123
```
