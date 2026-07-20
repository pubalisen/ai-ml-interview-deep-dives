# Part 14: Multi-Turn Conversations

> **The Question:** "How do you handle multi-turn conversations with LLMs?"

---

## The Technical Breakdown

### The Fundamental Challenge

LLMs are **stateless** — every API call is independent. "Conversation memory" is implemented by **re-sending the full conversation history** in each request:

```python
messages = [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "My name is Alice."},          # Turn 1
    {"role": "assistant", "content": "Hello Alice!"},           # Turn 1 response
    {"role": "user", "content": "What's my name?"},             # Turn 2
    # The model sees the full history and can answer "Alice"
]
```

### Managing Context Growth

| Strategy | How | When |
|----------|-----|------|
| **Full history** | Send all turns | Short conversations (< 20 turns) |
| **Sliding window** | Keep last N turns | Moderate conversations |
| **Summarization** | Summarize old turns, keep recent verbatim | Long conversations |
| **RAG on history** | Embed turns, retrieve relevant ones | Very long / multi-session |
| **Structured memory** | Extract key facts into a structured store | Persistent assistants |

### Context Window Budget

```
Available context = Model limit - System prompt - Output reserve
Example: 128K - 500 tokens - 2000 tokens = ~125,500 tokens for history

At ~50 tokens/turn (user + assistant), that's ~2,500 turns before overflow.
```

---

## Scenario Examples
### A: A coding assistant maintains the full conversation for a debugging session (typically 10-20 turns). Each turn includes code context. At turn 15, the history exceeds 32K tokens, so the system summarizes turns 1-10 into a 200-token summary and keeps turns 11-15 verbatim.
### B: A customer support bot extracts structured state after each turn: `{issue: "billing error", account: "12345", status: "escalated"}`. Even if conversation history is truncated, this structured context ensures continuity.

## Follow-Up Questions
### Q1: "How do you handle turn-level context relevance?"
**Answer:** Not all previous turns are equally relevant to the current query. Use embedding similarity to score which past turns are most relevant to the current question, and prioritize those in the context window. This is especially effective for conversations that span multiple topics.

### Q2: "How do session-based vs. persistent conversations differ?"
**Answer:** Session-based: conversation resets when the user closes the chat. Persistent: conversation history is stored in a database and loaded for returning users. Persistent conversations require a storage backend (database, file system) and careful privacy handling.

### Q3: "What's the cost impact of multi-turn?"
**Answer:** Every turn re-sends the full history, so input tokens grow linearly with conversation length. A 20-turn conversation sends the system prompt 20 times and early turns 20 times each. Prompt caching (caching the KV cache of the shared prefix) reduces this cost by 50-90%.

---

<p align="center"><a href="../../README.md">← Back to all questions</a></p>
