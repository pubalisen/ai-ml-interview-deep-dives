# Scenario 9: Multi-Hop Question Failure

> **The Scenario:** "Your RAG system fails on questions that require combining facts from multiple documents. How do you solve it?"

---

## Solutions

1. **Query decomposition** — Use an LLM to break the complex question into sub-questions. Retrieve for each sub-question independently, then synthesize.
2. **Iterative retrieval (FLARE pattern)** — Generate a partial answer, identify what's missing, retrieve more context, continue generating. Repeat until the answer is complete.
3. **Knowledge graph augmentation** — Build a graph of entities and relationships across documents. Traverse the graph to find connected facts.
4. **Chain-of-thought retrieval** — Prompt the model to reason step-by-step, triggering retrieval at each reasoning step.
5. **Increased context via re-ranking** — Retrieve top-50 (instead of top-5) to increase the chance of capturing all needed facts, then re-rank to select the most relevant combination.
6. **Pre-computed cross-references** — During indexing, link chunks that reference the same entities or topics. When one chunk is retrieved, automatically include its linked chunks.

```python
# Query decomposition pattern
def multi_hop_rag(complex_query):
    # Step 1: Decompose
    sub_queries = llm.generate(f"""
    Break this question into simple sub-questions:
    {complex_query}
    """)
    
    # Step 2: Retrieve for each sub-query
    all_context = []
    for sub_q in sub_queries:
        chunks = retrieve(sub_q, top_k=3)
        all_context.extend(chunks)
    
    # Step 3: De-duplicate and re-rank combined context
    unique_context = deduplicate(all_context)
    reranked = reranker.rerank(complex_query, unique_context, top_n=8)
    
    # Step 4: Synthesize final answer
    return llm.generate(complex_query, context=reranked)
```

## Scenario Examples
### A: A healthcare analyst asks "Which approved cancer drugs targeting EGFR have shown efficacy in combination with immunotherapy in Phase 3 trials?" This requires: (1) list of EGFR-targeting drugs, (2) FDA approval status for each, (3) clinical trial data for combination with immunotherapy. No single document contains all three. Query decomposition: sub-query 1 retrieves EGFR drug lists, sub-query 2 retrieves approval statuses, sub-query 3 retrieves combination trial results. The synthesized answer cross-references all three to provide a complete table.
### B: A due diligence analyst asks "Has any portfolio company's CTO previously worked at a company that was sued for patent infringement?" This is a 3-hop question: portfolio companies → CTOs → previous employers → patent lawsuits. Iterative retrieval: (1) retrieve portfolio company list → identify CTOs, (2) retrieve each CTO's LinkedIn/bio → identify previous employers, (3) search for patent infringement cases involving those employers. Without multi-hop retrieval, the system returns "I don't have this information" because no single document connects all the dots.

## Follow-Up Questions
### Q1: "How do you detect that a question requires multi-hop reasoning?"
**Answer:** Use an LLM classifier: "Does this question require information from multiple topics or documents? Answer yes/no and list the sub-topics." Multi-hop indicators: (1) comparative questions ("compare A and B"), (2) conditional questions ("if X, then what about Y?"), (3) questions with multiple entities, (4) questions requiring computation across facts. Route single-hop queries to standard RAG (fast, cheap) and multi-hop to the decomposition pipeline (thorough, expensive).

### Q2: "What's the latency impact of multi-hop retrieval?"
**Answer:** Significant — 2-5x standard RAG. A 3-hop query requires 3 retrieval rounds and potentially intermediate LLM calls. Typical: single-hop = 800ms, 3-hop = 2.5-4 seconds. Mitigations: (1) run sub-query retrievals in parallel, (2) use cached embeddings, (3) use a fast model for decomposition (GPT-4o-mini), (4) set a max-hops limit to prevent runaway chains.

### Q3: "How do you evaluate multi-hop RAG accuracy?"
**Answer:** Standard retrieval metrics aren't sufficient. You need: (1) **Sub-question coverage** — did the system correctly identify all required sub-questions? (2) **Per-hop recall** — was the right chunk found for each sub-question? (3) **Synthesis accuracy** — did the LLM correctly combine the facts? Datasets: HotpotQA, MuSiQue, and 2WikiMultiHopQA provide labeled multi-hop evaluation data with annotated reasoning chains.

---

<p align="center"><a href="../README.md">← Back to all questions</a></p>
