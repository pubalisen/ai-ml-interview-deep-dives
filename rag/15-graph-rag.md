# Part 15: GraphRAG

> **The Question:** "What is GraphRAG, and when would you use it over traditional RAG?"

---

## The Technical Breakdown

### What Is GraphRAG?

GraphRAG augments retrieval with a **knowledge graph** — a structured network of entities and relationships extracted from your documents. Instead of retrieving flat text chunks, you traverse connected concepts to find information that spans multiple documents.

```
Traditional RAG:
  Query → Vector search → Flat text chunks → LLM

GraphRAG:
  Query → Entity extraction → Graph traversal → Connected entities + context → LLM
```

### The Knowledge Graph Structure

```
[Aspirin] ──treats──→ [Headache]
    │                      │
    ├──side_effect──→ [Stomach bleeding]
    │                      │
    ├──interacts_with→ [Warfarin] ──treats──→ [Blood clots]
    │
    └──manufactured_by→ [Bayer] ──headquartered──→ [Germany]
```

Each node is an entity (drug, condition, company). Each edge is a relationship (treats, causes, interacts_with). The graph captures **structured knowledge** that flat text chunks miss.

### How GraphRAG Works

```
Step 1: BUILD THE GRAPH (offline)
  Documents → LLM extracts entities and relationships → Store in graph DB

Step 2: INDEX (offline)
  Create community summaries at different hierarchy levels
  Entities → Communities → Community summaries

Step 3: QUERY (online)
  Query → Identify relevant entities → Traverse graph →
  Collect entity descriptions + relationship context + community summaries →
  LLM generates answer from structured context
```

### Microsoft's GraphRAG Approach

Microsoft Research's GraphRAG (2024) introduced a specific method:

1. **Entity extraction** — Use LLM to extract all entities and relationships from each chunk
2. **Community detection** — Apply Leiden algorithm to find clusters of related entities
3. **Community summaries** — Generate LLM summaries for each community
4. **Hierarchical indexing** — Create summaries at multiple levels (local → global)
5. **Query modes:**
   - **Local search** — For specific questions: find relevant entities → retrieve their neighborhoods
   - **Global search** — For broad questions: use community summaries for high-level understanding

### GraphRAG vs Traditional RAG

| Aspect | Traditional RAG | GraphRAG |
|--------|----------------|----------|
| Best for | Specific factual questions | Relational, multi-hop, summarization |
| Data structure | Flat chunks | Connected graph |
| "How are X and Y related?" | Poor (needs luck in retrieval) | Excellent (direct edge traversal) |
| "Summarize all themes" | Poor (random chunk sampling) | Excellent (community summaries) |
| Indexing cost | Low (embed chunks) | High (LLM extraction + graph building) |
| Query latency | Fast | Moderate (graph traversal) |
| Maintenance | Update chunks | Update graph edges + re-cluster |

### When to Use GraphRAG

```
✅ Use GraphRAG when:
  - Questions involve relationships ("How is drug X related to disease Y?")
  - Multi-hop reasoning is common ("Who funds the company that makes the drug that treats...?")
  - Global summarization is needed ("What are the main themes across all documents?")
  - Entity-centric queries dominate ("Tell me everything about Entity X")
  - Data is inherently relational (medical, legal, financial)

❌ Stick with traditional RAG when:
  - Simple factual lookups ("What's the refund policy?")
  - Documents are independent (FAQ entries, product descriptions)
  - Budget is limited (GraphRAG indexing costs 10-50x more)
  - Low latency is critical
```

---

## Accuracy Check
| Claim | Verified? |
|-------|-----------|
| GraphRAG uses knowledge graphs to improve multi-hop retrieval | ✅ Edge et al. (2024), "From Local to Global: A Graph RAG Approach" (Microsoft Research) |
| Community detection enables global summarization queries | ✅ Edge et al. (2024), Leiden algorithm + community summaries |
| GraphRAG outperforms standard RAG on summarization and relationship queries | ✅ Microsoft GraphRAG benchmarks on comprehensiveness and diversity |
| GraphRAG indexing costs significantly more than standard RAG | ✅ Documented — requires LLM calls per chunk for entity extraction |

## Scenario Examples
### A: A pharmaceutical company needs to answer "What drugs interact with metformin across all our clinical trial reports?" Traditional RAG retrieves chunks mentioning metformin but misses drug interactions buried in other sections. GraphRAG extracts a graph: [Metformin]→interacts_with→[Insulin], [Metformin]→interacts_with→[Alcohol], [Metformin]→contraindicated→[Kidney disease]. A graph traversal from the Metformin node finds ALL interactions in one query — no chunks to miss.
### B: A consulting firm analyzes 500 industry reports to answer "What are the emerging trends in AI infrastructure?" Traditional RAG returns random relevant chunks from random reports. GraphRAG builds communities: (GPU/chip manufacturing cluster, cloud compute cluster, edge AI cluster, AI energy consumption cluster). Community summaries give a structured, comprehensive overview of all trends, ensuring no major theme is missed.

## Follow-Up Questions
### Q1: "How expensive is GraphRAG compared to standard RAG?"
**Answer:** Significantly more expensive at indexing time. For 1,000 documents: standard RAG embedding costs ~$5-10. GraphRAG entity extraction requires LLM calls per chunk (potentially 5,000+ GPT-4 calls = $50-200), plus community detection and summary generation (another $20-50). Total: 10-50x more expensive to index. However, query-time costs are comparable. The investment pays off when your use case genuinely requires relational reasoning.

### Q2: "Can you combine GraphRAG with traditional vector RAG?"
**Answer:** Yes, and this is the recommended production approach. Use vector RAG for simple factual queries (fast, cheap) and GraphRAG for relationship and summarization queries (thorough, expensive). Route queries based on type: entity questions ("What is X?") → vector RAG. Relationship questions ("How are X and Y connected?") → GraphRAG. Summarization ("What are the main themes?") → GraphRAG global search.

### Q3: "How do you maintain a knowledge graph when documents change?"
**Answer:** The hardest operational challenge. Options: (1) **Full rebuild** — re-extract entities from changed documents, rebuild affected subgraphs. Expensive but simple. (2) **Incremental updates** — detect which entities/relationships changed, update only those graph nodes. Complex but efficient. (3) **Versioned graphs** — maintain graph snapshots, query the latest version. Best for audit trails. Most teams start with periodic full rebuilds (weekly) and move to incremental updates as the system matures.

---

<p align="center"><a href="../README.md">← Back to all questions</a></p>
