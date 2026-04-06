---
name: long-context
description: "Solve LLM context window limitations and prevent memory loss during compression. Use this skill whenever the user mentions: context window, context limit, running out of context, context overflow, memory loss, compression losing information, conversation history too long, tokens limit, max context, context management, or when the conversation becomes long and important details seem to get lost. This skill helps maintain continuity and preserve critical information across context boundaries."
---

# Long Context Memory Skill

This skill provides strategies, techniques, and tools to overcome LLM context window limitations without losing important information. It addresses the core problem: when context is compressed or truncated, valuable information disappears.

## The Core Problem

LLMs have fixed context windows (e.g., 128K, 200K tokens). When conversations exceed this limit, two things typically happen:
1. **Truncation** - Old messages are simply deleted
2. **Compression** - Messages are summarized, losing nuance and detail

Both approaches cause memory loss. This skill provides a systematic solution.

## Solution Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Your Long Conversation                    │
├─────────────────────────────────────────────────────────────┤
│  Strategy 1: Proactive Summarization (within context)       │
│  Strategy 2: External Memory (persists across sessions)     │
│  Strategy 3: Hierarchical Context (importance filtering)    │
│  Strategy 4: Structured Extraction (key facts, decisions)   │
└─────────────────────────────────────────────────────────────┘
```

## Strategy 1: Proactive Summarization

### When to Summarize

```
Summarize when:
- Conversation reaches ~60% of context limit
- Multiple subtopics have been resolved
- Important decisions were made
- Technical details were clarified
```

### Summary Template

```
## Conversation Summary: [Topic]

### Key Decisions
- [Decision 1]: [Outcome]
- [Decision 2]: [Outcome]

### Current State
- Where we are: [Location/Stage]
- What we're working on: [Task]
- Next step: [Action]

### Important Facts
- [Fact 1]
- [Fact 2]

### Resolved Questions
- [Q]: [A]
- [Q]: [A]

### Open Questions
- [Q]
```

### Proactive Reframing

Instead of waiting for context to fill, periodically say:

```
"Let's save a summary here before we continue..."

"Summarizing our progress so far..."

"At this point, here's what we established:"
```

## Strategy 2: External Memory Systems

### Project Memory File

Create a `.context/` directory in your project:

```
.project/
├── MEMORY.md          # Persistent project facts
├── DECISIONS.md       # Architectural decisions
├── CURRENT_TASK.md    # What we're doing now
└── ARCHITECTURE.md    # System design
```

### Memory Update Protocol

```
When important information is established:

1. Immediately save to external file:
   - Decisions → DECISIONS.md
   - Facts → MEMORY.md
   - Current state → CURRENT_TASK.md

2. Reference the file in conversation:
   "As we documented in DECISIONS.md, we're using..."

3. On context refresh, read the files:
   "Let me check our project memory..."
```

### Example MEMORY.md

```markdown
# Project Memory

## Last Updated
2024-01-15

## Project Overview
- Project: E-commerce API refactor
- Tech stack: Node.js, PostgreSQL, Redis
- Team size: 3 developers

## Key Decisions
1. **Database**: PostgreSQL over MongoDB (2024-01-10)
   - Reason: Better for relational data, ACID compliance
2. **Cache**: Redis for session management (2024-01-12)
3. **API Style**: REST over GraphQL (2024-01-14)
   - Reason: Team familiarity

## Technical Details
- Auth: JWT with 15-min expiry, refresh tokens
- DB Schema: users, orders, products, order_items
- API Base: /api/v1

## Current Blocker
- Payment integration awaiting Stripe API keys
- Expected: Tomorrow

