# Prompt Before/After Examples

Real-world examples showing token optimization. Use these as reference when helping users optimize their prompts.

---

## Example 1: Code Generation

**Before (186 tokens):**
```
You are an experienced Python developer. I need your help to write a function.
Please write a Python function that takes a list of dictionaries as input, where each
dictionary has a "name" key and an "age" key. The function should sort the list by age
in descending order and return only the names of the top 3 oldest people. Please make
sure the code is clean and well-documented. Thank you!
```

**After (62 tokens):**
```
Python function: input=list of {name, age} dicts.
Sort by age descending, return top 3 names.
```

**Savings: 67%**

---

## Example 2: Data Extraction

**Before (210 tokens):**
```
I have this text and I need you to extract some information from it. Could you please
go through the text below and find the company name, the annual revenue, the number
of employees, and the headquarters location? Please format your response in a clear
and organized way so I can easily read it.

[text]
```

**After (48 tokens):**
```
Extract to JSON: {company, revenue, employees, hq_location}
From: [text]
```

**Savings: 77%**

---

## Example 3: Bug Fix

**Before (320 tokens):**
```
Hi, I'm having some trouble with my code and I was hoping you could help me figure out
what's going wrong. I've been working on this React component and when I try to render
it, I get this error message. I've tried a few things but nothing seems to work. Here's
my component code (it's pretty long, sorry about that):

[entire 200-line component file]

And here's the error I'm getting:
TypeError: Cannot read properties of undefined (reading 'map')
  at UserList (UserList.js:45:23)
```

**After (95 tokens):**
```
Fix this error in React component UserList.js:45:
"TypeError: Cannot read properties of undefined (reading 'map')"

Relevant code (lines 40-50):
[code snippet]
```

**Savings: 70%**

---

## Example 4: Document Summary

**Before (165 tokens):**
```
Please read the following document and provide me with a comprehensive summary. I'd like
you to cover the main points, key arguments, and any important conclusions or
recommendations that the author makes. Please make the summary detailed but concise,
around 200 words.

[document]
```

**After (35 tokens):**
```
Summarize in ≤200 words:
- Main points
- Key arguments
- Conclusions & recommendations

[document]
```

**Savings: 79%**

---

## Example 5: Multi-Task Request

**Before (3 separate requests, ~400 tokens total):**
```
Request 1: "Can you translate this text to English?"
Request 2: "Now can you summarize it?"
Request 3: "Can you also extract the key entities?"
```

**After (1 request, ~80 tokens):**
```
For the text below, provide:
1. English translation
2. 50-word summary
3. Key entities as JSON array

[text]
```

**Savings: 80%**

---

## Example 6: System Prompt Optimization

**Before (2800 tokens):**
```
You are an AI assistant designed to help users with their software development tasks.
You have extensive knowledge of programming languages including Python, JavaScript,
TypeScript, Java, C++, Go, Rust, and many others. You can help with writing code,
debugging, code review, architecture design, and general software engineering questions.

When the user asks you a question, you should:
1. First, carefully analyze what they're asking
2. Think about the best approach to solve their problem
3. Provide a clear, well-structured response
4. Include code examples when appropriate
5. Explain your reasoning when the question is complex

Always be helpful, accurate, and concise. If you're not sure about something, say so.
Format code blocks with proper syntax highlighting. Use Markdown for formatting.
[... 2000 more tokens of similar content ...]
```

**After (450 tokens):**
```
## Role
Software engineering assistant. Expert in Python, JS/TS, Go, Rust, Java, C++.

## Response Rules
- Code blocks with syntax highlighting
- Explain reasoning for complex questions
- Admit uncertainty; don't guess
- Be concise; skip obvious explanations

## Output Format
Markdown with code blocks. JSON for structured data when requested.
```

**Savings: 84%**

---

## Key Patterns

1. **Delete role-playing preamble** — "You are an experienced..." adds zero value
2. **Delete instructions the model already knows** — "Use proper formatting", "Be helpful"
3. **Specify constraints, not processes** — "Return JSON" > "Please format your response as JSON"
4. **Merge sequential requests** — one detailed prompt > three vague ones
5. **Provide structure, not freedom** — templates and schemas > open-ended instructions
