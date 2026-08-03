# Part 20: Metadata Filtering

> **The Question:** "What is the role of metadata filtering in RAG systems?"

---

## The Technical Breakdown

### What Is Metadata Filtering?

Metadata filtering narrows vector search to a subset of documents **before** calculating similarity. Instead of searching your entire knowledge base, you search only the chunks that match specific criteria.

```
Without metadata filtering:
  Query → Search ALL 1M chunks → Top 5

With metadata filtering:
  Query → Filter to department="Engineering" AND year>=2024 → Search 50K chunks → Top 5
```

### Common Metadata Fields

```json
{
  "id": "chunk_042",
  "vector": [0.023, -0.841, ...],
  "text": "Our refund policy allows...",
  "metadata": {
    "source": "policies/refund-policy.pdf",
    "page": 3,
    "department": "customer-service",
    "last_updated": "2024-11-15",
    "document_type": "policy",
    "language": "en",
    "access_level": "internal",
    "author": "legal-team",
    "version": "2.1",
    "tags": ["refund", "returns", "policy"]
  }
}
```

### Why Metadata Filtering Matters

#### 1. Precision Improvement
```
Query: "What's the engineering team's OKR process?"
Without filter: Returns HR OKRs, Sales OKRs, general OKR articles
With filter (department="engineering"): Returns only engineering-specific OKR docs
```

#### 2. Access Control
```python
results = vector_db.search(
    query_embedding=embed(query),
    filter={
        "access_level": {"$in": user.access_levels},
        "department": {"$in": user.departments}
    }
)
```

#### 3. Temporal Relevance
```python
# Only search documents from the last year
results = vector_db.search(
    query_embedding=embed(query),
    filter={"last_updated": {"$gte": "2024-01-01"}}
)
```

#### 4. Source Selection
```python
# Route financial questions to financial docs only
if query_type == "financial":
    filter = {"document_type": {"$in": ["financial_report", "earnings", "forecast"]}}
```

### Filter Operators

| Operator | Example | Meaning |
|----------|---------|---------|
| `$eq` | `{"status": "active"}` | Equals |
| `$ne` | `{"status": {"$ne": "archived"}}` | Not equals |
| `$in` | `{"dept": {"$in": ["eng", "product"]}}` | In list |
| `$gte` / `$lte` | `{"year": {"$gte": 2024}}` | Greater/less than |
| `$and` | Combine multiple conditions | All conditions must match |
| `$or` | Any of the conditions | At least one must match |

### Pre-Filtering vs Post-Filtering

```
Pre-filtering:  Filter FIRST, then vector search on filtered subset
  ✅ Fast — searches smaller set
  ✅ Results always match filters
  ❌ May miss relevant results if filter is too restrictive

Post-filtering: Vector search ALL docs, then filter results
  ✅ Finds all semantically relevant docs
  ❌ Slow — searches entire corpus
  ❌ May return fewer than top-k after filtering
```

Most vector databases use pre-filtering with optimized indexes.

---

## Accuracy Check
| Claim | Verified? |
|-------|-----------|
| Metadata filtering improves retrieval precision by narrowing search scope | ✅ Standard feature in Pinecone, Weaviate, Qdrant, Milvus |
| Pre-filtering is faster but may reduce recall | ✅ Documented trade-off in vector DB documentation |
| Metadata enables access control in multi-tenant RAG systems | ✅ Production pattern for enterprise RAG systems |

## Scenario Examples
### A: A multi-tenant SaaS platform where each customer has their own knowledge base. Without metadata filtering, Customer A's queries might retrieve Customer B's confidential documents (both are semantically similar — they're in the same industry). Metadata filter: `tenant_id = "customer_a"` ensures strict data isolation. This is non-negotiable for enterprise deployments.
### B: A legal research tool where lawyers search by jurisdiction. A query about "employment termination laws" returns very different results for California vs Texas. Metadata filter: `jurisdiction = "california"` ensures only California-specific statutes and case law are retrieved. Without filtering, the top results mix jurisdictions, leading to incorrect legal advice.

## Follow-Up Questions
### Q1: "How do you decide what metadata to capture?"
**Answer:** Start with five essential fields: (1) **source** — where the document came from, (2) **date** — when it was created/updated, (3) **type** — what kind of document (policy, FAQ, report), (4) **access level** — who can see it, (5) **section/topic** — broad category. Add domain-specific fields as needed (jurisdiction for legal, department for HR, product line for support). Don't over-index — each metadata field adds storage and query overhead.

### Q2: "How do you extract metadata automatically?"
**Answer:** Three approaches: (1) **From source** — file name, creation date, directory structure, API metadata (Confluence labels, Notion properties). (2) **From content** — use an LLM to extract metadata: "Given this document, extract: topic, department, document type." Costs ~$0.001 per document. (3) **Hybrid** — source metadata for structured fields (date, author), LLM for unstructured (topic, summary). Automate in your ingestion pipeline.

### Q3: "Can over-filtering hurt RAG performance?"
**Answer:** Yes — over-filtering reduces the search space too aggressively and causes recall to drop. Example: filtering by `department="engineering" AND year=2024 AND type="RFC"` might leave only 20 documents. If the answer isn't in those 20, the system fails. Solutions: (1) start with broad filters, narrow only if needed, (2) fall back to unfiltered search if filtered results have low similarity scores, (3) use soft filters (boost matching docs) instead of hard filters (exclude non-matching docs).

---

<p align="center"><a href="../README.md">← Back to all questions</a></p>
