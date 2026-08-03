# Part 17: RAG Failure Modes

> **The Question:** "What are the common failure modes of RAG systems, and how do you debug them?"

---

## The Technical Breakdown

### The Failure Taxonomy

RAG failures fall into two categories: **retrieval failures** (wrong context) and **generation failures** (wrong answer despite right context).

```
┌─────────────────────────────────────────┐
│           RAG FAILURE MODES              │
├────────────────────┬────────────────────┤
│  RETRIEVAL FAILURES│ GENERATION FAILURES│
│                    │                    │
│  1. Missing chunks │ 5. Hallucination   │
│  2. Wrong chunks   │ 6. Context ignored │
│  3. Partial context│ 7. Wrong synthesis │
│  4. Stale data     │ 8. Formatting fail │
└────────────────────┴────────────────────┘
```

### Retrieval Failures

#### 1. Missing Chunks (Low Recall)
The relevant document exists but wasn't retrieved.

```
Root causes:
  - Embedding model doesn't capture the semantic relationship
  - Chunk size too large (signal diluted in a large chunk)
  - Query and document use different terminology
  
Debug: Manually search your vector DB for the known-correct chunk.
  If it exists but wasn't retrieved → embedding/similarity issue
  If it doesn't exist → ingestion/chunking issue
```

#### 2. Wrong Chunks (Low Precision)
Retrieved chunks are topically related but don't contain the answer.

```
Query: "What is the cancellation fee?"
Retrieved: General pricing page (mentions prices but not cancellation)

Debug: Inspect the top-10 retrieved chunks. If they're topically
adjacent but miss the specific answer → chunking too coarse or
keyword matching needed (hybrid search).
```

#### 3. Partial Context
The answer spans multiple chunks but only one was retrieved.

```
Chunk A: "The refund policy allows returns within 30 days..."
Chunk B: "...except for sale items, which are final sale."

Only Chunk A retrieved → Answer misses the exception.

Fix: Increase chunk overlap, use parent-child chunking,
or increase top-k to retrieve more chunks.
```

#### 4. Stale Data
Documents have been updated but the index hasn't been refreshed.

### Generation Failures

#### 5. Hallucination Despite Context
The LLM generates facts not in the retrieved context.

```
Context: "We offer 20 vacation days."
Answer: "You get 20 vacation days plus 5 sick days."
← "5 sick days" was hallucinated, not in context.

Fix: Strengthen system prompt, lower temperature,
add faithfulness evaluation.
```

#### 6. Context Ignored
The answer is in the context but the LLM ignores it.

```
Usually caused by:
  - Lost in the middle problem
  - Context too long (relevant info buried)
  - Conflicting info (model picks the wrong one)
  
Fix: Re-rank to put relevant context first,
reduce number of chunks, use structured context labels.
```

### The Debugging Workflow

```
User reports wrong answer
        │
        ▼
Step 1: Was the correct chunk retrieved?
        ├── No → RETRIEVAL PROBLEM
        │   ├── Does the chunk exist in the index? → Re-index
        │   ├── Is the embedding similarity low? → Try hybrid search
        │   └── Is the query ambiguous? → Add query expansion
        │
        └── Yes → GENERATION PROBLEM
            ├── Is the context too long? → Reduce chunks, re-rank
            ├── Is the prompt unclear? → Improve system prompt
            └── Is the model hallucinating? → Lower temperature, add grounding
```

---

## Accuracy Check
| Claim | Verified? |
|-------|-----------|
| RAG failures split into retrieval and generation categories | ✅ Barnett et al. (2024), "Seven Failure Points When Engineering a RAG" |
| Low recall is typically caused by embedding or chunking issues | ✅ Industry experience, LlamaIndex debugging guides |
| LLMs can hallucinate even with correct context provided | ✅ Shuster et al. (2021), documented in multiple studies |

## Scenario Examples
### A: A customer support bot gives wrong shipping times. Debugging: (1) Check retrieved chunks — the old 5-7 days chunk is retrieved, but the updated 3-5 days document exists in S3. Problem: the index wasn't refreshed after the policy update. The old chunk's embedding is being matched. Fix: re-index updated documents, add a freshness timestamp to metadata, filter by most recent version.
### B: A legal research tool retrieves the correct statute but the LLM's answer contradicts it. The statute says "within 90 days" but the answer says "within 60 days." Debugging: (1) The correct chunk IS in the context. (2) The temperature is 0.7 (too high for factual extraction). (3) Another retrieved chunk from a different jurisdiction says 60 days. Fix: lower temperature to 0, add "cite which source you used" to the prompt, filter retrieval by jurisdiction metadata.

## Follow-Up Questions
### Q1: "How do you build a systematic debugging process for RAG?"
**Answer:** Log everything: (1) The original query, (2) the embedding used, (3) every retrieved chunk with its score, (4) the assembled prompt, (5) the model's response. Store these in a tracing system (LangSmith, Arize Phoenix). When a failure occurs, you can replay the exact pipeline and identify which step failed. Without logging, you're guessing. With logging, you're diagnosing.

### Q2: "What's the most common failure mode in production RAG?"
**Answer:** Low retrieval recall — the answer exists in your knowledge base but isn't retrieved. This accounts for ~50-60% of RAG failures. The fix is usually a combination of: (1) better chunking (don't split the answer across chunks), (2) hybrid search (catch keyword matches that vector search misses), (3) query expansion (rephrase the query to match how the document was written). Generation failures are the other ~40%, mostly hallucination.

### Q3: "How do you prevent RAG failures proactively rather than reactively?"
**Answer:** Three strategies: (1) **Automated evaluation pipeline** — run a test suite of 100+ known queries against your RAG system on every deployment. Alert if metrics drop. (2) **User feedback loop** — add thumbs up/down on responses and route negative feedback to a review queue. (3) **Confidence scoring** — if retrieval similarity scores are below a threshold (e.g., 0.7), the system says "I'm not confident" instead of guessing. Catch failures before users see them.

---

<p align="center"><a href="../README.md">← Back to all questions</a></p>
