# Scenario 14: Topic Switching in Chat

> **The Scenario:** "Your chatbot fails when users switch topics mid-conversation. How do you handle topic switches?"

---

## Solutions
1. **Topic detection classifier** — Lightweight classifier that detects topic changes. On switch, adjust system prompt context or reset relevant state.
2. **Per-topic memory slots** — Maintain separate context summaries per topic. When the user switches back, retrieve that topic's context.
3. **Explicit system prompt guidance** — "Users may switch topics. Always address the current topic. If the user returns to a previous topic, reference earlier context."
4. **Hybrid retrieval** — RAG retrieval over the full conversation, keyed by topic. The current query retrieves relevant past turns regardless of when they occurred.

## Scenario Examples
### A: User discusses a billing issue for 5 turns, then asks about a product feature, then returns to billing. Per-topic memory stores {billing: "issue with invoice #123, awaiting refund"} and {product: "asking about feature X"}. On return, billing context is seamlessly restored.
### B: A coding assistant switches between Python and JavaScript. Topic detection identifies the language change and swaps the relevant documentation context in the system prompt.

## Follow-Up Questions
### Q1: "How do you detect topic switches?"
**Answer:** Use embedding similarity — if the current message's embedding has low cosine similarity to the previous turn's embedding, it's likely a topic switch. Threshold: cosine similarity < 0.5 typically indicates a switch.

### Q2: "Should you clear context on topic switch?"
**Answer:** No — never clear context entirely. Users often reference earlier topics. Instead, demote older topic context (summarize it) and promote the new topic context (expand it).

### Q3: "How do multi-agent systems handle topic switching?"
**Answer:** Route to different specialized agents per topic: billing agent, product agent, technical support agent. A router agent classifies the topic and dispatches to the appropriate specialist, each maintaining their own conversation state.

---

<p align="center"><a href="../README.md">← Back to all questions</a></p>
