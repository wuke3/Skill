---
name: token-saver
description: >
  A universal skill for reducing LLM token consumption across all task types. Use this skill whenever the user
  mentions saving tokens, reducing token usage, lowering API costs, optimizing prompts for efficiency,
  managing context windows, or wants to make their LLM interactions more cost-effective. Also trigger when
  the user asks about prompt optimization, context management, output control, or conversation efficiency.
  This skill covers prompt writing optimization, context window management, output control, caching strategies,
  structured output, multi-turn conversation management, and model selection for cost efficiency.
---

# Token Saver — Universal LLM Token Optimization

This skill teaches you how to systematically reduce token consumption in every LLM interaction — whether
you're writing prompts, managing conversations, building agents, or optimizing API costs. The strategies
here are model-agnostic and applicable to any LLM (Claude, GPT, Gemini, DeepSeek, etc.).

## Core Philosophy

Token is a finite resource with diminishing marginal returns. More tokens ≠ better results. The goal is
to maximize the **signal-to-noise ratio** of every token — both input and output.

Three foundational principles:

1. **Output tokens cost 2-5x more than input tokens** — controlling output length is the highest-ROI optimization
2. **Give the model a table of contents, not an encyclopedia** — point to knowledge locations rather than embedding all knowledge
3. **Measure before optimizing** — establish a baseline, apply one change at a time, measure the impact

---

## Strategy 1: Prompt Writing Optimization

This is the highest-impact optimization (50-70% token reduction possible) and should be applied to every prompt.

### 1.1 Eliminate Redundancy

Remove filler words, polite phrases, and unnecessary context. Use direct imperative sentences.

| Wasteful (127 tokens) | Optimized (38 tokens) |
|---|---|
| You are a helpful assistant. Please help me with the following task. I would like you to analyze the following text and provide me with a summary. Here is the text: [text]. Please provide a concise summary. | Summarize key points: [text] |

**Rules:**
- Delete: "请你帮我", "我想要你", "能否请你", "非常详细地", "尽可能地"
- Replace: "例如" → "如", "但是" → "但", "因此/所以" → "故"
- Merge multiple related instructions into single sentences
- Use lists instead of paragraphs (lists naturally eliminate transition words)

### 1.2 Use Structured Markdown

"Clean natural language + structured Markdown" is the optimal balance — 57% token reduction without increasing parsing difficulty.

- Use headers (`##`, `###`) instead of "关于...的部分"
- Use bullet points instead of flowing prose
- Use tables for structured comparisons
- Use code blocks for examples and templates

**Avoid custom compression formats.** BPE tokenizers are trained on natural language and Markdown. Custom symbols like `KEY=val;val` often produce *more* single-character tokens than plain text. 81% of savings come from deleting redundant text, not from encoding tricks.

### 1.3 Minimize Few-Shot Examples

- 1-3 examples are usually sufficient
- Keep examples minimal — delete unnecessary words
- Share common prefixes across examples to avoid repeating instructions
- If the task is simple enough, zero-shot with a clear instruction often works

### 1.4 "Fill in the Blank" Over "Open Essay"

Provide scaffolding (pseudocode, templates, outlines) so the model fills in specifics rather than generating from scratch. This is both more token-efficient and more accurate.

Bad: "Write a function that sorts a list of users by age"
Good: "Complete this function:\n```python\ndef sort_users(users):\n    # Sort by age, descending\n    return sorted(users, key=lambda u: u.age, reverse=True)\n```"

### 1.5 Specify Output Format Upfront

Ambiguous outputs lead to wasted tokens on unnecessary explanations. Specify the exact format you want:

- "Return JSON only, no explanation"
- "Reply in ≤50 words"
- "Use this exact template: [template]"
- "Only output the final code, no comments"

---

## Strategy 2: Context Window Management

Context window is the model's "workspace." Everything in it gets processed on every token generation step.
Attention computation is O(n²), so larger contexts mean disproportionately higher costs and latency.

### 2.1 Layered Knowledge Architecture

Don't embed all knowledge into the system prompt. Use three layers:

