# Part 19: Common Failure Modes in Prompting

> **The Question:** "What are the common failure modes in prompting, and how do you debug them?"

---

## The Technical Breakdown

### Failure Taxonomy

| Failure Mode | Symptom | Root Cause | Fix |
|-------------|---------|-----------|-----|
| **Hallucination** | Fabricates facts | No grounding data | Add RAG, citation requirements |
| **Instruction ignoring** | Doesn't follow format | Prompt too complex or ambiguous | Simplify, use structured output APIs |
| **Verbosity** | Overly long responses | RLHF bias toward length | Add length constraints, max_tokens |
| **Sycophancy** | Always agrees | Alignment training bias | Add "challenge assumptions" instruction |
| **Inconsistency** | Different answers for same input | Temperature > 0, prompt sensitivity | Lower temperature, use self-consistency |
| **Refusal** | Refuses benign requests | Over-aligned safety | Rephrase, add context justifying the request |
| **Off-topic drift** | Wanders from the question | Weak task framing | Stronger task constraints, output format |
| **Repetition** | Repeats phrases/paragraphs | Decoding loop | Frequency penalty, temperature > 0 |
| **Lost in the middle** | Ignores middle context | Attention bias | Put important info at start/end |

### Debugging Workflow

```
1. Identify: What specifically went wrong? (failure category)
2. Isolate: Does the failure occur on one input or many?
3. Hypothesis: What prompt element is causing the failure?
4. Test: Modify one element at a time and retest
5. Validate: Run on the full eval set (not just the failing case)
6. Deploy: Ship the fix, monitor for regressions
```

---

## Scenario Examples
### A: A legal bot hallucinating case citations. Debug: the prompt says "cite relevant cases" but provides no cases to cite. Fix: add RAG retrieval of real case law. The model was inventing citations to follow instructions — the fix is providing real data.
### B: An API returns JSON 95% of the time but occasionally returns prose. Debug: certain long, complex inputs cause the model to "forget" the format instruction. Fix: repeat the format instruction at the end of the prompt (recency bias) + add structured output enforcement.

## Follow-Up Questions
### Q1: "How do you distinguish prompt issues from model issues?"
**Answer:** Test the same prompt on a different model. If both fail similarly, it's a prompt issue. If only one fails, it's model-specific behavior. Also test with temperature=0 — if the failure is intermittent, it's a sampling issue; if consistent, it's a systematic prompt problem.

### Q2: "What tools help with prompt debugging?"
**Answer:** LangSmith (trace each step of a chain), Weights & Biases (log prompts and outputs), PromptLayer (version and compare prompts), custom logging (log the full prompt + response for every production request). Observability is the foundation of debugging.

### Q3: "How do you prevent prompt regression when updating?"
**Answer:** Maintain a regression test suite: 50-100 examples where the prompt is known to work correctly. Before deploying any prompt change, run the new prompt against this suite. Any accuracy drop > 1% requires investigation before shipping.

---

<p align="center"><a href="../../README.md">← Back to all questions</a></p>
