# Scenario 11: Outdated Answers

> **The Scenario:** "Your RAG system gives outdated answers because the knowledge base evolves faster than your re-indexing cadence. How do you solve it?"

---

## Solutions

1. **Event-driven re-indexing** — Trigger re-indexing immediately when source documents change (webhooks from Confluence, S3 events, GitHub push notifications) instead of relying on scheduled batch re-indexing.
2. **TTL (Time-to-Live) on chunks** — Set expiration dates on chunks. Automatically exclude expired chunks from retrieval until they're refreshed.
3. **Freshness scoring** — Boost recently updated documents in ranking. A chunk updated yesterday should score higher than one from 6 months ago, all else being equal.
4. **Confidence disclaimers** — When the retrieved chunk's `last_updated` metadata is older than a threshold, prepend a disclaimer: "⚠️ This information was last verified on [date]. Please confirm it's still current."
5. **Real-time data for volatile information** — For data that changes daily (prices, stock levels, schedules), use API calls instead of RAG. Reserve RAG for semi-static knowledge.
6. **Human review alerts** — Flag answers sourced from old chunks for periodic human verification.

```python
def freshness_aware_retrieve(query, max_age_days=90):
    # Retrieve with recency boost
    results = vector_db.search(
        query_embedding=embed(query),
        top_k=10,
        filter={"last_updated": {"$gte": days_ago(max_age_days)}}
    )
    
    # If filtered results are insufficient, expand with warning
    if len(results) < 3:
        older_results = vector_db.search(
            query_embedding=embed(query),
            top_k=5
        )
        for r in older_results:
            r.metadata["freshness_warning"] = True
        results.extend(older_results)
    
    return results
```

## Scenario Examples
### A: A SaaS product updates pricing quarterly, but the RAG index re-processes weekly. In the first few days after a price change, the old pricing appears in the index. A customer asks "How much does the Pro plan cost?" and gets last quarter's price. Fix: pricing pages have a webhook that triggers instant re-indexing on change. Additionally, the system prompt includes "If discussing pricing, check that the source was updated within the last 7 days. If not, direct the user to the pricing page."
### B: A cybersecurity knowledge base tracks CVEs (Common Vulnerabilities). New CVEs are published daily, but re-indexing runs weekly. A security engineer searches for a CVE published yesterday and gets no results. Fix: (1) implement incremental indexing that runs hourly, (2) for the latest 24 hours, supplement RAG with a direct NVD API search that checks for CVEs in real-time.

## Follow-Up Questions
### Q1: "How do you balance freshness with stability?"
**Answer:** Not all content needs to be fresh. Categorize your knowledge: (1) **Volatile** (prices, availability, announcements): real-time or near-real-time updates. (2) **Semi-static** (policies, procedures, documentation): daily or event-driven updates. (3) **Static** (historical data, reference material): monthly or on-change updates. Apply different re-indexing cadences to each category. Over-indexing static content wastes resources; under-indexing volatile content causes wrong answers.

### Q2: "How do you detect that an answer is outdated if the system doesn't know the current truth?"
**Answer:** You can't detect it with certainty — the system doesn't know what it doesn't know. But you can flag risk: (1) if `last_updated` is older than a domain-specific threshold (30 days for policies, 1 day for pricing), add a staleness warning. (2) Monitor user feedback — a spike in negative ratings may indicate stale content. (3) Scheduled audits — sample 20 answers weekly and have domain experts verify they're still correct.

### Q3: "What about content that should never expire?"
**Answer:** Some knowledge is stable: company founding date, physical laws, historical facts. Tag these with `expiry: "never"` or `type: "reference"`. They don't need freshness warnings or frequent re-indexing. The challenge: determining what's truly stable vs what might change unexpectedly. Even "permanent" facts like company addresses or regulatory frameworks can change. Default to freshness-aware unless explicitly marked as permanent by a domain expert.

---

<p align="center"><a href="../README.md">← Back to all questions</a></p>
