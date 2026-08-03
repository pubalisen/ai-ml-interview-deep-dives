# Part 19: RAG Latency Optimization

> **The Question:** "How do you optimize RAG for latency in production?"

---

## The Technical Breakdown

### Where Latency Lives

A typical RAG request has four latency components:

```
Total RAG Latency = Embedding + Retrieval + Prompt Assembly + LLM Generation
                    (~50ms)     (~20-100ms)  (~5ms)          (~500-3000ms)
                    
Typical total: 600-3500ms per request
```

The LLM generation dominates, but retrieval latency compounds when you add re-ranking, multiple sources, or multi-hop retrieval.

### Optimization Strategies

#### 1. Embedding Caching
Cache query embeddings for repeated or similar queries:
```python
cache_key = hash(query_text)
if cache_key in embedding_cache:
    query_vector = embedding_cache[cache_key]
else:
    query_vector = embed(query_text)
    embedding_cache[cache_key] = query_vector
```

#### 2. Semantic Caching
Cache entire RAG responses for semantically similar queries:
```python
# Check if a similar query was asked before
similar = semantic_cache.search(query_embedding, threshold=0.95)
if similar:
    return similar.cached_response  # Skip retrieval + generation entirely

# Cache miss — run full pipeline
response = rag_pipeline(query)
semantic_cache.store(query_embedding, response)
```

#### 3. Streaming Generation
Don't wait for the full response — stream tokens:
```
Without streaming: User waits 2500ms → sees full answer
With streaming:    User sees first token at ~300ms → rest streams in
```

#### 4. Parallel Retrieval
Search multiple sources simultaneously:
```python
import asyncio

async def parallel_retrieve(query):
    vector_results, bm25_results, sql_results = await asyncio.gather(
        vector_search(query),
        bm25_search(query),
        sql_query(query),
    )
    return fuse(vector_results, bm25_results, sql_results)
```

#### 5. Pre-computed Responses
For common queries, pre-generate and cache responses:
```
Top 100 FAQ questions → Pre-generate RAG answers nightly
Query match: exact → return cached answer (0ms generation)
Query match: semantic > 0.95 → return cached answer
Query match: none → run full RAG pipeline
```

#### 6. Smaller, Faster Models
```
GPT-4o:      High quality, ~2000ms TTFT
GPT-4o-mini: Good quality, ~400ms TTFT
Local Llama: Good quality, ~200ms TTFT (on GPU)
```

#### 7. Reduce Context Size
Fewer retrieved chunks = shorter prompt = faster generation:
```
10 chunks × 500 tokens = 5000 token context → ~2000ms generation
3 chunks × 500 tokens  = 1500 token context → ~800ms generation
```

### Latency Budget Example

| Component | Unoptimized | Optimized | Technique |
|-----------|-------------|-----------|-----------|
| Embedding | 80ms | 5ms | Cache hit |
| Retrieval | 120ms | 50ms | ANN index tuning |
| Re-ranking | 200ms | 0ms | Skip for simple queries |
| Prompt assembly | 10ms | 10ms | — |
| LLM generation | 2500ms | 800ms | Smaller model + less context |
| **Total** | **2910ms** | **865ms** | **3.4x faster** |

---

## Accuracy Check
| Claim | Verified? |
|-------|-----------|
| LLM generation dominates RAG latency | ✅ Consistent across production benchmarks |
| Semantic caching can eliminate retrieval + generation for similar queries | ✅ GPTCache, Redis semantic caching implementations |
| Streaming reduces perceived latency by showing first token quickly | ✅ Standard practice, OpenAI/Anthropic streaming APIs |

## Scenario Examples
### A: A customer support chatbot with a 1-second SLA. Current latency: 2.8 seconds. Optimizations: (1) Switch from GPT-4o to GPT-4o-mini (saves 1200ms), (2) reduce from 10 chunks to 3 with re-ranking (saves 400ms), (3) add semantic caching for top 50 FAQ questions (0ms for 30% of queries). New average latency: 0.7 seconds. SLA met.
### B: An internal knowledge search tool where users type queries and expect near-instant results. Implement speculative retrieval: as the user types, start embedding and retrieving after 300ms of pause (debounce). By the time they hit Enter, retrieval is already done. Only generation remains. Perceived latency drops from 2s to 800ms.

## Follow-Up Questions
### Q1: "How does semantic caching work without returning stale answers?"
**Answer:** Set a TTL (time-to-live) on cached responses — e.g., 24 hours for support FAQs, 1 hour for pricing data. When the underlying documents change, invalidate affected cache entries. The cache key is the query embedding, but you also hash the retrieved document IDs — if retrieval would return different documents, the cache is invalid. This prevents stale answers while maintaining the speed benefit.

### Q2: "When is it worth using a local/self-hosted model for latency?"
**Answer:** When your query volume exceeds ~10K requests/day AND you have GPU infrastructure. A Llama 3.1 8B on an A10G processes queries at ~100 tokens/second with ~50ms TTFT vs GPT-4o-mini's ~200ms TTFT over the network. The 150ms saving per request adds up. Also essential when you need guaranteed latency — API calls have variable latency depending on provider load.

### Q3: "How do you measure and monitor RAG latency in production?"
**Answer:** Instrument every component with timing: (1) p50, p95, p99 latency for embedding, retrieval, re-ranking, generation. (2) Track token counts (longer prompts = longer generation). (3) Monitor cache hit rates (higher = faster average latency). (4) Set SLO alerts: page if p95 exceeds 3 seconds. Tools: OpenTelemetry tracing with custom spans for each RAG step, dashboarded in Grafana or Datadog.

---

<p align="center"><a href="../README.md">← Back to all questions</a></p>
