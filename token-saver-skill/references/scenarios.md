# Scenario-Specific Token Optimization Guide

This reference provides detailed, scenario-based optimization strategies. Consult the relevant section
when the user's task falls into a specific category.

---

## Coding Tasks

### Code Generation
- Provide function signatures, type definitions, and interfaces instead of full implementations
- Give pseudocode or skeleton code for the model to fill in
- Specify the exact file path, function name, and constraints upfront
- "Complete this function" > "Write a function that..."

### Code Review
- Reference specific files and line ranges, not entire codebases
- Ask for specific aspects: "Check for security issues in auth.go lines 40-80"
- Use structured output: "Return findings as JSON array: {file, line, severity, issue, fix}"

### Debugging
- Paste only the error message and the 5-10 lines around it, not the entire file
- Include the stack trace but truncate repetitive frames
- "Fix this error: [error]" > "I'm getting an error, can you help me figure out what's wrong?"

### Refactoring
- Describe the current structure briefly, then specify the target structure
- Provide the specific files/functions to refactor
- Use /plan pattern: first generate the refactoring plan, approve, then execute

---

## Document & Content Tasks

### Summarization
- Specify target length: "Summarize in ≤100 words"
- Specify focus: "Focus on financial metrics and risk factors"
- Specify format: "Output as 3 bullet points"

### Translation
- Batch multiple short texts into one request
- Provide glossary for domain-specific terms (saves back-and-forth)
- "Translate to English, preserve Markdown formatting"

### Data Extraction
- Use JSON schema to define expected output structure
- "Extract to JSON: {name, date, amount, category}" > "Please extract the information"

### Content Generation
- Provide a template or outline for the model to fill in
- Specify tone, length, and audience in one sentence
- "Write a 200-word product description for [product], professional tone, highlight [features]"

---

## Multi-Turn Conversations

### Customer Support / FAQ
- Use RAG with high-precision retrieval (top 3 chunks, reranked)
- Cache common question-answer pairs (semantic caching)
- Set max_tokens to prevent overly long responses
- Route simple queries to small models, complex ones to large models

### Brainstorming / Creative Tasks
- Start with a structured prompt that defines constraints
- Use "generate 5 options" to get multiple concise outputs
- Avoid open-ended "help me think about..." prompts that encourage rambling

### Iterative Development
- Compress conversation at phase boundaries
- Carry forward only: decisions made, files modified, errors encountered
- Start fresh when switching subtasks

---

## Agent / Tool-Use Tasks

### Tool Definition Optimization
- Each tool's JSON Schema description adds to every request's token count
- Remove unused tools from the tool list (most impactful single action)
- Prefer Skills (loaded on-demand) over MCP tools (always in context)
- Keep tool descriptions minimal but unambiguous

### Tool Output Processing
- Filter tool outputs before including in context
- For test results: only include failures, not successes
- For file reads: only include relevant sections, not entire files
- For search results: summarize top results, don't include full content

### Agent Loop Control
- Set maximum iteration counts
- Detect loops (same action repeated 3x) and break
- Use lightweight models for sub-tasks (search, filtering, summarization)
- Reserve heavy models for final synthesis and decision-making

---

## API / Production Optimization

### Request-Level
- Always set max_tokens
- Use stop sequences for predictable output patterns
- Enable prompt caching by structuring requests with static prefix first
- Batch non-real-time requests

### System-Level
- Implement semantic caching layer (L1 memory + L2 Redis + L3 vector DB)
- Build model routing based on query complexity classification
- Monitor: per-request cost, input/output token ratio, cache hit rate, model distribution
- Set budget alerts and automatic fallback to cheaper models

### Cost Monitoring KPIs
- Cost per request (by request type)
- Token usage distribution (input vs output ratio)
- Cache hit rate (target: 60-80%)
- Model routing distribution (what % goes to each tier)
- User satisfaction (ensure optimization doesn't degrade quality)

---

## Chinese Language Specific Optimization

Chinese text consumes roughly 2x more tokens than English for the same meaning, due to how BPE tokenizers work.

### Token-Efficient Chinese Writing

| Wasteful | Optimized | Savings |
|---|---|---|
| 请你帮我分析一下 | 分析 | ~60% |
| 非常详细地描述 | 详细描述 | ~40% |
| 尽可能地优化 | 优化 | ~50% |
| 例如 | 如 | ~50% |
| 但是 | 但 | ~33% |
| 因此 / 所以 | 故 | ~50% |
| 关于...的问题 | ...问题 | ~40% |

### When to Use English vs Chinese

- If the model primarily processes English, writing prompts in English saves ~50% input tokens
- For code-related tasks, use English variable/function names and English prompts
- If using a Chinese-optimized model (e.g., Doubao, DeepSeek), Chinese token efficiency is comparable to English

---

## Frontend / UI Development

### Design-to-Code
- Use multimodal models (Gemini, GPT-4V) to process design mockups directly
- Provide design specs in structured format: "Layout: flexbox, Colors: #primary=#333, Font: 14px Inter"
- Reference component libraries: "Use Ant Design components" > "Make it look professional"

### Iterative UI Refinement
- Paste only the component being modified, not the entire page
- Use screenshots for visual feedback instead of describing issues in words
- "Change button color to blue" > "The button doesn't look right, can you make it more appealing?"

---

## Academic / Research Tasks

### Paper Analysis
- Provide the abstract and key sections, not the full paper
- Ask specific questions: "What is the main contribution of Section 3?" > "Analyze this paper"
- Request structured output: "Return as: {method, dataset, metrics, key_findings, limitations}"

### Literature Review
- Provide a list of paper titles/abstracts, not full texts
- Ask for comparison tables: "Compare these 5 papers on: method, dataset, performance"
- Use RAG to retrieve relevant passages instead of loading full papers

---

## Data Science / Analysis Tasks

### Data Analysis Requests
- Describe the data schema concisely: "CSV: columns=[date, product, revenue, region]"
- Specify the analysis framework: "Use pandas + matplotlib"
- Request specific output: "Return as Markdown table: top 10 products by revenue"

### Model Training / ML Tasks
- Provide model architecture as code skeleton, not prose description
- Specify hyperparameters as a dict, not a paragraph
- "Train XGBoost on this data, optimize for F1" > "Help me build a machine learning model"
