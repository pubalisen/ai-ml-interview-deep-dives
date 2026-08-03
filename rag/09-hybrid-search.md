# Part 9: Hybrid Search

> **The Question:** "What is hybrid search, and why is it better than pure vector search?"

---

## The Technical Breakdown

### The Problem with Vector-Only Search

Vector (dense) search excels at semantic similarity but fails on:
- **Exact keyword matches** — Searching "error code E-4021" with vector search might return general error handling docs instead of the specific error.
- **Named entities** — "Dr. Sarah Mitchell's publications" might return any doctor's publications because the embedding captures "doctor + publications" semantically.
- **Acronyms and IDs** — "HIPAA compliance" embedded semantically might miss documents that only use "Health Insurance Portability and Accountability Act."

### The Problem with Keyword-Only Search

Keyword (sparse) search excels at exact matching but fails on:
- **Semantic understanding** — Searching "car problems" won't find documents about "vehicle issues" or "automobile defects."
- **Paraphrased content** — "How to terminate employment" won't match docs about "firing an employee" or "letting someone go."
- **Intent understanding** — "Best way to store leftovers" won't match a doc titled "Food preservation techniques."

### Hybrid Search: Best of Both Worlds

```
User Query: "HIPAA compliance requirements for cloud storage"

Dense Search (Vector):                    Sparse Search (BM25):
┌─────────────────────────┐              ┌─────────────────────────┐
│ 1. Cloud security guide  │ score: 0.89 │ 1. HIPAA compliance doc  │ score: 12.4
│ 2. Data privacy overview │ score: 0.85 │ 2. Cloud storage policy  │ score: 11.2
│ 3. GDPR requirements     │ score: 0.82 │ 3. HIPAA audit checklist │ score: 10.8
│ 4. AWS storage best...   │ score: 0.81 │ 4. Storage encryption    │ score: 9.1
│ 5. Healthcare data...    │ score: 0.79 │ 5. Compliance framework  │ score: 8.3
└─────────────────────────┘              └─────────────────────────┘

        Hybrid Fusion (RRF):
        ┌─────────────────────────┐
        │ 1. HIPAA compliance doc  │ ← Ranked high by BOTH
        │ 2. Cloud security guide  │ ← Strong vector match
        │ 3. Cloud storage policy  │ ← Strong keyword match
        │ 4. Healthcare data...    │ ← Good semantic relevance
        │ 5. HIPAA audit checklist │ ← Exact keyword match
        └─────────────────────────┘
```

### Reciprocal Rank Fusion (RRF)

The most common method to combine dense and sparse scores:

```python
def reciprocal_rank_fusion(dense_results, sparse_results, k=60):
    """
    RRF score = Σ 1 / (k + rank_i)
    
    k=60 is the standard constant that prevents high-ranked
    items from dominating too heavily.
    """
    fused_scores = {}
    
    for rank, doc in enumerate(dense_results):
        fused_scores[doc.id] = fused_scores.get(doc.id, 0) + 1 / (k + rank + 1)
    
    for rank, doc in enumerate(sparse_results):
        fused_scores[doc.id] = fused_scores.get(doc.id, 0) + 1 / (k + rank + 1)
    
    return sorted(fused_scores.items(), key=lambda x: x[1], reverse=True)
```

### Alternative Fusion Methods

| Method | How It Works | When to Use |
|--------|-------------|-------------|
| **RRF** | Combine ranks, ignore scores | Default — robust and simple |
| **Weighted sum** | `α × dense_score + (1-α) × sparse_score` | When you can normalize scores |
| **Convex combination** | Tune α per query type | Advanced — requires query classification |
| **Re-ranking** | Retrieve with both, re-rank combined set with cross-encoder | Highest quality, most expensive |

### Vector DBs with Native Hybrid Search

| Database | Hybrid Support |
|----------|---------------|
| **Weaviate** | Native BM25 + vector, auto-fusion |
| **Qdrant** | Sparse + dense vectors in same collection |
| **Pinecone** | Sparse-dense vectors via sparse_values |
| **Elasticsearch** | kNN + BM25 in single query |
| **Vespa** | Full hybrid with custom ranking |

---

## Accuracy Check
| Claim | Verified? |
|-------|-----------|
| Hybrid search outperforms dense-only and sparse-only on most retrieval benchmarks | ✅ BEIR benchmark results, Ma et al. (2023) |
| RRF with k=60 is the standard fusion method | ✅ Cormack et al. (2009), widely adopted in production |
| Dense search fails on exact keyword/entity matching | ✅ Known limitation documented across vector search providers |

## Scenario Examples
### A: A legal research platform where lawyers search case law by statute number ("Section 230") and by concept ("internet platform liability"). Pure vector search finds conceptual matches but misses the exact statute reference. Pure BM25 finds the statute but misses related case law that discusses the concept without mentioning the number. Hybrid search catches both — statute number via BM25, related concepts via dense retrieval. Lawyers report 40% better search results.
### B: An IT ticketing system where support engineers search for error codes ("ORA-12541") and symptoms ("database connection timeout after migration"). Hybrid search nails both: BM25 matches the exact Oracle error code, dense search finds tickets with similar symptoms described in different words. Resolution: time-to-find-related-ticket drops from 8 minutes to 45 seconds.

## Follow-Up Questions
### Q1: "How do you tune the balance between dense and sparse results?"
**Answer:** Start with equal weighting (RRF with default k=60). Then evaluate on your query test set: if exact-match queries are underperforming, increase sparse weight. If semantic/paraphrase queries are underperforming, increase dense weight. Some systems route queries: detect if a query contains specific identifiers (error codes, product IDs) and boost sparse weight; for natural language questions, boost dense weight.

### Q2: "Is hybrid search always better than dense-only?"
**Answer:** Almost always, but not literally always. For purely conversational queries ("explain how machine learning works"), dense search alone is sufficient — there are no keywords to match exactly. Hybrid search adds latency (running two search systems) and complexity. If your queries are exclusively semantic and never contain specific identifiers, entities, or codes, pure dense search is simpler and almost as good.

### Q3: "How do you implement hybrid search if your vector DB doesn't support it natively?"
**Answer:** Run two systems in parallel: (1) Elasticsearch/OpenSearch for BM25 keyword search, (2) Pinecone/FAISS for dense vector search. Query both, collect results, apply RRF in your application code, and return the fused top-k. This is the "federated search" pattern. It adds infrastructure complexity but gives you full control over fusion logic. Many production systems run this exact setup.

---

<p align="center"><a href="../README.md">← Back to all questions</a></p>
