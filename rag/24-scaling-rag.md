# Part 24: Scaling RAG to Millions of Documents

> **The Question:** "How do you scale a RAG system to millions of documents?"

---

## The Technical Breakdown

### The Scaling Challenges

| Challenge | At 10K docs | At 1M docs | At 100M docs |
|-----------|------------|------------|--------------|
| Index size | ~500 MB | ~50 GB | ~5 TB |
| Indexing time | Minutes | Hours | Days |
| Search latency | <10ms | 20-50ms | 50-200ms |
| Re-indexing cost | $5 | $500 | $50,000 |
| Memory (in-RAM) | Fits laptop | Needs server | Needs cluster |

### Scaling Strategies

#### 1. Approximate Nearest Neighbor (ANN) Indexes
Exact search is O(N). ANN reduces to O(log N) with 95-99% recall:

```
Algorithms:
  HNSW (Hierarchical Navigable Small World) — Best quality, more memory
  IVF (Inverted File Index) — Good balance of speed/quality
  PQ (Product Quantization) — Smallest memory footprint

HNSW at 100M vectors:
  Recall@10: 98.5%
  Latency: ~5ms
  Memory: ~100GB (for 768-dim vectors)
```

#### 2. Vector Quantization
Reduce vector size to save memory:

```
Full precision:  768 dims × 4 bytes = 3,072 bytes per vector
Scalar quantization: 768 dims × 1 byte = 768 bytes (4x reduction)
Product quantization: 768 dims → 96 bytes (32x reduction)

100M vectors:
  Full:  ~300 GB
  SQ:    ~75 GB
  PQ:    ~10 GB  (fits in RAM on a single server!)
```

#### 3. Sharding and Distribution
Split the index across multiple servers:

```
Shard Strategy:
  By document type:  Shard 1 (policies), Shard 2 (technical), Shard 3 (FAQ)
  By tenant:         Shard 1 (Customer A-M), Shard 2 (Customer N-Z)
  By time:           Shard 1 (2024), Shard 2 (2023), Shard 3 (archive)

Query routing: Send query to relevant shards only.
```

#### 4. Tiered Storage
Hot data in memory, cold data on disk:

```
Tier 1 (Hot):  Frequently accessed docs → In-memory index (~fast)
Tier 2 (Warm): Recent docs → SSD-backed index (~medium)
Tier 3 (Cold): Archive docs → Disk-backed index (~slow)

Route queries: Check hot tier first. If confidence low, expand to warm/cold.
```

#### 5. Incremental Indexing
Avoid full re-index by processing only changes:

```python
# Track document hashes
for doc in changed_documents_since(last_index_time):
    old_chunks = get_chunks_for_doc(doc.id)
    new_chunks = chunk_and_embed(doc)
    
    # Upsert changed chunks
    vector_db.delete(ids=[c.id for c in old_chunks])
    vector_db.upsert(new_chunks)
```

#### 6. Pre-Filtering with Metadata
Reduce search space before vector comparison:

```
100M total vectors
  → Filter: department="engineering" → 5M vectors
  → Filter: year=2024 → 500K vectors
  → Vector search on 500K (not 100M) → Fast!
```

### Managed vs Self-Hosted at Scale

| Approach | Best For | Examples |
|----------|----------|---------|
| Managed cloud | <50M vectors, low ops | Pinecone, Weaviate Cloud |
| Self-hosted | >50M vectors, cost control | Qdrant, Milvus, Elasticsearch |
| Hybrid | Variable load | Managed hot tier + self-hosted cold |

---

## Accuracy Check
| Claim | Verified? |
|-------|-----------|
| HNSW provides 95-99% recall with O(log N) search time | ✅ Malkov & Yashunin (2018), HNSW paper, ann-benchmarks.com |
| Product quantization reduces memory by 32x with minimal recall loss | ✅ Jégou et al. (2011), PQ paper; FAISS documentation |
| Sharding enables horizontal scaling across servers | ✅ Standard distributed systems pattern, Milvus/Qdrant docs |

## Scenario Examples
### A: An enterprise knowledge platform indexes 50M internal documents (emails, docs, wikis, tickets). Using Qdrant self-hosted on a 3-node cluster: each node handles ~17M vectors with HNSW + scalar quantization. Total memory: 150 GB across the cluster. Search latency: p95 < 30ms. Cost: $3K/month for 3 servers vs $15K/month for a managed service at this scale.
### B: A legal discovery platform needs to search 500M court documents. Strategy: PQ-compressed vectors in a 10-node Milvus cluster. Pre-filter by jurisdiction and date range (reduces search space by 95%). Tiered storage: last 5 years in memory, older on SSD. Full-text keyword search (Elasticsearch) runs in parallel for case numbers and statute references. End-to-end search latency: <200ms.

## Follow-Up Questions
### Q1: "At what scale does self-hosting become cheaper than managed services?"
**Answer:** Typically at 5-10M vectors. Below that, managed services (Pinecone at ~$70/month for 1M vectors) are cheaper than running your own server ($200+/month). Above 10M, self-hosted becomes significantly cheaper: 50M vectors on Pinecone ≈ $3,500/month; self-hosted Qdrant on 3 servers ≈ $900/month. The break-even depends on your team's ops capability — self-hosting requires DevOps expertise for backups, monitoring, and scaling.

### Q2: "How do you handle re-indexing 100M documents when the embedding model changes?"
**Answer:** You can't avoid it — all 100M documents must be re-embedded. Strategy: (1) Run the new model in parallel, building a shadow index. (2) Use batch embedding with GPU clusters (A100 can embed ~10K docs/min). (3) Blue-green deployment: switch traffic from old index to new index atomically. Timeline: 100M docs at 10K/min ≈ 7 days on one GPU, or ~17 hours on 10 GPUs. Plan for this cost when choosing an embedding model.

### Q3: "How does retrieval quality change at scale?"
**Answer:** It can degrade if not managed. At 100M vectors, the chance of a "false positive" (irrelevant chunk with high similarity score) increases. Mitigations: (1) Use metadata filtering to narrow the search space. (2) Increase top-k and add re-ranking (retrieve 100, re-rank to 5). (3) Use higher-dimensional embeddings (3072-dim captures more nuance than 384-dim at scale). (4) Regular evaluation — test retrieval recall weekly to catch degradation.

---

<p align="center"><a href="../README.md">← Back to all questions</a></p>