## Pending Tasks
- [ ] Implement payment webhooks
- [ ] Add unit tests for order service
- [ ] Write API documentation
```

## Strategy 3: Hierarchical Context

### Importance Classification

| Tier | Content | Retention Policy |
|------|---------|-----------------|
| **Critical** | Decisions, architecture, credentials | Always externalize |
| **High** | Current task details, code patterns | Summarize in memory |
| **Medium** | Discussion context, explanations | Keep in recent context |
| **Low** | Casual comments, greetings | Can be dropped |

### Tier Classification Prompt

When information is revealed, classify it:

```
Tier 1 (Critical - EXTERNALIZE):
- Decisions and their rationale
- Architecture choices
- Credentials, API keys
- User preferences that persist
- File locations and paths

Tier 2 (High - SUMMARIZE):
- Current implementation details
- Function/class purposes
- Bug root causes
- API contract details

Tier 3 (Medium - REFERENCE):
- Discussion flow
- Exploration details
- Alternative approaches considered

Tier 4 (Low - DISCARD):
- Casual conversation
- Repetitive confirmations
- Test outputs (if passed)
```

### Reframing Technique

When you need to preserve Tier 1-2 information but context is tight:

```
CRITICAL: Before moving on, please save these to MEMORY.md:
- The decision to use PostgreSQL
- The JWT auth approach
- The current API base path

Then summarize: "We decided on X for Y reason, and we're currently working on Z"
```

## Strategy 4: Structured Extraction

### Decision Record Format

```markdown
## Decision Record: [Number]

**Date**: YYYY-MM-DD
**Context**: [Situation that required decision]

**Decision**: [What was decided]
**Alternatives Considered**:
- [Alternative 1] - rejected because...
- [Alternative 2] - rejected because...

**Consequences**:
- Positive: [Benefit]
- Negative: [Drawback]

**Status**: [Accepted|Superseded by DR-XXX]
```

### Key Fact Extraction

When you learn something important:

```markdown
## Key Fact: [Brief Title]

**Source**: [File:Line or Conversation timestamp]
**Learned**: [YYYY-MM-DD]
**Content**:
```
[Exact information or quote]
```
**Relevance**: [Why this matters for the project]
**Verified**: [Yes/No/Needs verification]
```

### Code Pattern Documentation

When establishing a pattern:

```markdown
## Code Pattern: [Name]

**Pattern**: [Brief description]
**Use When**: [Situations to apply]
**Example**:
```[language]
[Example code]
```
**Related**: [Other patterns, files, decisions]
```
```

## Practical Workflows

### Workflow 1: End-of-Session Preservation

At the end of a session (or before context runs out):

```
Before we wrap up, let me save our progress:

1. Current state → CURRENT_TASK.md
2. Decisions made → DECISIONS.md
3. Key findings → MEMORY.md

Next session should:
- Resume from: [file]
- Priority: [task]
- Blocker: [issue if any]
```

### Workflow 2: Mid-Conversation Checkpoint

When conversation is getting long (~50% context):

```
Checkpoint - saving our progress:

What's been completed:
- [Item 1]
- [Item 2]

What's in progress:
- [Item]: [current state]

What needs to happen:
- [Next action]

Let me update the memory files...
```

### Workflow 3: Context Recovery

When starting fresh after context was lost:

```
Context appears to have been reset. Let me check our project memory:

1. Read MEMORY.md
2. Read CURRENT_TASK.md
3. Read DECISIONS.md

Based on these files, it looks like we were working on [X].
The last decision was [Y]. 

Can you confirm this understanding before we continue?
```

### Workflow 4: Proactive Externalization

When critical information appears:

```
IMPORTANT - Saving to project memory:
```

*Then write the information to the appropriate external file and summarize briefly in context.*

## Context-Aware Behaviors

### Signs Context is Running Low

```
Watch for:
- Earlier parts of conversation being referenced as "mentioned earlier"
- Difficulty connecting current work to previous decisions
- Vague references to "the file we discussed"
- Repeated questions about established facts
```

### When You Notice Memory Loss

```
If you notice context has been lost:

1. Check external files if available
2. Ask user to confirm critical context
3. Rebuild context from project files
4. Re-establish current state clearly

Don't pretend you remember if you don't.
```

