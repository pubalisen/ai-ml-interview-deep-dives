# Part 30: Dense vs. Sparse Models

> **The Question:** "What is the difference between dense and sparse models?"

---

## What They're Actually Testing

Quick conceptual check — whether you can articulate the activation pattern difference and its implications for compute and memory.

---

## The Technical Breakdown

| Property | Dense Model | Sparse Model (MoE) |
|----------|------------|-------------------|
| **Activation** | ALL parameters active for every token | Only a SUBSET active per token |
| **Compute per token** | Proportional to total parameters | Proportional to active parameters (much less) |
| **Memory** | Load all parameters | Still load all parameters (all experts in VRAM) |
| **Examples** | LLaMA 3 70B, GPT-4 (if dense) | Mixtral 8×7B, DeepSeek-V2, GPT-4 (rumored MoE) |
| **Scaling** | More params = proportionally more compute | More params ≠ proportionally more compute |
| **Training** | Simpler — standard backprop | Complex — load balancing, routing stability |

### Key Insight

Sparse models **decouple** model capacity (total parameters = knowledge storage) from inference cost (active parameters = compute per token). A 47B-parameter MoE model can match a dense 70B while using only ~13B worth of compute per token.

---

## Accuracy Check

| Claim | Verified? |
|-------|-----------|
| Dense models activate all parameters | ✅ Standard MLP/Transformer architecture |
| Sparse models activate a subset via routing | ✅ Shazeer et al. (2017), MoE literature |
| Sparse models decouple capacity from compute | ✅ Core MoE value proposition |

---

## Scenario Examples

### Scenario 1: Cost Optimization

A team runs a dense 70B model at $50K/month (4× A100s). They switch to Mixtral 8×7B (sparse, 47B total, ~13B active). Inference speed improves 3× because each token only uses 2/8 experts. They now need only 2× A100s, cutting costs to $25K/month while maintaining comparable quality.

### Scenario 2: Edge vs. Cloud

For edge deployment (laptop/phone), a dense 3B model is the ceiling — all parameters must be active. For cloud deployment, a sparse 47B model works because you have enough GPU memory for all experts but only pay the compute cost of the active subset. Sparse models are a cloud optimization; dense models are the only option for resource-constrained edges.

---

## Follow-Up Questions

### Q1: "Can you make a dense model sparse after training?"

**Answer:** Sort of — **pruning** removes low-magnitude weights from a dense model, creating structured or unstructured sparsity. But this is different from MoE's learned routing. Pruning removes weights permanently; MoE dynamically selects different subsets per input. Post-hoc MoE conversion (splitting a dense FFN into experts) has been explored but doesn't match training-time MoE quality.

### Q2: "Are sparse models always better?"

**Answer:** No. Sparse models are better for large-scale serving (compute savings) but worse for: (1) small batch sizes (routing overhead dominates), (2) memory-constrained environments (all experts in memory), (3) training stability (load balancing is hard). Dense models are simpler, more predictable, and better understood.

### Q3: "What is unstructured sparsity vs. structured sparsity?"

**Answer:** **Unstructured**: individual weights are zeroed out randomly — requires specialized sparse compute kernels. **Structured**: entire neurons, heads, or layers are removed — works with standard hardware. MoE is a form of structured sparsity at the FFN level. Unstructured sparsity achieves higher compression ratios but has limited hardware support.

---

<p align="center">
  <a href="../README.md">← Back to all questions</a>
</p>
