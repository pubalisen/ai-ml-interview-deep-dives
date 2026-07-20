# Scenario 2: Context Window Overflow

> **The Scenario:** "Your LLM-powered tool hits the context window limit on long documents. How do you handle it?"

---

## Solutions

### 1. Chunking + Map-Reduce
Split the document into overlapping chunks, process each independently, then combine results:
```
Document (50K tokens) → Chunk 1 (4K) → Summary 1
                      → Chunk 2 (4K) → Summary 2
                      → ...
                      → Combine all summaries → Final answer
```

### 2. RAG (Retrieval-Augmented Generation)
Don't feed the entire document. Embed chunks in a vector database, retrieve only the relevant chunks for each query. Typical retrieval: 3-10 chunks (3-10K tokens) regardless of document size.

### 3. Hierarchical Summarization
Summarize chunks recursively: Level 1 (chunk summaries) → Level 2 (summary of summaries) → Final summary. Preserves key information across arbitrary document lengths.

### 4. Use a Longer-Context Model
Switch from 4K→128K or 1M context models. But remember: longer context ≠ better retrieval. "Lost in the middle" still applies.

---

## Accuracy Check
| Solution | Max Doc Size | Quality | Cost |
|----------|-------------|---------|------|
| Chunking + Map-Reduce | Unlimited | Good for summarization | Medium |
| RAG | Unlimited | Best for Q&A | Low per query |
| Hierarchical | Unlimited | Good for compression | High (multiple passes) |
| Longer context | Model limit | Best (no information loss) | Highest (more tokens) |

---

## Scenario Examples

### A: A legal platform processes 200-page contracts. RAG retrieves only clauses relevant to each user query, keeping context under 8K tokens regardless of contract length.

### B: A research assistant summarizes 50 papers. Hierarchical summarization: summarize each paper (Level 1), then summarize the 50 summaries (Level 2), producing a literature review.

---

## Follow-Up Questions

### Q1: "How do you handle cross-chunk dependencies?"
**Answer:** Use overlapping chunks (e.g., 200-token overlap) so information at chunk boundaries appears in both adjacent chunks. For stronger coherence, use a sliding window or iterative refinement where later chunks can reference earlier summaries.

### Q2: "When is RAG better than stuffing the full document?"
**Answer:** RAG is better when: the document is very long (>50K tokens), only a small portion is relevant to any given query, and you're doing Q&A rather than holistic analysis. Full-document processing is better for summarization, translation, or analysis requiring global context.

### Q3: "How do you choose chunk size?"
**Answer:** Balance between context (larger chunks) and precision (smaller chunks). Typical: 500-1000 tokens with 100-200 token overlap. Use semantic chunking (split at paragraph/section boundaries) rather than fixed-size splits.

---

<p align="center">
  <a href="../README.md">← Back to all questions</a>
</p>