### Budgeting Context

```
Context budget estimate:
- System prompt: ~3K tokens
- Skill files loaded: ~1-5K tokens each
- Available for conversation: ~[Context Limit] - [Overhead]

Rule of thumb:
- 80% of available = start summarizing
- 90% = must summarize now
- 95% = emergency preservation mode
```

## Prompt Engineering for Context

### Explicit Context Requests

When you need the user to re-provide context:

```
To continue effectively, I need some context:

1. What were we working on?
2. What was the last decision made?
3. What files are relevant?
4. What's the immediate next step?

You can paste relevant code snippets or error messages if that helps.
```

### Summary Requests

When you need a summary from the user:

```
Our context is getting full. To continue effectively, please summarize:

1. The current task/problem
2. What we've tried so far
3. The current state (what's working/not working)
4. What you want to happen next

This helps me maintain continuity even after context refreshes.
```

## External Tools Integration

### File-Based Memory (No External Tools)

```
Always available:
- Create .context/ directory in project
- Write MEMORY.md, DECISIONS.md, CURRENT_TASK.md
- Read these files when context refreshes
- Update them as decisions change
```

### Git-Based Memory

```
For version-controlled memory:

.gitmessage template (for commits):
```
TYPE: Brief description

Context: [Why this change]
Decisions: [What was decided]
Files: [What was changed]
```

This creates a searchable history of decisions.
```

### Bookmark-Based Memory

```
Important positions/bookmarks:
- File:Line references
- Decision locations
- Configuration keys

Keep a "bookmarks.md" file:
```markdown
# Project Bookmarks

## Key Files
- Main entry: src/index.ts:1
- Config: config/default.yaml:15
- Auth: src/middleware/auth.ts:42

## Key Functions
- init(): src/core/init.ts:100
- process(): src/core/process.ts:25
```

```

## Best Practices

1. **Externalize Early** - Don't wait until context is full
2. **Be Specific** - Vague summaries are useless after reset
3. **Verify Recovery** - Always confirm context was restored correctly
4. **Version Memory** - Update memory files, don't overwrite without backup
5. **Include Citations** - Note where information came from
6. **Mark Uncertainty** - If you're not sure, say so
7. **Persist Preferences** - User preferences go to external memory immediately

## Summary Techniques

### When Summarizing Code

```markdown
## Code Summary: [File/Module]

**Purpose**: [What this code does]

**Key Components**:
- [Component 1]: [Purpose]
- [Component 2]: [Purpose]

**Important Patterns**:
- [Pattern]: [Where used]

**Dependencies**:
- [Module A]
- [Module B]

**Known Issues**:
- [Issue]: [Location/Status]
```

### When Summarizing Discussion

```markdown
## Discussion Summary: [Topic]

**Started with**: [Initial problem/question]

**Explored**:
- [Approach 1]: [Finding]
- [Approach 2]: [Finding]

**Decided**: [Final choice]

**Rationale**: [Why this choice]

**Next Steps**: [Action items]
```

## Recovery Scripts

When context resets, run this protocol:

```
1. "Let me check our project memory..."
2. Read .context/MEMORY.md
3. Read .context/DECISIONS.md
4. Read .context/CURRENT_TASK.md
5. Summarize what I found
6. "Does this match what you remember?"
7. If gaps exist, ask user to fill them
```

## Warning Signs and Actions

| Warning Sign | Action |
|--------------|--------|
| "Earlier we discussed..." | Context is tight, summarize NOW |
| Vague references | External memory may have gaps |
| Repeated clarification requests | Context was likely lost |
| Inconsistent state | Recovery protocol needed |

## Final Principle

**The goal is not to remember everything, but to remember what matters and have a system to recover it.**

This skill exists because:
1. LLMs have hard context limits
2. Compression loses nuance and detail
3. Not all information is equally important
4. External memory persists across sessions
5. Structured extraction enables recovery

Use this skill proactively, not just when problems occur.
