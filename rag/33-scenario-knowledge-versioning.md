# Scenario 8: Knowledge Base Versioning

> **The Scenario:** "Your knowledge base evolves over time. How do you handle versioning and updates without breaking existing functionality?"

---

## Solutions

1. **Blue-green indexing** — Build a new index alongside the old one. Test the new index, then switch traffic atomically. If issues arise, roll back instantly to the old index.
2. **Document versioning with metadata** — Track version numbers in chunk metadata. Default queries to the latest version; allow explicit version queries for audit purposes.
3. **Change detection pipeline** — Compare new document versions against indexed versions using content hashing. Only re-index changed documents.
4. **Snapshot-based rollback** — Take snapshots of the vector index before each update. If the new version degrades quality, restore from snapshot.
5. **A/B testing infrastructure** — Route 10% of traffic to the new index and 90% to the old. Compare quality metrics before full rollout.
6. **Changelog tracking** — Maintain a log of what changed, when, and why. This is essential for debugging when answer quality suddenly shifts.

```python
# Blue-green deployment for vector index
def deploy_new_index(new_docs):
    # Build new index
    new_collection = vector_db.create_collection("docs_v2")
    for doc in new_docs:
        chunks = chunk_and_embed(doc)
        new_collection.upsert(chunks)
    
    # Run evaluation suite against new index
    eval_results = evaluate_rag(new_collection, test_queries)
    
    if eval_results.all_metrics_pass():
        # Atomic switch
        vector_db.alias("docs_live", "docs_v2")   # Point live to new
        vector_db.delete_collection("docs_v1")      # Clean up old
    else:
        # Rollback
        vector_db.delete_collection("docs_v2")
        alert("New index failed evaluation", eval_results)
```

## Scenario Examples
### A: A regulatory compliance system updates quarterly when new regulations are published. The team uses blue-green indexing: new regulations are ingested into a shadow index, tested against 200 standard compliance questions, and only promoted to production when all answers match expected outputs. If a quarterly update causes 5 questions to fail, the shadow index is held back while the team investigates (usually a chunking issue with new document formatting).
### B: A product documentation team pushes updates 3 times per week. They use incremental updates with version metadata. Each doc has `version` and `is_current`. When a doc is updated: the old version is marked `is_current=false`, the new version is indexed with `is_current=true`. Customer-facing queries filter by `is_current=true`. Internal quality team can query any version for regression testing.

## Follow-Up Questions
### Q1: "How do you handle if the new index is worse than the old one?"
**Answer:** That's why evaluation before deployment is critical. Run your evaluation suite (50-200 known queries with expected answers) against the new index BEFORE switching traffic. If metrics drop by more than a threshold (e.g., faithfulness drops >5%), block the deployment. For gradual degradation that evaluation doesn't catch: monitor production metrics (user feedback, answer quality scores) and auto-rollback if they degrade within 24 hours of deployment.

### Q2: "How do you manage embedding model changes alongside document updates?"
**Answer:** They're separate concerns but both require re-indexing. Document updates: re-embed only changed documents. Embedding model change: re-embed ALL documents (new model produces different vectors). Never mix vectors from different embedding models in the same index. Best practice: version your indexes by embedding model (`docs_bge_v1.5`, `docs_openai_3_small`) so you can maintain multiple indexes during migration.

### Q3: "How do you provide 'as-of-date' queries for compliance?"
**Answer:** Maintain a time-series of index snapshots or versioned chunks with timestamps. Query pattern: `vector_search(query, filter={"valid_at": {"$lte": "2024-06-15"}})` returns the knowledge state as of June 15, 2024. This requires never deleting old chunks — only marking them as superseded. Storage grows linearly with updates, so implement a retention policy (keep last 2 years of versions, archive older ones).

---

<p align="center"><a href="../README.md">← Back to all questions</a></p>
