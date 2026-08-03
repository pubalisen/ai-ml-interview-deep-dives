# Part 10: Re-Ranking

> **The Question:** "What is re-ranking, and how does it improve RAG retrieval quality?"

---

## The Technical Breakdown

### The Two-Stage Retrieval Problem

Vector search is fast but imprecise. It uses **bi-encoder** embeddings — query and document are embedded independently, then compared via cosine similarity. This is efficient (search millions of vectors in milliseconds) but misses nuanced relevance because it never sees query and document together.

```
Bi-Encoder (Retrieval):
  Query  → Encoder → query_vector  ─┐
                                     ├→ cosine_similarity → score
  Doc    → Encoder → doc_vector   ─┘
  
  ❌ Never sees query + doc together. Misses fine-grained relevance.

Cross-Encoder (Re-Ranking):
  [Query + Doc] → Encoder → relevance_score
  
  ✅ Sees both together. Captures nuanced relevance.
```

### How Re-Ranking Works

```
Stage 1: RETRIEVAL (fast, broad)
  Query → Bi-encoder → Top 50 candidates from vector DB
  Latency: ~20ms for 1M documents

Stage 2: RE-RANKING (slow, precise)
  Query + Each of 50 candidates → Cross-encoder → Relevance score
  Re-sort by cross-encoder score → Top 5 final results
  Latency: ~200ms for 50 candidates
```

### Why Re-Ranking Improves Quality

The cross-encoder processes the **concatenation** of query and document through the full Transformer. This enables:

1. **Token-level interaction** — The model attends from query tokens to document tokens directly
2. **Negation understanding** — "Does NOT support Python" vs "Supports Python"
3. **Contextual relevance** — "Apple stock price" matched against a document about Apple Inc. vs apple fruit
4. **Partial match scoring** — A document that answers 80% of the question scores higher than one that answers 30%

### Re-Ranking Models

| Model | Quality | Speed | Type |
|-------|---------|-------|------|
| Cohere Rerank v3 | Excellent | ~100ms/batch | API |
| Jina Reranker v2 | Very good | ~80ms/batch | API + open |
| Voyage Reranker | Excellent | ~120ms/batch | API |
| bge-reranker-v2-m3 | Very good | ~50ms/batch | Open-source |
| ms-marco-MiniLM-L-6-v2 | Good | ~20ms/batch | Open-source |
| FlashRank | Good | ~10ms/batch | Lightweight |

### Implementation Pattern

```python
# Stage 1: Retrieve top-50 candidates
candidates = vector_db.search(query_embedding, top_k=50)

# Stage 2: Re-rank with cross-encoder
reranked = reranker.rerank(
    query=user_query,
    documents=[c.text for c in candidates],
    top_n=5
)

# Stage 3: Use top-5 re-ranked results in prompt
context = "\n\n".join([r.text for r in reranked])
```

### The Quality Impact

Typical improvements from adding re-ranking to a RAG pipeline:

| Metric | Without Re-ranking | With Re-ranking | Improvement |
|--------|-------------------|-----------------|-------------|
| Precision@5 | 0.62 | 0.78 | +26% |
| Recall@5 | 0.71 | 0.84 | +18% |
| NDCG@10 | 0.58 | 0.73 | +26% |
| Answer correctness | 68% | 81% | +19% |

---

## Accuracy Check
| Claim | Verified? |
|-------|-----------|
| Cross-encoders outperform bi-encoders on relevance judgments | ✅ Nogueira & Cho (2019), MS MARCO leaderboard |
| Two-stage retrieve-then-rerank is the standard production pattern | ✅ Industry standard (Google, Bing, Cohere docs) |
| Re-ranking adds ~100-300ms latency for 50 candidates | ✅ Benchmarks from Cohere, Jina, and open-source models |

## Scenario Examples
### A: A technical documentation search retrieves the top-50 results for "How to configure SSL certificates in Kubernetes." Without re-ranking, the top result is a general "Kubernetes security overview" (high semantic similarity to the broad topic). With re-ranking, the cross-encoder promotes the specific "cert-manager installation guide" to #1 because it directly addresses SSL certificate configuration — something the bi-encoder's coarse embedding missed.
### B: A financial analysis tool searches for "Tesla Q3 2024 revenue vs Q2." Without re-ranking, results include Tesla blog posts, general EV market reports, and competitor analyses — all semantically related to "Tesla revenue." The cross-encoder re-ranks the actual Q3 earnings report to #1 because it processes the query and document together and identifies the specific quarter-over-quarter comparison.

## Follow-Up Questions
### Q1: "Why not just use the cross-encoder for initial search?"
**Answer:** Cross-encoders are O(N) — they must process every query-document pair. For 1 million documents, that's 1 million forward passes per query (~hours). Bi-encoders pre-compute document embeddings, making search O(log N) via ANN. The two-stage pattern combines the speed of bi-encoders (milliseconds to search millions) with the quality of cross-encoders (applied to only 20-100 candidates).

### Q2: "How many candidates should you retrieve before re-ranking?"
**Answer:** The sweet spot is typically 20-100 candidates. Too few (10): you might miss relevant documents that the bi-encoder ranked 11th. Too many (500): re-ranking latency becomes prohibitive (5x slower) with diminishing returns — the 200th candidate is rarely relevant. Start with top-50, measure recall, and adjust. If recall@50 is already 95%+, you can reduce to top-20 for speed.

### Q3: "Can you re-rank without a dedicated re-ranking model?"
**Answer:** Yes — you can use an LLM as a re-ranker. Send the query + candidate documents to GPT-4 and ask "Rank these documents by relevance to the query." This is called LLM-based re-ranking. It's more expensive ($$$) and slower but can be more accurate because the LLM understands complex query intent. Best used when you don't have a fine-tuned cross-encoder and quality matters more than cost.

---

<p align="center"><a href="../README.md">← Back to all questions</a></p>