| Layer | Content | Strategy | Token Cost |
|---|---|---|---|
| **Always loaded** | Type definitions, key interfaces, config schemas, critical constraints | Inline in system prompt | Low (high value per token) |
| **Index layer** | File maps, endpoint directories, event names, error codes | Path + one-line summary | Medium (enables tool calls) |
| **On-demand** | Full file contents, web pages, API responses, large datasets | Provide URL/path only, model fetches as needed | Minimal |

**Key insight:** The model needs to know *what exists* and *where to find it*, not the full content of everything.

### 2.2 Precision Referencing

When working with code or documents, reference specific locations rather than entire files:

- Reference specific functions/classes: `#UserService.login` instead of `#UserService.ts`
- Specify line ranges: "Check lines 88-105 of /src/utils/auth.go"
- Only add files directly relevant to the current task
- Use search tools to find relevant code sections rather than dumping entire files

### 2.3 Noise Filtering — The Signal-Noise Inversion Principle

In agent observations (tool outputs, test results, logs), normal results are noise and anomalies are signal.

Example: 97 tests pass + 3 fail → the 97 passing tests occupy 99% of output but contain zero useful information. Only the 3 failures matter.

**Solutions:**
- Write custom test scripts that only output failures and key error messages
- Filter tool outputs to extract only relevant results before including in context
- Use lightweight models to pre-filter and summarize large outputs before feeding to the main model
- Set clear "reading goals" before the model accesses large files

### 2.4 Context Window Pruning Strategies

For multi-turn conversations, apply pruning when context grows too large:

| Strategy | Method | Best For |
|---|---|---|
| **OldestFirst** | Delete oldest messages, keep system + recent turns | General chat, FAQ bots |
| **MiddleOut** | Keep beginning (system + setup) and end (recent), delete middle | Complex reasoning that needs initial context |
| **RecentN** | Keep only last N complete turns | Turn-based games, structured dialogues |
| **Adaptive** | Auto-calculate optimal window based on model size and cost sensitivity | Production environments |

**Best practices:**
- Always preserve system messages
- Reserve space for output (e.g., if context is 100K, use 70K for input, leave 30K for output)
- Keep at minimum the 3 most recent conversation turns
- Use token-based limits, not message-count limits

---

## Strategy 3: Output Control

Since output tokens cost 2-5x more than input tokens, controlling output length has outsized impact on cost.

### 3.1 Hard Limits

- Set `max_tokens` on all API requests to prevent runaway outputs
- Use `stop` sequences to halt generation at logical boundaries (e.g., `stop=["END", "\n\n\n"]`)

### 3.2 Prompt-Level Output Constraints

- "Answer in ≤50 words"
- "Only output the final result, no analysis"
- "Return JSON only"
- "List 3 items maximum"
- "No preamble, start directly with the answer"

### 3.3 Structured Output Formats

JSON and other structured formats eliminate redundant natural language:

Bad: "Please extract the person's name, age, and occupation from this text and present the results clearly."
Good: "Extract to JSON: {name, age, occupation}"

### 3.4 Stream + Early Termination

Use streaming output so you can detect when the model goes off-track and terminate early, avoiding wasted tokens on incorrect continuations.

### 3.5 Interrupt Invalid Outputs

When you notice the model producing incorrect or irrelevant content, stop generation immediately. Don't wait for it to finish hundreds of lines of wrong output.

---

## Strategy 4: Conversation Management

Multi-turn conversations accumulate token debt. Manage them actively.

### 4.1 One Task Per Conversation

- Start a new conversation when switching tasks
- Carry forward only conclusions and key artifacts (not full history)
- If the model fails 3 consecutive times on the same issue, start fresh

### 4.2 Conversation Compression

Compress when:
- A task phase completes and you're moving to the next
- Response quality degrades (vague answers, repeating solved issues)
- The topic shifts slightly

Methods:
- Manual: Summarize key decisions and carry them forward
- Automatic: Use built-in context compression when available

### 4.3 Merge Related Tasks

- Batch similar operations into a single request
- Run parallel searches in one prompt instead of sequential requests
- Package complete task flows: "Do A, then B, then C" in one message

