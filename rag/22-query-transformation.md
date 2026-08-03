# Part 22: Query Transformation

> **The Question:** "What is query transformation in RAG (HyDE, query decomposition, step-back prompting)?"

---

## The Technical Breakdown

### Why Transform Queries?

User queries are often short, ambiguous, or use different vocabulary than the indexed documents. Query transformation rewrites the query to improve retrieval.

```
Original query:   "Why is my app slow?"
Problem:          Vague, could match anything about performance.

Transformed:      "Application performance degradation causes: 
                   memory leaks, database query optimization, 
                   network latency, CPU bottlenecks"
Result:           Better semantic match with technical docs.
```

### The Main Query Transformation Techniques

#### 1. HyDE (Hypothetical Document Embeddings)

Generate a **hypothetical answer** and use IT for retrieval instead of the query:

```
Query: "What causes database deadlocks?"

Step 1: LLM generates a hypothetical answer:
  "Database deadlocks occur when two or more transactions
   hold locks on resources that the other transactions need.
   Common causes include: circular lock dependencies, 
   long-running transactions, and poor indexing..."

Step 2: Embed the hypothetical answer (not the original query)

Step 3: Search vector DB with the hypothetical answer embedding

Why it works: The hypothetical answer is longer and uses
the same vocabulary as actual documents, producing a better
embedding for retrieval.
```

#### 2. Query Decomposition

Break a complex query into simpler sub-queries:

```
Original: "Compare AWS and GCP pricing for GPU instances 
           suitable for LLM fine-tuning"

Decomposed:
  1. "AWS GPU instance types for machine learning"
  2. "AWS GPU instance pricing 2024"
  3. "GCP GPU instance types for machine learning"  
  4. "GCP GPU instance pricing 2024"
  5. "GPU requirements for LLM fine-tuning"

Retrieve for each sub-query, combine results, generate answer.
```

#### 3. Step-Back Prompting

Generate a more general, abstract query first:

```
Original: "Why did my Python 3.11 asyncio task 
           throw CancelledError after timeout?"

Step-back: "How does asyncio handle task cancellation 
            and timeouts in Python?"

The step-back query retrieves broader, more foundational
docs that likely contain the specific answer.
```

#### 4. Query Expansion

Add related terms to broaden retrieval:

```
Original: "heart attack symptoms"
Expanded: "heart attack symptoms myocardial infarction 
           cardiac event chest pain warning signs"
```

#### 5. Query Rewriting

Use an LLM to rewrite the query for better retrieval:

```python
rewritten = llm.generate(f"""
Rewrite this user query to be more specific and search-friendly.
Add relevant technical terms that documents might use.

Original query: {user_query}
Rewritten query:
""")
```

### Comparison of Techniques

| Technique | When to Use | Cost | Quality Impact |
|-----------|-------------|------|----------------|
| **HyDE** | Short/vague queries | 1 LLM call | +15-25% retrieval |
| **Decomposition** | Multi-part questions | 1 LLM call | +20-30% for complex Q |
| **Step-back** | Overly specific queries | 1 LLM call | +10-20% retrieval |
| **Expansion** | Domain jargon gaps | 1 LLM call or synonym dict | +5-15% recall |
| **Rewriting** | General improvement | 1 LLM call | +10-20% retrieval |

---

## Accuracy Check
| Claim | Verified? |
|-------|-----------|
| HyDE uses hypothetical documents for embedding-based retrieval | ✅ Gao et al. (2023), "Precise Zero-Shot Dense Retrieval without Relevance Labels" |
| Query decomposition improves multi-hop question retrieval | ✅ Press et al. (2023), "Measuring and Narrowing the Compositionality Gap" |
| Step-back prompting generates broader queries for better recall | ✅ Zheng et al. (2023), "Take a Step Back" |

## Scenario Examples
### A: A developer support bot receives "segfault in prod" — a 3-word query. HyDE generates a hypothetical answer: "A segmentation fault in production is typically caused by null pointer dereference, buffer overflow, or accessing freed memory. Common debugging steps include checking core dumps, running with AddressSanitizer, and reviewing recent code changes." This hypothetical is embedded and retrieves debugging guides that the 3-word query would never match.
### B: A financial analyst asks "How does Nvidia's AI chip business compare to AMD's in terms of market share, revenue growth, and product lineup?" Query decomposition breaks this into 6 sub-queries (3 topics × 2 companies). Each sub-query retrieves targeted financial documents. The combined context gives the LLM everything it needs for a comprehensive comparison table.

## Follow-Up Questions
### Q1: "Does HyDE always improve retrieval?"
**Answer:** No — HyDE can hurt if the LLM's hypothetical answer is wrong or off-topic. If the model generates an inaccurate hypothetical, the embedding will be biased toward incorrect information. HyDE works best when: (1) the LLM has reasonable domain knowledge, (2) the query is short/vague, (3) you're doing factual retrieval. For highly specialized domains where the LLM lacks knowledge, HyDE's hypothetical may mislead retrieval.

### Q2: "How much latency does query transformation add?"
**Answer:** One LLM call per transformation — typically 200-500ms. For query decomposition with 4 sub-queries: 200ms for decomposition + 4× retrieval time. Mitigations: (1) run sub-query retrievals in parallel, (2) use a fast model (GPT-4o-mini) for transformation, (3) cache transformations for repeated queries. The quality improvement usually justifies the latency cost, but for real-time applications, consider skipping transformation for simple queries.

### Q3: "Can you combine multiple transformation techniques?"
**Answer:** Yes — the pipeline approach: (1) Rewrite the query for clarity, (2) decompose if complex, (3) expand each sub-query with domain terms. In practice, most teams pick one primary technique based on their query patterns. HyDE for short/vague queries, decomposition for complex multi-part queries, step-back for overly specific queries. A router can classify the query type and select the appropriate transformation.

---

<p align="center"><a href="../README.md">← Back to all questions</a></p>
