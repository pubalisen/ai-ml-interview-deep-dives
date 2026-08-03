# Scenario 2: Chunk Overlap Redundancy

> **The Scenario:** "Your RAG chunk overlap causes redundant results. How do you reduce redundancy?"

---

## Solutions

1. **Reduce overlap percentage** — If using 50% overlap, reduce to 10-20%. Some overlap is needed to prevent boundary information loss, but excessive overlap duplicates content.
2. **De-duplicate retrieved results** — After retrieval, check for near-duplicate chunks (cosine similarity > 0.95 between chunks) and keep only the highest-scoring one.
3. **Use MMR (Maximal Marginal Relevance)** — MMR balances relevance with diversity: it penalizes results that are too similar to already-selected results.
4. **Switch to semantic chunking** — Semantic chunks don't use sliding windows, so they have zero artificial overlap.
5. **Parent-child chunking** — De-duplicate at the parent level. If multiple child chunks from the same parent are retrieved, only include the parent once.
6. **Hash-based de-duplication** — Store content hashes with each chunk and filter duplicates post-retrieval.

```python
# MMR implementation
def mmr_rerank(query_embedding, candidates, lambda_param=0.5, top_k=5):
    selected = []
    remaining = list(candidates)
    
    for _ in range(top_k):
        best = max(remaining, key=lambda c:
            lambda_param * similarity(query_embedding, c.embedding) -
            (1 - lambda_param) * max(similarity(c.embedding, s.embedding) for s in selected) if selected else 0
        )
        selected.append(best)
        remaining.remove(best)
    
    return selected
```

## Scenario Examples
### A: A knowledge base with 50% chunk overlap retrieves 5 chunks for "refund policy" — but 3 of them contain 80% of the same text (the overlap region). The LLM sees the same sentences repeated 3 times, wasting context window space. Fix: reduce overlap to 15%, apply MMR with λ=0.6, and de-duplicate chunks with >90% content overlap.
### B: A product documentation system retrieves 10 chunks but 6 are from the same 3-page section with different overlapping windows. The LLM generates a repetitive answer. Fix: switch to parent-child chunking — retrieve at sentence level, expand to section level, de-duplicate at the section level. Now the LLM sees 3 unique sections instead of 10 overlapping fragments.

## Follow-Up Questions
### Q1: "What is MMR and why does it help with diversity?"
**Answer:** Maximal Marginal Relevance (MMR) selects results that are both relevant to the query AND diverse from each other. The formula: score = λ × relevance(query, doc) - (1-λ) × max_similarity(doc, already_selected). When λ=1, it's pure relevance (may be redundant). When λ=0, it's pure diversity (may be irrelevant). λ=0.5-0.7 is typical for RAG — you get the most relevant chunks that also cover different aspects of the query.

### Q2: "Is zero overlap ever acceptable?"
**Answer:** Yes, when using semantic or document-structure-aware chunking. These methods split at natural boundaries (topic changes, section headers), so there's no risk of cutting important information at arbitrary positions. Zero overlap is also fine when chunks are complete semantic units (one FAQ entry per chunk, one API endpoint per chunk). The only time overlap is essential is with fixed-size chunking, where arbitrary cut points can split sentences.

### Q3: "How do you measure redundancy in your retrieved results?"
**Answer:** Calculate the average pairwise cosine similarity between all retrieved chunks. If avg_pairwise_similarity > 0.85, you have a redundancy problem. Another metric: unique information coverage — after de-duplicating at the sentence level, what percentage of unique sentences remain? If 10 chunks contain only 60% unique content, 40% is redundant. Target: >85% unique content across retrieved chunks.

---

<p align="center"><a href="../README.md">← Back to all questions</a></p>
