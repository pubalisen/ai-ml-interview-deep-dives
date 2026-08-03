# Part 13: RAG Evaluation

> **The Question:** "How do you evaluate a RAG system? Explain faithfulness, relevance, and context precision/recall."

---

## The Technical Breakdown

### Why RAG Evaluation Is Different

RAG has two components that can fail independently:
1. **Retrieval** — Did you find the right documents?
2. **Generation** — Did the LLM use them correctly?

A wrong answer could mean bad retrieval (right answer existed but wasn't found) OR bad generation (right context was found but LLM ignored it).

```
                    Generation Quality
                    Good            Bad
Retrieval   Good  │ ✅ Correct   │ ❌ Hallucination despite context
Quality     Bad   │ ❌ Lucky     │ ❌ Wrong everything
```

### The Four Core Metrics

#### 1. Faithfulness (Answer ↔ Context)
Does the answer **only contain information from the retrieved context**?

```
Context: "The refund window is 30 days for electronics."
Question: "What's the refund policy?"

Faithful:     "Electronics can be refunded within 30 days."
Unfaithful:   "Electronics can be refunded within 30 days, and clothing within 60 days."
              ↑ The "60 days for clothing" was hallucinated — not in context.
```

Measurement: Compare each claim in the answer against the retrieved context. Score = claims_supported / total_claims.

#### 2. Answer Relevance (Answer ↔ Question)
Does the answer **actually address the user's question**?

```
Question: "How do I reset my password?"

Relevant:    "Go to Settings > Security > Reset Password and follow the steps."
Irrelevant:  "Password security is important. Use a mix of letters and numbers."
             ↑ Related topic, but doesn't answer the question.
```

Measurement: Generate hypothetical questions from the answer and check semantic similarity with the original question.

#### 3. Context Precision (Retrieved Context ↔ Question)
Of the retrieved chunks, **how many are actually relevant** to the question?

```
Question: "What's the return policy?"
Retrieved chunks:
  1. Return policy details ← Relevant ✅
  2. Shipping information  ← Irrelevant ❌
  3. Payment methods       ← Irrelevant ❌
  4. Return form template  ← Relevant ✅
  5. Company history       ← Irrelevant ❌

Context Precision = 2/5 = 0.40
```

#### 4. Context Recall (Retrieved Context ↔ Ground Truth)
Of all the information needed to answer correctly, **how much was retrieved**?

```
Ground truth answer requires: refund window (30 days) + exceptions (sale items) + process (online form)

Retrieved context contains:
  ✅ Refund window: 30 days
  ✅ Process: online form
  ❌ Exceptions: not retrieved

Context Recall = 2/3 = 0.67
```

### Evaluation Frameworks

| Framework | Metrics | Method |
|-----------|---------|--------|
| **RAGAS** | Faithfulness, relevance, precision, recall | LLM-as-judge |
| **DeepEval** | All RAGAS metrics + hallucination, toxicity | LLM-as-judge + NLI |
| **TruLens** | Groundedness, relevance, context relevance | LLM-as-judge |
| **LangSmith** | Custom metrics, tracing | Custom evaluators |
| **Arize Phoenix** | Retrieval + generation metrics | LLM-as-judge + embeddings |

### The RAGAS Evaluation Pipeline

```python
from ragas import evaluate
from ragas.metrics import faithfulness, answer_relevancy, context_precision, context_recall

results = evaluate(
    dataset=eval_dataset,  # (question, answer, contexts, ground_truth)
    metrics=[faithfulness, answer_relevancy, context_precision, context_recall],
    llm=ChatOpenAI(model="gpt-4o"),  # Judge model
)

# Output: { faithfulness: 0.82, answer_relevancy: 0.91, context_precision: 0.75, context_recall: 0.68 }
```

---

## Accuracy Check
| Claim | Verified? |
|-------|-----------|
| Faithfulness measures whether the answer is supported by retrieved context | ✅ Es et al. (2024), RAGAS paper |
| Context precision measures the ratio of relevant chunks in retrieved set | ✅ Standard IR metric adapted for RAG |
| LLM-as-judge is the dominant evaluation method for RAG | ✅ RAGAS, DeepEval, TruLens all use this approach |
| RAG evaluation requires evaluating retrieval and generation separately | ✅ Industry consensus, documented across frameworks |

## Scenario Examples
### A: A healthcare RAG system shows: faithfulness=0.95 (great, rarely hallucinates), answer_relevancy=0.88 (good), but context_recall=0.45 (terrible). Diagnosis: the retrieval is missing key medical guidelines. The LLM is faithful to what it receives but answers are incomplete. Fix: improve chunking of medical guidelines, add hybrid search for drug names and condition codes.
### B: An internal knowledge base shows: context_precision=0.90 (retrieving relevant chunks), context_recall=0.85 (finding most needed info), but faithfulness=0.55. Diagnosis: the LLM is hallucinating despite having the right context. Fix: strengthen the system prompt ("Answer ONLY based on the provided context"), lower temperature to 0, consider switching to a more instruction-following model.

## Follow-Up Questions
### Q1: "How do you build a ground truth evaluation dataset?"
**Answer:** Three approaches: (1) **Manual curation** — domain experts write 50-100 (question, ideal_answer) pairs from your actual documents. Time-consuming but highest quality. (2) **Synthetic generation** — use an LLM to generate questions from your documents, then have humans verify. 10x faster. (3) **Production sampling** — log real user queries, have humans annotate which retrieved chunks were relevant and what the correct answer was. Best for ongoing evaluation.

### Q2: "How reliable is LLM-as-judge for evaluation?"
**Answer:** ~80-85% agreement with human judges for faithfulness and relevance. Known limitations: (1) position bias — the judge LLM may prefer the first option presented, (2) verbosity bias — longer answers get higher scores, (3) self-preference — GPT-4 rates GPT-4 outputs higher. Mitigations: randomize position, use multiple judge models, calibrate against human annotations. For critical applications, supplement with human evaluation.

### Q3: "How often should you evaluate a RAG system?"
**Answer:** Three cadences: (1) **Development** — evaluate every code change (embedding model, chunking, prompt). Automated CI/CD pipeline. (2) **Weekly** — run evaluation on a held-out test set to catch drift. (3) **Continuous** — monitor production metrics (user feedback, answer length, retrieval latency). Set alerts for metrics dropping below thresholds. The biggest risk is silent degradation — the system gets worse but no one notices.

---

<p align="center"><a href="../README.md">← Back to all questions</a></p>
