# Scenario 13: Long Conversation Context Loss

> **The Scenario:** "Your chatbot loses context after 10 turns in a conversation. How do you maintain a long conversation context?"

---

## Solutions
1. **Conversation summarization** — Periodically summarize older turns into a compact summary, keeping recent turns verbatim. This compresses history while preserving key information.
2. **Sliding window + summary** — Keep the last N turns verbatim, summarize everything before that.
3. **RAG on conversation history** — Embed all turns in a vector store. Retrieve relevant past turns based on the current query instead of including everything.
4. **Longer context model** — Use a 128K+ context model that can hold 100+ turns without truncation.
5. **Structured memory** — Extract key facts from conversation into structured storage (user name, preferences, issue description) and inject them into each turn.

## Scenario Examples
### A: A support bot summarizes the conversation every 5 turns: "Customer reported billing error on June 15. Account #12345. Attempted fix: reset billing cycle. Issue persists." This 30-token summary replaces 2000 tokens of raw chat history.
### B: A therapy chatbot uses structured memory: extracts and stores {mood: "anxious", topics: ["work stress", "insomnia"], session_count: 5}. This persistent context is injected into every session, regardless of conversation length.

## Follow-Up Questions
### Q1: "When should you summarize vs. use RAG?"
**Answer:** Summarize when the entire conversation arc matters (support tickets, therapy sessions). Use RAG when only specific past statements are relevant (knowledge-intensive Q&A where the user mentioned something 20 turns ago).

### Q2: "How do you handle contradictions in long conversations?"
**Answer:** Use timestamps in the structured memory. "User prefers product A (turn 3)" vs "User now prefers product B (turn 15)" — the latest information takes precedence. The system prompt should instruct the model to use the most recent user statements.

### Q3: "What is the 'lost in the middle' impact on conversations?"
**Answer:** In a 50-turn conversation stuffed into context, the model attends most to the first few turns (system prompt) and the last few turns (recent context). Middle turns (turns 10-40) are effectively invisible. Summarization solves this by compressing middle turns into the end of the context.

---

<p align="center"><a href="../../README.md">← Back to all questions</a></p>
