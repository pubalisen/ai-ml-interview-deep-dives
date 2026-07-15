# Part 42: Recursive Language Models (RLMs)

> **The Question:** "What are Recursive Language Models (RLMs)?"

---

## What They're Actually Testing

They want to see if you're current on emerging architectures that go beyond flat autoregressive generation — models that can plan, revise, and self-correct.

---

## The Technical Breakdown

### What Are RLMs?

Recursive Language Models generate text through **iterative refinement** rather than single-pass left-to-right generation. The model generates an initial draft, then recursively improves it through multiple passes:

```
Pass 1: Generate initial draft
Pass 2: Critique the draft → identify weaknesses
Pass 3: Revise based on critique
Pass 4: Repeat until quality threshold or max iterations
```

This mimics how humans write — draft, review, revise — rather than producing a final answer in one shot.

### Key Approaches

| Approach | Mechanism | Example |
|----------|-----------|---------|
| **Self-refine** | Generate → Critique → Revise loop | Madaan et al. (2023) |
| **Iterative revision** | Multiple decoding passes with feedback | REST (Reinforced Self-Training) |
| **Recursive decomposition** | Break complex problems into sub-problems | Least-to-most prompting |
| **Tree search** | Explore multiple solution paths | Tree-of-thought |

### Why This Matters

Standard autoregressive models commit to each token irrevocably. RLMs address the fundamental limitation that LLMs can't "look ahead" or "go back" — by building revision into the generation process itself.

---

## Accuracy Check

| Claim | Verified? |
|-------|-----------|
| Self-refine uses generate-critique-revise loops | ✅ Madaan et al. (2023) |
| Iterative refinement improves output quality | ✅ Demonstrated across coding, math, and writing tasks |
| Standard autoregressive models can't self-correct without explicit prompting | ✅ Huang et al. (2024), "Large Language Models Cannot Self-Correct Reasoning Yet" |

---

## Scenario Examples

### Scenario 1: A code generation system uses recursive refinement: generate code → run tests → if tests fail, feed errors back to the model → regenerate. After 3 iterations, pass rate improves from 45% to 72%.

### Scenario 2: An essay writing assistant generates a first draft, then uses a separate "critic" prompt to identify weak arguments, then revises. The final output has 40% higher human preference ratings than single-pass generation.

---

## Follow-Up Questions

### Q1: "Don't recursive passes multiply latency and cost?"

**Answer:** Yes — 3 passes = ~3× the latency and tokens. This trade-off makes RLMs suitable for offline/batch tasks (document generation, code reviews) but impractical for real-time chat where sub-second responses are needed. The value proposition is quality over speed.

### Q2: "How is this different from chain-of-thought?"

**Answer:** CoT generates reasoning tokens in a single forward pass — it extends the generation but doesn't revise it. RLMs generate, evaluate, and then modify previous output. CoT is "think harder in one pass." RLMs are "write a draft and improve it."

### Q3: "Can models truly self-correct without external feedback?"

**Answer:** Research is mixed. Huang et al. (2024) showed that LLMs often can't improve their own reasoning without external signals (test results, retrieval, human feedback). Self-refinement works best when there's a verifiable quality signal — code that can be tested, math that can be checked, facts that can be retrieved.

---

<p align="center">
  <a href="../../README.md">← Back to all questions</a>
</p>
