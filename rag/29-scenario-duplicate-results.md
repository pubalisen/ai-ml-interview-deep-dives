# Scenario 4: Duplicate Results

> **The Scenario:** "Your RAG system keeps returning duplicate or near-duplicate results. How do you fix it?"

---

## Solutions

1. **Content-hash de-duplication at ingestion** — Before indexing, hash each chunk's content. Skip chunks whose hash already exists in the index.
2. **MMR (Maximal Marginal Relevance)** — Re-rank results to balance relevance and diversity, penalizing chunks too similar to already-selected ones.
3. **Source-level de-duplication** — If the same document exists in multiple source systems (Confluence AND Google Drive AND email), de-duplicate at the document level before chunking.
4. **Embedding-space de-duplication** — After retrieval, compute pairwise cosine similarity. If two results have similarity > 0.95, keep only the higher-scored one.
5. **Metadata-based grouping** — Group retrieved results by source document. If multiple chunks from the same document are retrieved, merge them or keep only the most relevant.
6. **Canonical document tracking** — Maintain a mapping of document versions and sources. Mark one version as canonical and suppress duplicates.

```python
def deduplicate_results(results, similarity_threshold=0.92):
    unique_results = []
    for result in results:
        is_duplicate = False
        for unique in unique_results:
            if cosine_similarity(result.embedding, unique.embedding) > similarity_threshold:
                is_duplicate = True
                break
        if not is_duplicate:
            unique_results.append(result)
    return unique_results
```

## Scenario Examples
### A: A company's knowledge base indexes both Confluence and Notion, which contain mirrored copies of the same policies. A query about PTO policy returns 5 results — 3 from Confluence and 2 from Notion, all nearly identical. Fix: (1) designate Confluence as the canonical source, (2) add a content-hash check during ingestion that skips documents already indexed from another source, (3) add source metadata to enable per-source de-duplication at query time.
### B: A documentation site has versioned pages (v1, v2, v3 of the same guide). All three versions are indexed. A query returns chunks from all three versions, confusing the user with outdated info mixed with current. Fix: add a `version` and `is_current` metadata field. Filter retrieval to `is_current=true`. Archive old versions but keep them queryable via explicit "show history" filter.

## Follow-Up Questions
### Q1: "Should de-duplication happen at ingestion time or query time?"
**Answer:** Both. Ingestion-time de-duplication prevents the same content from being indexed twice (saves storage, improves index quality). Query-time de-duplication handles edge cases where slightly different content is semantically identical. Ingestion-time is cheaper (runs once), query-time is safer (catches duplicates the ingestion step missed). Production systems use both layers.

### Q2: "How do you handle documents that are similar but not identical?"
**Answer:** This is the hardest case — e.g., a policy document updated with minor wording changes. Content hashes won't catch these. Solutions: (1) MinHash/LSH (Locality-Sensitive Hashing) to detect near-duplicates at ingestion. (2) Embedding similarity with a threshold (>0.95 = duplicate, 0.85-0.95 = similar, <0.85 = different). (3) LLM comparison: "Are these two chunks conveying the same information?" Most expensive but most accurate.

### Q3: "How does de-duplication interact with re-ranking?"
**Answer:** Apply de-duplication AFTER re-ranking. Re-ranking refines relevance scores — you want to keep the highest-quality version of a duplicate, which the re-ranker identifies. If you de-duplicate before re-ranking, you might keep a lower-quality version because the re-ranker hadn't scored them yet. Pipeline: retrieve top-50 → re-rank → de-duplicate → take top-5.

---

<p align="center"><a href="../README.md">← Back to all questions</a></p>