### 4.4 Plan Before Execute

For complex tasks, use a plan-then-execute pattern:
1. First request: Generate a detailed execution plan (files to create, functions to modify)
2. Review and approve the plan
3. Second request: Execute the approved plan

This avoids the model going down wrong paths and wasting tokens on incorrect approaches.

---

## Strategy 5: Caching and Reuse

### 5.1 Prompt Caching (Prefix Caching)

Place static content (system instructions, reference docs, few-shot examples) at the **beginning** of the prompt. Most major providers (OpenAI, Anthropic, Google) offer automatic prefix caching:

- Cached tokens cost 50-90% less
- Content must be byte-identical to hit cache
- Minimum cacheable size varies (1024-2048 tokens depending on provider)
- Cache TTL is typically 5-60 minutes; send at least one request within the window to keep it "warm"

### 5.2 Semantic Caching

For applications with repetitive queries, use semantic caching:
- Convert queries to vector embeddings
- Search for semantically similar cached queries (similarity threshold ~0.8)
- Return cached response if match found
- At threshold 0.8: ~70% cost reduction with 95%+ answer accuracy

### 5.3 Batch Processing

For non-real-time tasks, use batch APIs (50% discount from most providers):
- Data labeling, content generation, report generation, bulk translation, dataset synthesis

---

## Strategy 6: Model Selection for Cost Efficiency

Not every task needs the most powerful (and expensive) model. Use the right tool for the job.

### Decision Framework

| Scenario | Recommended Model Tier |
|---|---|
| Simple extraction, formatting, classification | Small/cheap model |
| Code generation from clear specs | Medium model |
| Complex reasoning, architecture design | Large model |
| Multimodal input (images, design mockups) | Multimodal model |
| High-volume, cost-sensitive workloads | Cheapest adequate model |
| Novel problems requiring creative solutions | Largest available model |

### Cost Optimization Levers

- **Model routing**: Automatically classify query complexity and route to appropriate model tier (50-70% cost reduction for routed queries)
- **Fine-tuned small models**: For repetitive specialized tasks, a fine-tuned small model often outperforms a large general model at a fraction of the cost
- **Cascading**: Try a cheap model first; only escalate to expensive models if confidence is low

---

## Strategy 7: RAG and Retrieval Optimization

RAG (Retrieval-Augmented Generation) is a major token consumer — every retrieved chunk adds to the input.
Optimizing retrieval directly reduces token costs while often improving answer quality.

### 7.1 Improve Retrieval Precision

Better retrieval means fewer chunks needed, which means fewer tokens injected into the prompt.

- **Hybrid search**: Combine semantic vector search + BM25 keyword matching. Each catches what the other misses.
- **Cross-encoder reranking**: After initial retrieval, rerank with a cross-encoder. 3 high-quality chunks > 10 mediocre ones.
- **Metadata filtering**: Pre-filter by document type, date, category, or other metadata to narrow the search space.
- **Optimal chunk size**: 512-1024 tokens balances semantic completeness and retrieval precision. Smaller chunks retrieve more precisely but may lack context; larger chunks are more self-contained but noisier.
- **Semantic chunking**: Split by meaning (paragraph boundaries, topic shifts) rather than fixed character counts.

### 7.2 Compress Retrieved Context

- Strip Markdown formatting from retrieved chunks before injection
- Consider chunk-level summaries for large documents (summarize once, reuse the summary)
- Only include top-K most relevant chunks (K=3-5 is usually sufficient)
- Deduplicate overlapping content across chunks

### 7.3 Query Transformation

Before retrieval, optimize the user's query:
- Rewrite vague queries into specific search terms
- Decompose complex queries into sub-queries
- Generate hypothetical answer fragments to use as search anchors (hyde technique)

---

## Strategy 8: Batch Processing and Async Optimization

Not all LLM calls need to happen in real-time. Batching provides significant cost savings.

### 8.1 Batch APIs

Most providers offer batch APIs at ~50% discount:
- OpenAI Batch API: 50% off, 24-hour processing window
- Anthropic Message Batches: 50% off
- Suitable for: data labeling, content generation, report generation, bulk translation, dataset synthesis, evaluation runs

