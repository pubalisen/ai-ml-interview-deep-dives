# Part 12: Prompt Engineering vs. Prompt Tuning

> **The Question:** "What is the difference between prompt engineering and prompt tuning?"

---

## The Technical Breakdown

| Property | Prompt Engineering | Prompt Tuning |
|----------|-------------------|---------------|
| **What changes** | Natural language text | Learned continuous vectors |
| **Requires training** | ❌ No | ✅ Yes (backpropagation) |
| **Model weights** | Frozen | Frozen (only prompt vectors train) |
| **Human-readable** | ✅ Yes | ❌ No (floating-point vectors) |
| **Requires labeled data** | ❌ No | ✅ Yes |
| **Iteration speed** | Minutes | Hours/days |
| **Performance ceiling** | Limited by prompt design | Higher (learned representations) |
| **Cost** | $0 (inference only) | $100-$1000 (training compute) |

### Prompt Tuning

Instead of writing text prompts, you prepend **learnable soft tokens** (continuous vectors) to the input. These vectors are optimized through backpropagation:

```python
# Hard prompt (prompt engineering):
"Classify this email as spam or not: {email}"

# Soft prompt (prompt tuning):
[v1, v2, v3, v4, v5] + tokenize("{email}")
# v1-v5 are learned 768-dimensional vectors, not words
```

### Variants

| Method | Trainable Parameters | Approach |
|--------|---------------------|----------|
| **Prompt tuning** | ~0.01% of model | Learned prefix tokens only |
| **P-tuning** | ~0.1% | Learned tokens + reparameterization |
| **Prefix tuning** | ~0.5% | Learned prefix for all layers |
| **LoRA** | ~1-5% | Low-rank weight updates (different category) |

---

## Scenario Examples
### A: A team can't share their proprietary prompt (contains business logic). Prompt tuning encodes the "prompt knowledge" into continuous vectors that are opaque — competitors can't reverse-engineer the instructions.
### B: On a medical classification task, prompt engineering achieves 82% accuracy. Prompt tuning on 500 labeled examples pushes accuracy to 91% — the learned soft tokens find representations that no human-written prompt could express.

## Follow-Up Questions
### Q1: "When should you choose prompt tuning over prompt engineering?"
**Answer:** Choose prompt tuning when: (1) you have labeled data, (2) prompt engineering has plateaued below your accuracy target, (3) you want IP protection (learned vectors are opaque), or (4) you need to serve multiple tasks from one model (each task gets its own soft prefix).

### Q2: "How does prompt tuning compare to full fine-tuning?"
**Answer:** Prompt tuning trains 0.01% of parameters vs. 100% for full fine-tuning. It's cheaper, faster, and preserves the base model's general capabilities. Full fine-tuning achieves higher accuracy but risks catastrophic forgetting and is 100-1000× more expensive.

### Q3: "Can you interpret learned soft tokens?"
**Answer:** Partially — you can find the nearest vocabulary token to each learned vector, but the results are often nonsensical (e.g., "the the about regarding"). The learned representations operate in a continuous space that doesn't map cleanly to natural language. This is both a feature (IP protection) and a limitation (no interpretability).

---

<p align="center"><a href="../../README.md">← Back to all questions</a></p>
