# Scenario 3: Slow RAG Retrieval

> **The Scenario:** "Your RAG retrieval is too slow with a large knowledge base. How do you speed it up?"

---

## Solutions

1. **Optimize ANN index parameters** — Tune HNSW `ef_construction` and `M` parameters. Higher values = better recall but slower. Lower = faster but may miss results. Find the sweet spot for your recall requirements.
2. **Add metadata pre-filtering** — Filter by department, date, or document type BEFORE vector search to reduce the search space from millions to thousands.
3. **Use vector quantization** — Scalar quantization (float32 → int8) reduces memory 4x and speeds up distance calculations. Product quantization gives 32x compression.
4. **Implement semantic caching** — Cache results for similar queries. If a nearly identical query was asked recently, return the cached result without searching.
5. **Use smaller embeddings** — Switch from 3072-dim to 768-dim or use Matryoshka embeddings (truncate to 256-dim for initial search, full dims for re-ranking).
6. **Shard across nodes** — Distribute the index across multiple servers and query in parallel.
7. **Pre-compute for hot queries** — Identify the top 100-500 most common query patterns and pre-compute their results.

## Scenario Examples
### A: A customer support system searches 5M chunks and retrieval takes 800ms (target: <100ms). Analysis: using full 3072-dim OpenAI embeddings in Pinecone without metadata filtering. Fix: (1) add `document_type` filter (reduces search to 500K chunks), (2) switch to `text-embedding-3-small` at 1536 dims, (3) enable scalar quantization. New retrieval time: 45ms. 18x improvement.
### B: An enterprise search platform queries 50M vectors across all departments. Retrieval takes 2 seconds. Fix: (1) shard by department (10 shards of 5M each), (2) route queries to relevant shards only (most queries target 1-2 departments), (3) add semantic caching with 1-hour TTL (35% cache hit rate). Average retrieval: 120ms. For cached queries: <5ms.

## Follow-Up Questions
### Q1: "What's the trade-off between search speed and recall quality?"
**Answer:** The fundamental ANN trade-off: faster search = lower recall. At 1M vectors, exact search gives 100% recall in ~500ms. HNSW with aggressive parameters gives 99% recall in 5ms. With quantization: 95% recall in 2ms. For most RAG applications, 95% recall is acceptable — the 5% missed results are edge cases. Set a recall@10 target (e.g., 97%) and tune parameters to be as fast as possible while meeting it.

### Q2: "How do you diagnose what's causing the slowness?"
**Answer:** Profile each component: (1) **Embedding latency** — time to convert query to vector (API: 50-200ms, local: 10-50ms). (2) **Network latency** — time to reach the vector DB (cloud: 20-50ms, same-region: 5-10ms). (3) **Search latency** — time for ANN search (depends on index size, params, dims). (4) **Post-processing** — metadata retrieval, de-duplication, re-ranking. Instrument with timers. The bottleneck is usually search latency at scale, but embedding API calls can dominate at low scale.

### Q3: "When is it faster to use a local model vs an API?"
**Answer:** For embedding: local is faster above ~100 queries/second (no network roundtrip). For generation: depends on GPU. A10G running Llama 3.1 8B: ~50ms TTFT locally vs ~200ms for GPT-4o-mini API. But local requires GPU infrastructure. Rule of thumb: if you have GPUs AND high query volume, local is faster AND cheaper. Below 1K queries/day, API is simpler and the latency difference is negligible compared to generation time.

---

<p align="center"><a href="../README.md">← Back to all questions</a></p>
