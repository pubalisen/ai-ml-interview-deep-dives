# Part 25: Parent-Child Chunking

> **The Question:** "What is parent-child chunking, and how does it improve retrieval?"

---

## The Technical Breakdown

### The Problem with Fixed Chunks

Standard chunking forces a trade-off:
- **Small chunks** (256 tokens): precise retrieval but too little context for generation
- **Large chunks** (2048 tokens): rich context but imprecise retrieval (diluted embeddings)

```
Small chunk retrieved: "The refund window is 30 days."
→ Precise match, but the LLM doesn't know about exceptions, process, or eligibility.

Large chunk retrieved: [2000 tokens mixing refund, shipping, warranty, exchange policies]
→ Contains the answer but also noise. Embedding quality is diluted.
```

### Parent-Child Chunking: Best of Both Worlds

Split documents into two levels:
- **Child chunks** (small, ~256 tokens) — used for embedding and retrieval
- **Parent chunks** (large, ~1024-2048 tokens) — used for LLM context

```
Document: "Complete Returns & Refund Policy"
    │
    ├── Parent Chunk 1 (1024 tokens): "Section: Return Eligibility..."
    │       ├── Child 1 (256 tokens): "Items purchased within 30 days..."
    │       ├── Child 2 (256 tokens): "Electronics must be unopened..."
    │       ├── Child 3 (256 tokens): "Sale items are final sale..."
    │       └── Child 4 (256 tokens): "Gift card purchases are..."
    │
    └── Parent Chunk 2 (1024 tokens): "Section: Refund Process..."
            ├── Child 5 (256 tokens): "To initiate a return, visit..."
            ├── Child 6 (256 tokens): "Refunds are processed within..."
            └── Child 7 (256 tokens): "For international orders..."
```

### How It Works

```
Step 1: RETRIEVAL (search small child chunks)
  Query: "Can I return an opened laptop?"
  Match: Child 2 ("Electronics must be unopened...") — Score: 0.92
  
Step 2: EXPANSION (fetch the parent chunk)
  Child 2 → belongs to → Parent Chunk 1
  Parent Chunk 1 contains: eligibility rules, exceptions, conditions
  
Step 3: GENERATION (use parent chunk as context)
  Context: Parent Chunk 1 (1024 tokens of complete return eligibility info)
  → LLM has enough context to give a complete, nuanced answer
```

### Implementation

```python
# Indexing
for document in documents:
    parent_chunks = split(document, size=1024)
    
    for parent in parent_chunks:
        parent_id = generate_id()
        store_parent(parent_id, parent.text)  # Store in document store
        
        children = split(parent.text, size=256)
        for child in children:
            embedding = embed(child.text)
            vector_db.upsert(
                id=child.id,
                vector=embedding,
                metadata={"parent_id": parent_id, "text": child.text}
            )

# Retrieval
def retrieve_with_parent_expansion(query, top_k=5):
    # Search child chunks
    child_results = vector_db.search(embed(query), top_k=top_k)
    
    # Fetch parent chunks (deduplicated)
    parent_ids = set(r.metadata["parent_id"] for r in child_results)
    parent_texts = [get_parent(pid) for pid in parent_ids]
    
    return parent_texts  # Use these as LLM context
```

### Variations

| Strategy | Retrieval Unit | Context Unit |
|----------|---------------|-------------|
| **Standard** | 512 tokens | Same 512 tokens |
| **Parent-child** | 256 tokens | 1024 token parent |
| **Sentence window** | Single sentence | ±3 surrounding sentences |
| **Section retrieval** | Paragraph | Entire section |
| **Document retrieval** | Paragraph | Entire document |

### Quality Impact

Typical improvements from parent-child over standard chunking:

| Metric | Standard Chunking | Parent-Child | Change |
|--------|------------------|-------------|--------|
| Retrieval precision | 0.72 | 0.82 | +14% |
| Answer completeness | 0.61 | 0.79 | +30% |
| Answer correctness | 0.74 | 0.84 | +14% |

---

## Accuracy Check
| Claim | Verified? |
|-------|-----------|
| Parent-child chunking separates retrieval units from context units | ✅ LlamaIndex ParentNodeParser, industry standard |
| Small chunks produce more precise embeddings for retrieval | ✅ Empirically validated — smaller = more focused vectors |
| Expanding to parent chunks provides better generation context | ✅ LlamaIndex, LangChain parent document retriever benchmarks |

## Scenario Examples
### A: A technical documentation system uses parent-child chunking where child chunks are individual API endpoint descriptions (small, precise) and parent chunks are entire API sections (authentication, data, billing). When a user asks "How do I refresh an OAuth token?", the child chunk for the refresh endpoint is matched precisely. The parent chunk provides the full authentication section — including token lifetimes, error handling, and code examples — giving the LLM rich context for a complete answer.
### B: A medical knowledge base uses sentence-level retrieval with paragraph-level context. Query: "What is the maximum daily dose of ibuprofen?" The exact sentence "Maximum daily dose: 3200mg for prescription, 1200mg OTC" is matched. The surrounding paragraph provides critical context about age adjustments, contraindications, and duration limits. Without the parent context, the LLM might give the dose without the safety caveats.

## Follow-Up Questions
### Q1: "How do you choose the right child and parent sizes?"
**Answer:** Child size should be the smallest self-contained semantic unit — typically a paragraph or 2-3 sentences (200-400 tokens). Parent size should be the smallest complete context — typically a section or full topic (800-2000 tokens). Test empirically: too-small children lose meaning ("30 days" without context is ambiguous). Too-large parents dilute the context window. Start with 256/1024 and adjust based on retrieval evaluation.

### Q2: "Does parent-child chunking increase storage requirements?"
**Answer:** Marginally. You store both child embeddings (in the vector DB) and parent text (in a document store). The vector storage is the same total — you have more vectors but each is smaller. The document store adds raw text storage for parents. For 1M documents: standard approach stores ~1M chunks. Parent-child stores ~4M child vectors + 1M parent texts. Vector storage increases ~4x, but with smaller dimensions this may not increase total size significantly.

### Q3: "What about sentence window retrieval — how does it compare?"
**Answer:** Sentence window is a lightweight alternative: retrieve a single sentence, but expand the context to include N sentences before and after it. Simpler than parent-child (no explicit hierarchy), works well for prose where sentences are the natural unit. Doesn't work as well for structured documents (tables, lists, code) where parent-child's explicit section boundaries are better. Sentence window: good default. Parent-child: best for structured/hierarchical documents.

---

<p align="center"><a href="../README.md">← Back to all questions</a></p>
