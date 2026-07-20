# Part 15: The √dₖ Scaling Factor

> **The Question:** "Why do we scale the dot product attention by √dₖ in the Transformer architecture?"

---

## What They're Actually Testing

This is a math-flavored question. The interviewer wants you to explain the **statistical reasoning** — that dot products grow with dimension, which pushes softmax into saturation, killing gradients.

---

## The Technical Breakdown

### The Problem Without Scaling

When Q and K vectors have dimension dₖ, the dot product Q·K is the **sum of dₖ terms**:

```
Q·K = q₁k₁ + q₂k₂ + ... + q_dₖ k_dₖ
```

If each qᵢ and kᵢ are independent with mean 0 and variance 1:
- **Mean** of Q·K = 0
- **Variance** of Q·K = dₖ

As dₖ increases (e.g., dₖ = 128), the dot products become very large in magnitude. When these large values are passed to softmax:

```
softmax([100, 1, 1, 1]) ≈ [1.0, 0.0, 0.0, 0.0]  ← almost one-hot
softmax([2, 1, 1, 1])   ≈ [0.39, 0.20, 0.20, 0.20]  ← smooth distribution
```

Large dot products → softmax outputs near 0 or 1 → **near-zero gradients** → the model can't learn.

### The Fix

Dividing by √dₖ rescales the variance back to 1:

```
Var(Q·K / √dₖ) = Var(Q·K) / dₖ = dₖ / dₖ = 1
```

This keeps the softmax inputs in a range where gradients are healthy.

### Why √dₖ Specifically?

It's the standard deviation (√variance). Since Var(Q·K) = dₖ, the standard deviation is √dₖ. Dividing by the standard deviation normalizes the distribution to unit variance — a standard statistical normalization.

---

## Accuracy Check

| Claim | Verified? |
|-------|-----------|
| Dot product variance grows with dₖ | ✅ Elementary statistics: variance of sum of independent products |
| Large softmax inputs cause near-zero gradients | ✅ Vaswani et al. (2017), Section 3.2.1, footnote |
| Dividing by √dₖ normalizes variance to 1 | ✅ Mathematical proof in original paper |

---

## Scenario Examples

### Scenario 1: Training Instability

A team implements attention from scratch but forgets the scaling factor. With dₖ = 128, dot products reach magnitudes of 50-100. The softmax produces near-one-hot distributions — the model always attends to just one token. Gradients vanish, loss plateaus, and training fails. Adding `/ math.sqrt(d_k)` immediately fixes training.

### Scenario 2: Custom Attention Variants

A researcher designs a new attention variant using **additive** attention instead of dot-product. Since additive attention doesn't have the same variance growth problem (it uses a learned vector, not dot products), they don't need √dₖ scaling. Understanding the **reason** for scaling (variance control) lets you decide when it applies and when it doesn't.

---

## Follow-Up Questions

### Q1: "What happens if you use a different scaling factor?"

**Answer:** Using a smaller factor (e.g., √(dₖ/2)) keeps larger variance → softmax is sharper (more focused attention). Using a larger factor makes softer attention (more diffuse). Some architectures treat the scaling factor as a **learnable parameter** per head, letting the model decide its own attention sharpness. The √dₖ choice is a reasonable default that works well, but it's not the only valid option.

### Q2: "Does Flash Attention still use this scaling?"

**Answer:** Yes — Flash Attention is an **implementation optimization**, not an algorithmic change. It computes the exact same scaled dot-product attention but does so in a memory-efficient way using tiled computation and kernel fusion. The scaling by √dₖ is preserved because it's mathematically necessary for training stability, regardless of how the computation is implemented.

### Q3: "Is the scaling factor related to temperature in sampling?"

**Answer:** Conceptually yes — both control the "sharpness" of a probability distribution. Temperature divides logits before the final softmax at generation time. The √dₖ scaling divides attention scores before softmax in every attention layer. Both prevent softmax saturation. You could view the scaling factor as "attention temperature" — lower scaling = sharper attention (like low temperature), higher scaling = softer attention (like high temperature).

---

<p align="center">
  <a href="../README.md">← Back to all questions</a>
</p>
