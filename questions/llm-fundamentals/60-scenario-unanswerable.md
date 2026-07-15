# Scenario 15: Detecting Unanswerable Questions

> **The Scenario:** "Your QA system always generates an answer even when no answer exists in the context. How do you detect unanswerable questions?"

---

## Solutions
1. **Retrieval confidence threshold** — If RAG retrieval scores are below a threshold (cosine similarity < 0.6), flag as unanswerable.
2. **Entailment verification** — After generating an answer, use an NLI (Natural Language Inference) model to check if the context *entails* the answer. If not, reject.
3. **Dual-prompt approach** — First ask "Is this question answerable from the given context? Yes/No." Then only generate an answer if "Yes."
4. **Abstention fine-tuning** — Fine-tune on SQuAD 2.0-style data that includes unanswerable questions labeled with "unanswerable."
5. **Evidence highlighting** — Require the model to cite the specific passage supporting its answer. If no passage is cited, the answer is likely hallucinated.

## Scenario Examples
### A: A legal QA system over contracts. User asks about a clause that doesn't exist. Retrieval returns low-relevance chunks (max similarity 0.45). The system responds: "I couldn't find information about this clause in the provided contract."
### B: A medical QA system generates "Take aspirin for headaches" when the context only discusses surgery. NLI verification detects the answer isn't supported by the context and triggers: "The provided context doesn't address medication recommendations."

## Follow-Up Questions
### Q1: "How do you set the confidence threshold?"
**Answer:** Calibrate using a validation set with known answerable/unanswerable questions. Plot precision-recall curve, choose threshold that maximizes F1 or meets your false-positive/false-negative cost trade-off.

### Q2: "Can the model self-assess answerability?"
**Answer:** Partially — prompting "Is this answerable?" works ~70-80% of the time. But models are biased toward answering (they were trained to be helpful). External verification (NLI, retrieval scores) is more reliable.

### Q3: "What about partially answerable questions?"
**Answer:** Generate what can be answered and explicitly state what cannot: "Based on the provided context, X is true. However, the context doesn't contain information about Y."

---

<p align="center"><a href="../../README.md">← Back to all questions</a></p>
