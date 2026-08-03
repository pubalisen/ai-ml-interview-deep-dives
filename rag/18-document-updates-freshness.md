# Part 18: Document Updates and Freshness

> **The Question:** "How do you handle document updates and maintain freshness in a RAG system?"

---

## The Technical Breakdown

### The Freshness Problem

RAG knowledge bases are not static. Documents change: policies get updated, product prices change, employees join and leave, research findings evolve. If your index contains stale embeddings, the LLM generates confidently wrong answers.

```
January: Index contains "Refund window: 30 days"
March:   Policy updated to "Refund window: 14 days"
April:   User asks about refund policy → RAG returns "30 days" (WRONG)
```

### Freshness Strategies

#### 1. Full Re-Indexing (Simplest)
Periodically re-process all documents:

```
Schedule: Every night at 2 AM
  1. Fetch all documents from sources
  2. Re-chunk and re-embed everything
  3. Replace the vector index

Pros: Simple, guaranteed consistency
Cons: Expensive for large corpora, unnecessary for unchanged docs
Cost: 10M chunks × $0.02/1M tokens = ~$100 per full re-index
```

#### 2. Incremental Updates (Efficient)
Only re-process changed documents:

```python
def incremental_update():
    for doc in get_all_documents():
        current_hash = hash(doc.content)
        stored_hash = get_stored_hash(doc.id)
        
        if current_hash != stored_hash:
            # Document changed — re-chunk and re-embed
            delete_chunks(doc.id)
            new_chunks = chunk(doc.content)
            embeddings = embed(new_chunks)
            upsert_to_vector_db(doc.id, new_chunks, embeddings)
            update_hash(doc.id, current_hash)
```

#### 3. Metadata-Based Freshness Filtering
Store timestamps with each chunk and filter at query time:

```python
results = vector_db.search(
    query_embedding=embed(query),
    top_k=10,
    filter={
        "last_updated": {"$gte": "2024-01-01"},  # Only recent docs
        "status": "active"                         # Skip archived docs
    }
)
```

#### 4. Version-Aware Retrieval
Keep multiple versions, retrieve the latest:

```
Chunk: "Refund policy v1" (created: Jan 2024, superseded: Mar 2024)
Chunk: "Refund policy v2" (created: Mar 2024, current: true)

Query filter: version = "current" → Only retrieves v2
```

#### 5. Source-Driven Webhooks
Trigger re-indexing when source documents change:

```
Confluence page updated → Webhook → Re-index that page
Google Doc modified → Change notification → Re-embed affected chunks
GitHub PR merged → CI/CD → Re-index documentation
```

### The Architecture for Freshness

```
┌────────────────┐     ┌──────────────┐     ┌──────────────┐
│  Source Systems │ ──→ │  Change      │ ──→ │  Indexing     │
│  (Confluence,   │     │  Detector    │     │  Pipeline     │
│   Notion, S3)   │     │  (webhooks,  │     │  (chunk,      │
└────────────────┘     │   polling,   │     │   embed,      │
                        │   hashes)    │     │   upsert)     │
                        └──────────────┘     └──────┬───────┘
                                                    │
                                                    ▼
                                             ┌──────────────┐
                                             │  Vector DB    │
                                             │  (latest      │
                                             │   embeddings) │
                                             └──────────────┘
```

### Conflict Resolution

When old and new versions of a document both exist in the index:

| Strategy | Approach |
|----------|----------|
| **Replace** | Delete old chunks, insert new ones |
| **Versioned** | Keep both, filter by version at query time |
| **Soft delete** | Mark old as inactive, filter out at query time |
| **TTL** | Auto-expire chunks after N days |

---

## Accuracy Check
| Claim | Verified? |
|-------|-----------|
| Stale indexes cause RAG to return outdated answers | ✅ Common production issue documented across RAG implementations |
| Incremental updates using content hashing are more efficient than full re-indexing | ✅ Standard engineering practice, LlamaIndex ingestion pipeline docs |
| Metadata filtering enables freshness control at query time | ✅ Supported by Pinecone, Weaviate, Qdrant metadata filtering |

## Scenario Examples
### A: An HR knowledge base has 500 policy documents. 5-10 change per week. Full nightly re-indexing costs $100/night ($36K/year) and is wasteful. Incremental updates: hash each document, detect the 5-10 changes, re-index only those. Cost drops to ~$1/day ($365/year). A change detection service polls Confluence hourly, triggers re-indexing for modified pages within 15 minutes of the change.
### B: A financial services company needs audit trails for regulatory compliance. They use version-aware retrieval: every document update creates a new version in the index. Queries default to the latest version, but compliance officers can query "as of date X" to see what the system would have answered on any past date. This satisfies SEC audit requirements for AI-generated advice.

## Follow-Up Questions
### Q1: "What if a document is deleted — how do you handle removals?"
**Answer:** Three approaches: (1) **Hard delete** — remove all chunks for that document from the vector DB immediately. Risk: if the deletion was accidental, data is lost. (2) **Soft delete** — mark chunks as deleted, filter them out at query time. Recoverable but increases index size. (3) **TTL-based** — chunks auto-expire after a configured period. The change detector must track deletions, not just modifications.

### Q2: "How do you handle documents that update very frequently (real-time data)?"
**Answer:** Don't embed real-time data — query it directly. For stock prices, live metrics, or dashboards: use a tool-calling approach where the LLM calls an API to get the current value. For semi-frequent updates (daily): use a queue-based system that re-indexes changed documents within minutes. The general rule: if data changes faster than your re-indexing cadence, use direct API queries instead of RAG.

### Q3: "How do you know if your freshness strategy is working?"
**Answer:** Monitor two metrics: (1) **Index lag** — the time between a source document changing and the index reflecting the change. Target: <1 hour for critical docs, <24 hours for general docs. (2) **Freshness hit rate** — what percentage of user queries are answered from content updated in the last N days? If most answers come from months-old content, your freshness pipeline may be broken. Set alerts for index lag exceeding thresholds.

---

<p align="center"><a href="../README.md">← Back to all questions</a></p>
