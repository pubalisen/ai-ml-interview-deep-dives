# Scenario 3: Teaching LLMs to Say "I Don't Know"

> **The Scenario:** "Your LLM does not admit when it does not know the answer. How do you make it say 'I don't know'?"

---

## Solutions

### 1. System Prompt Engineering
```
If the information needed to answer is not present in the provided context, 
respond with: "I don't have enough information to answer this question."
Never fabricate facts. Never guess.
```

### 2. Confidence-Based Gating
Use log-probabilities to detect low-confidence responses. If the model's average log-prob is below a threshold, replace the response with "I'm not confident enough to answer."

### 3. RAG with Retrieval Verification
When using RAG, if the retrieved documents have low relevance scores (cosine similarity < threshold), flag the response as potentially unsupported.

### 4. Calibration Fine-Tuning
Fine-tune the model on examples where the correct answer is "I don't know" — datasets with unanswerable questions (SQuAD 2.0 pattern).

---

## Accuracy Check
| Approach | Reliability | False "I don't know" rate |
|----------|------------|--------------------------|
| System prompt | ~70% | Low (under-triggers) |
| Log-prob gating | ~80% | Medium (threshold-dependent) |
| RAG verification | ~85% | Low |
| Fine-tuning | ~90% | Tunable |

---

## Scenario Examples
### A: A medical chatbot must never hallucinate diagnoses. RAG retrieval scores below 0.7 trigger: "I couldn't find reliable information. Please consult a healthcare professional."
### B: A customer support bot is asked about a competitor's product. The system prompt instructs: "Only answer questions about our products. For others, respond: 'I can only help with [Company] products.'"

---

## Follow-Up Questions
### Q1: "How do you distinguish 'I don't know' from 'the answer is complex'?"
**Answer:** Use structured evaluation: check if the query is within the model's domain (topic classifier) AND if supporting evidence exists (retrieval score). A complex but answerable question will have good retrieval results; a truly unknown question won't.

### Q2: "Can you train a separate classifier for answerability?"
**Answer:** Yes — train a lightweight classifier on (question, context) pairs labeled as answerable/unanswerable. This classifier runs before the LLM and gates whether to generate an answer. SQuAD 2.0 provides a good training signal for this.

### Q3: "What's the cost of false 'I don't know' responses?"
**Answer:** Depends on the domain. In healthcare: false "I don't know" is far better than a hallucinated answer. In customer support: too many "I don't know" responses frustrate users. Tune the confidence threshold based on the cost asymmetry of false positives vs false negatives.

---

<p align="center">
  <a href="../README.md">← Back to all questions</a>
</p>
