# Scenario 5: Per-User Access Control

> **The Scenario:** "You need per-user access control on a RAG system that searches internal company documents. How do you implement it?"

---

## Solutions

1. **Metadata-based filtering** — Store access control metadata (department, role, clearance level) with each chunk. At query time, filter to only chunks the user has permission to see.
2. **Tenant isolation** — Separate vector collections/namespaces per team or access level. Users only query their permitted collections.
3. **Pre-query ACL resolution** — Before retrieval, resolve the user's permissions from your IAM system (Okta, Azure AD), then construct the metadata filter.
4. **Document-level permissions** — Inherit access controls from the source system (Google Drive sharing, Confluence space permissions). Sync these to chunk metadata during ingestion.
5. **Post-retrieval filtering** — Retrieve broadly, then filter results based on permissions before sending to the LLM. Simpler but wastes retrieval budget on inaccessible chunks.
6. **Row-level security in the vector DB** — Some databases support native RLS policies that enforce access control at the query level.

```python
# Pre-query ACL resolution pattern
def secure_retrieve(query, user):
    # Resolve user permissions from IAM
    user_groups = iam.get_groups(user.id)        # ["engineering", "all-hands"]
    user_clearance = iam.get_clearance(user.id)  # "internal"
    
    # Build metadata filter
    filter = {
        "$and": [
            {"access_groups": {"$in": user_groups}},
            {"clearance_level": {"$lte": user_clearance}},
        ]
    }
    
    # Search with access filter
    return vector_db.search(
        query_embedding=embed(query),
        filter=filter,
        top_k=5
    )
```

## Scenario Examples
### A: A law firm with 200 lawyers across 5 practice groups. Each group's client documents are confidential — corporate lawyers shouldn't see litigation docs. Implementation: during ingestion, each document inherits its Casepoint access group as metadata. At query time, the lawyer's practice group is resolved from Azure AD, and retrieval is filtered to `practice_group IN user.groups`. A corporate lawyer querying "contract termination clause" only retrieves corporate contract templates, never litigation case files.
### B: A multi-tenant SaaS platform where each customer has their own knowledge base in the same Pinecone index. Namespace isolation: each customer gets a separate Pinecone namespace (`customer_abc`, `customer_xyz`). Customer A's queries only search `customer_abc`. Even if the embeddings are similar across customers, namespace isolation guarantees zero data leakage. A customer searching "pricing" only sees THEIR pricing docs.

## Follow-Up Questions
### Q1: "How do you handle users who belong to multiple groups with different access levels?"
**Answer:** Union of permissions (most common) — the user can access documents from ANY of their groups. Implementation: filter with `$in` operator matching any of the user's groups. For stricter scenarios (government, healthcare), use intersection — the user must have ALL required clearances. Some documents have multiple access requirements: `required_clearances: ["top-secret", "project-omega"]` — the user must hold both.

### Q2: "What happens if permissions change after documents are indexed?"
**Answer:** Two approaches: (1) **Re-sync on change** — when ACLs change in the source system (Google Drive sharing updated), trigger a metadata update in the vector DB. No need to re-embed — just update the metadata. (2) **Real-time resolution** — don't store permissions in the vector DB at all. At query time, retrieve broadly, then check each result's source document permissions against the current IAM state. More accurate but slower.

### Q3: "Can the LLM itself leak restricted information?"
**Answer:** Yes — if restricted chunks are included in the prompt context, the LLM might reference them in its response. The access filter must happen BEFORE the LLM sees any context. Never rely on prompting ("don't mention restricted information") — the LLM can't reliably enforce access control. Additional safeguard: post-generation check that scans the response for content matching restricted documents.

---

<p align="center"><a href="../README.md">← Back to all questions</a></p>
