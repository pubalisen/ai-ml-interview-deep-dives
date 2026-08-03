# Scenario 1: RAG Hallucination Despite Correct Context

> **The Scenario:** "Your RAG system is hallucinating despite having the right context. How do you fix it?"

---

## Solutions

1. **Lower temperature to 0** — Reduce randomness in generation. For factual QA, temperature 0 ensures the model picks the most likely token, reducing creative fabrication.
2. **Strengthen the system prompt** — Add explicit instructions: "Answer ONLY based on the provided context. If the information is not in the context, say 'I don't have this information.' Do NOT use your general knowledge."
3. **Reduce context volume** — Too many retrieved chunks dilute the signal. Re-rank and keep only the top 3-5 most relevant chunks. The "lost in the middle" problem causes the model to ignore key context.
4. **Add faithfulness verification** — Post-generation, use an NLI model to verify each claim in the answer is supported by the retrieved context. Block unsupported claims.
5. **Use structured context** — Label chunks with relevance scores: "MOST RELEVANT: [chunk]" and "SUPPORTING: [chunk]" to guide the model's attention.
6. **Switch to a more instruction-following model** — Some models (GPT-4o, Claude 3.5 Sonnet) follow grounding instructions more reliably than others.

## Scenario Examples
### A: A financial advisor bot retrieves the correct earnings report but adds "and analysts predict 15% growth next quarter" — a hallucinated claim not in any retrieved document. Fix: temperature → 0, add "Never predict or speculate beyond what the documents state," and add post-generation NLI check. Hallucination rate drops from 18% to 3%.
### B: A healthcare FAQ bot retrieves the correct drug information but confuses dosages between two drugs mentioned in different chunks. The context contains both Metformin (500mg) and Glipizide (5mg), but the answer swaps them. Fix: reduce to 3 chunks (only the most relevant drug), add structured labels, and re-rank to ensure the queried drug's info is first.

## Follow-Up Questions
### Q1: "How do you distinguish between a retrieval failure and a generation failure?"
**Answer:** Log the retrieved chunks alongside the generated answer. If the correct answer IS in the retrieved chunks but the LLM's response differs → generation failure. If the correct answer is NOT in the retrieved chunks → retrieval failure. Build a debugging pipeline that checks this automatically for flagged responses. RAGAS faithfulness metric quantifies this: high context_recall + low faithfulness = generation problem.

### Q2: "Can you completely eliminate hallucination in RAG?"
**Answer:** Not completely, but you can reduce it to near-zero for most use cases. The residual risk comes from: (1) ambiguous context that the model misinterprets, (2) implicit reasoning the model applies beyond the text, (3) formatting/phrasing that unintentionally changes meaning. Multiple layers of defense (prompt engineering + low temperature + NLI verification + human review for critical domains) get you to <1% hallucination.

### Q3: "Does using a larger context window help or hurt hallucination?"
**Answer:** It usually hurts. More context = more noise = more opportunities for the model to blend information from different sources incorrectly. The optimal strategy is LESS context of HIGHER quality. Re-rank aggressively, keep only the most relevant 3-5 chunks, and ensure they're coherent and on-topic. A focused 2K token context produces more faithful answers than a 20K token context with mixed-relevance chunks.

---

<p align="center"><a href="../README.md">← Back to all questions</a></p>