### 8.2 Request Deduplication

- Cache identical requests within a short time window
- For the same user asking the same question repeatedly, return the cached response
- Deduplicate concurrent requests to the same endpoint

### 8.3 Async Processing Patterns

- Use streaming to improve perceived latency (doesn't save tokens but improves UX)
- Pre-generate responses for predictable high-frequency queries during off-peak hours
- Queue non-urgent tasks for batch processing windows

---

## Strategy 9: Cost Monitoring and Governance

Optimization without measurement is guesswork. Establish monitoring to track the impact of every change.

### 9.1 Key Metrics to Track

| Metric | Why It Matters | Target |
|---|---|---|
| **Cost per request** (by type) | Identifies the most expensive request patterns | Decreasing trend |
| **Input/output token ratio** | Output tokens cost 2-5x more; high ratio = optimization opportunity | < 1:3 for most tasks |
| **Cache hit rate** | Measures caching effectiveness | 60-80% |
| **Model routing distribution** | Shows what % goes to each model tier | 70%+ on cheapest adequate tier |
| **Average conversation length** | Long conversations = high token debt | 5-8 turns for most tasks |
| **User satisfaction / task completion rate** | Ensures optimization doesn't degrade quality | No degradation |

### 9.2 Governance Rules

- Set per-user or per-project token budgets with alerts
- Define maximum context window usage thresholds (e.g., alert at 70% of window)
- Require max_tokens on all API requests
- Automate fallback to cheaper models when budgets are tight
- Review and prune unused tools/skills from agent configurations regularly

### 9.3 Optimization Workflow

1. **Baseline**: Measure current cost per request, token distribution, and quality metrics
2. **Apply one change**: Implement a single optimization strategy
3. **Measure**: Compare against baseline — both cost AND quality
4. **Decide**: Keep if quality is maintained and cost decreased; revert if quality drops
5. **Repeat**: Move to the next strategy in priority order

**Never optimize cost at the expense of output quality.** The goal is to maintain or improve quality while reducing token consumption.

---

## Quick Reference: Token Audit Checklist

Before sending any prompt, run through this checklist:

- [ ] **Redundancy check**: Are there filler words, polite phrases, or repeated instructions?
- [ ] **Format check**: Is the prompt using structured Markdown instead of prose?
- [ ] **Scope check**: Am I only including information directly relevant to this task?
- [ ] **Output check**: Have I specified the exact output format and length constraints?
- [ ] **Context check**: Is the context window filled with noise that could be filtered?
- [ ] **Cache check**: Can static content be positioned for prefix caching?
- [ ] **Model check**: Am I using the cheapest model adequate for this task?
- [ ] **Merge check**: Can multiple requests be combined into one?
- [ ] **RAG check**: If using retrieval, am I using top-3 reranked chunks instead of top-10 raw results?
- [ ] **Batch check**: Is this request time-sensitive, or can it wait for batch processing?

---

## Implementation Priority (ROI Ranked)

| Priority | Strategy | Conservative Savings | Optimistic Savings | Effort |
|---|---|---|---|---|
| 1 | Prompt writing optimization | 30-40% | 50-70% | Low |
| 2 | Output length control | 20-30% | 30-40% | Low |
| 3 | Prompt caching | 40-50% (cached portion) | 60-80% (cached portion) | Low |
| 4 | Conversation management | 15-25% | 20-40% | Low |
| 5 | Context window pruning | 15-30% | 20-50% | Medium |
| 6 | Model routing/selection | 30-50% (routed queries) | 50-70% (routed queries) | Medium |
| 7 | RAG optimization | 30-50% (retrieval context) | 60-80% (retrieval context) | Medium |
| 8 | Semantic caching | 40-60% (cacheable queries) | 70-86% (cacheable queries) | High |
| 9 | Batch processing | 30-50% (batchable tasks) | 50% (provider discount) | Low |

**Realistic combined effect: 50-70% total cost reduction is readily achievable; 80%+ requires full-stack optimization across all strategies.**
