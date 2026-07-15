# Part 28: Model Distillation

> **The Question:** "What is model distillation, and how is it used with LLMs?"

---

## What They're Actually Testing

They want to know how a large "teacher" model's knowledge is transferred to a smaller "student" model, and the practical trade-offs in production.

---

## The Technical Breakdown

### What Is Distillation?

Knowledge distillation trains a smaller **student** model to mimic the behavior of a larger **teacher** model, rather than training from raw data alone.

```
Teacher (70B) → produces soft probability distributions
Student (7B)  → trained to match the teacher's distributions, not just hard labels
```

### Why Soft Distributions Matter

```
Hard label: "Paris" (correct answer)
Teacher's soft distribution: {Paris: 0.72, Lyon: 0.15, Marseille: 0.08, London: 0.03, Berlin: 0.02}
```

The soft distribution contains **richer information** — it tells the student that Lyon and Marseille are also French cities, that London is somewhat plausible, and that Berlin is less likely. This "dark knowledge" transfers nuanced understanding that hard labels can't convey.

### Distillation Loss

```python
loss = α × KL_divergence(student_probs, teacher_probs) + (1-α) × cross_entropy(student_probs, hard_labels)
```

The student optimizes a weighted combination of:
1. **Matching the teacher's soft distributions** (KL divergence)
2. **Predicting the correct answer** (standard cross-entropy)

### LLM-Specific Distillation Methods

| Method | Approach | Example |
|--------|----------|---------|
| **Output distillation** | Student matches teacher's token probabilities | DistilBERT |
| **Synthetic data** | Teacher generates training data, student trains on it | Alpaca (GPT-4 → LLaMA) |
| **Feature distillation** | Student matches teacher's intermediate representations | TinyBERT |
| **On-policy distillation** | Student generates, teacher scores/corrects | Used in RLHF pipelines |

---

## Accuracy Check

| Claim | Verified? |
|-------|-----------|
| Distillation transfers knowledge from teacher to student | ✅ Hinton et al. (2015), "Distilling the Knowledge in a Neural Network" |
| Soft distributions carry "dark knowledge" | ✅ Hinton et al. (2015), core contribution |
| Synthetic data distillation is widely used | ✅ Alpaca, Vicuna, Orca — all use GPT-4 outputs |
| Distilled models are smaller but retain significant capability | ✅ DistilBERT retains 97% of BERT's performance at 40% fewer parameters |

---

## Scenario Examples

### Scenario 1: Edge Deployment

A company needs an LLM on mobile devices with 4GB RAM. GPT-4 is obviously impossible. They use GPT-4 to generate 100K high-quality instruction-response pairs, then train a 1.5B model on this data. The student model captures 70% of GPT-4's quality on their specific domain while running locally on a phone in 200ms per response.

### Scenario 2: Cost Reduction

A production API serves 50M requests/day using GPT-4 at $150K/month. They distill GPT-4's outputs into a fine-tuned LLaMA 3 8B. For their use case (customer support), the distilled model achieves 92% of GPT-4's quality. Hosting cost: $5K/month. They save $145K/month with an 8% quality trade-off.

---

## Follow-Up Questions

### Q1: "Is it legal to distill from closed-source models like GPT-4?"

**Answer:** This is a legal gray area. OpenAI's terms of service prohibit using outputs to train competing models. The Alpaca and Vicuna projects (which distilled from GPT-3.5/4) acknowledged this is for research only. For production use, you should either use open-source teachers (LLaMA 3, Mistral) or generate synthetic data with proper licensing. Legal counsel is recommended.

### Q2: "What's the minimum student size that retains quality?"

**Answer:** It depends on task complexity. For narrow, well-defined tasks (classification, extraction), a 1-3B student can match a 70B teacher. For general-purpose reasoning, the quality gap grows: a 7B student retains ~70-80% of a 70B teacher's general capability. Below 1B parameters, quality drops sharply for complex tasks. The rule of thumb: distillation works best when the task is **narrow and well-defined**.

### Q3: "How does distillation differ from fine-tuning?"

**Answer:** Fine-tuning trains on (input, correct_answer) pairs — hard labels. Distillation trains on (input, teacher_output_distribution) — soft labels. Distillation transfers more information because the teacher's distribution over all tokens encodes nuanced relationships the hard label doesn't. You can combine both: first distill from a teacher, then fine-tune on task-specific labeled data.

---

<p align="center">
  <a href="../../README.md">← Back to all questions</a>
</p>
