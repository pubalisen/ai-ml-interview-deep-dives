# Part 7: System Prompts

> **The Question:** "What is a system prompt, and how does it influence model behavior?"

---

## The Technical Breakdown

### What Is a System Prompt?

The system prompt is a **persistent instruction block** that sets the model's persona, rules, constraints, and behavior for an entire conversation. It's processed before any user messages:

```python
messages = [
    {"role": "system", "content": "You are a senior financial advisor..."},  # ← System prompt
    {"role": "user", "content": "Should I invest in crypto?"},
    {"role": "assistant", "content": "..."}
]
```

### What Goes in a System Prompt

| Component | Example | Purpose |
|-----------|---------|---------|
| **Role/Persona** | "You are a medical triage nurse" | Sets expertise and tone |
| **Rules** | "Never provide specific dosage recommendations" | Safety guardrails |
| **Output format** | "Always respond in JSON with {answer, confidence}" | Consistent structure |
| **Context** | "Today's date is July 20, 2026" | Ground truth facts |
| **Constraints** | "Keep responses under 100 words" | Control verbosity |
| **Examples** | "Here's an ideal response: ..." | Demonstrate quality bar |

### System Prompt Best Practices

1. **Be explicit** — Don't assume the model will infer constraints
2. **Prioritize early** — Most important rules first (attention decay)
3. **Use positive framing** — "Do X" works better than "Don't do Y"
4. **Test adversarially** — Try to break your own system prompt
5. **Version control** — Treat system prompts as code artifacts

---

## Accuracy Check
| Claim | Verified? |
|-------|-----------|
| System prompts are processed before user messages | ✅ API specification for OpenAI, Anthropic, Google |
| System prompt instructions have highest priority | ✅ By design, though can be overridden by prompt injection |
| Positive framing outperforms negative constraints | ✅ Empirical observation in prompt engineering literature |

## Scenario Examples
### A: A legal chatbot's system prompt includes: "You are a legal information assistant. Provide general legal information. Always add: 'This is not legal advice. Consult an attorney.' Never suggest specific legal actions." This prevents liability while being helpful.
### B: An API assistant's system prompt defines exact JSON schema, error handling formats, and rate limit messages. Every response follows the same parseable structure regardless of the query.

## Follow-Up Questions
### Q1: "Can users override the system prompt?"
**Answer:** System prompts are designed to be authoritative, but prompt injection can override them. Defense: layer multiple instruction sources, use output filtering, and separate the system prompt from user-accessible context. No system prompt is 100% immune to adversarial manipulation.

### Q2: "How long should a system prompt be?"
**Answer:** As short as possible while covering all critical behaviors. 200-500 tokens is typical. Long system prompts (>1000 tokens) consume context window on every request and increase cost. Move detailed examples or policies to RAG retrieval instead.

### Q3: "What is a developer system prompt vs. a user system prompt?"
**Answer:** Some APIs (Anthropic) separate "developer" instructions (from the app developer, highest priority) from "user" instructions (from the end user, lower priority). This hierarchy helps prevent end users from overriding the developer's safety constraints.

---

<p align="center"><a href="../../README.md">← Back to all questions</a></p>
