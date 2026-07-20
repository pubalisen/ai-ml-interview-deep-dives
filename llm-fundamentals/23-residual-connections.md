# Part 24: Skip Connections (Residual Connections)

> **The Question:** "What are skip connections (residual connections) in Transformers?"

---

## What They're Actually Testing

They want you to explain why deep networks need residual connections and how they enable gradient flow through 100+ Transformer layers.

---

## The Technical Breakdown

### What Are Residual Connections?

A residual connection adds the input of a sub-layer directly to its output:

```python
output = x + SubLayer(x)    # NOT: output = SubLayer(x)
```

In a Transformer block:
```python
# Attention sub-layer
x = x + Attention(LayerNorm(x))

# FFN sub-layer
x = x + FFN(LayerNorm(x))
```

The input `x` **bypasses** the transformation and is added back to the result. This creates a "highway" for information and gradients to flow through the entire network.

### Why They're Essential

Without residual connections in a 96-layer Transformer:

```
Layer 1 output → Layer 2 → ... → Layer 96 → Output
  (each layer multiplies by weight matrices)

Gradient at Layer 1 = ∂Loss/∂Layer96 × ∂Layer96/∂Layer95 × ... × ∂Layer2/∂Layer1
                    = product of 95 Jacobian matrices
                    → vanishes (all < 1) or explodes (any > 1)
```

With residual connections:

```
∂output/∂input = 1 + ∂SubLayer/∂input

The "1" term means the gradient has a DIRECT path regardless of layer depth.
Even if ∂SubLayer/∂input → 0, the gradient through the skip connection is 1.
```

### The Residual Stream View

Modern mechanistic interpretability views the Transformer as a **residual stream** — a persistent vector that attention and FFN layers **read from and write to**:

```
x₀ (initial embedding)
  → x₁ = x₀ + Attn₁(x₀) + FFN₁(x₀)     # Layer 1 adds to the stream
  → x₂ = x₁ + Attn₂(x₁) + FFN₂(x₁)     # Layer 2 adds to the stream
  → ...
  → x_N = accumulated information from all layers
```

Each layer makes a **small contribution** to the residual stream. The final representation is the sum of contributions from all layers plus the original embedding.

---

## Accuracy Check

| Claim | Verified? |
|-------|-----------|
| Residual connections add input directly to sub-layer output | ✅ He et al. (2015), Vaswani et al. (2017) |
| They prevent vanishing gradients in deep networks | ✅ He et al. (2015), "Deep Residual Learning for Image Recognition" |
| The residual stream perspective is used in interpretability | ✅ Elhage et al. (2021), Anthropic's "A Mathematical Framework for Transformer Circuits" |

---

## Scenario Examples

### Scenario 1: Training a Very Deep Model

A team tries to train a 128-layer Transformer without residual connections. Loss never decreases — gradients vanish before reaching the first layers. Adding residual connections (the standard `x + f(x)` pattern) immediately fixes training. The model converges in the expected number of steps because gradients can now reach all layers through the skip connections.

### Scenario 2: Mechanistic Interpretability

A researcher wants to understand what information each layer contributes. By examining the **residual stream** at each layer, they can measure each layer's contribution vector and determine that Layer 5 adds syntactic information, Layer 20 adds factual knowledge, and Layer 60 adds reasoning patterns. This is only possible because of the additive structure of residual connections — each layer's contribution is separable.

---

## Follow-Up Questions

### Q1: "What happens if the residual connection scale is wrong?"

**Answer:** If `output = x + α × SubLayer(x)` and α is too large, the sub-layer contributions dominate and the network behaves as if there's no skip connection (training instability). If α is too small, the sub-layer contributions are ignored and the model can't learn. Some architectures use **learnable scaling** or **layer-scale** (initializing α small and learning it). DeepNet (Wang et al., 2022) proposes specific initialization schemes to stabilize very deep Transformers.

### Q2: "How do residual connections interact with layer normalization?"

**Answer:** Pre-norm: `x + SubLayer(LayerNorm(x))` — normalizes before the sub-layer, keeping the residual stream unnormalized. This is more stable for deep models. Post-norm: `LayerNorm(x + SubLayer(x))` — normalizes after adding, which can produce better final quality but is harder to train. Most modern LLMs use pre-norm with RMSNorm.

### Q3: "Are residual connections unique to Transformers?"

**Answer:** No — they were introduced by He et al. (2015) for CNNs (ResNets) and are used across deep learning: ResNets (vision), DenseNets (dense connections), U-Nets (skip connections across encoder-decoder), and all modern Transformers. They're a fundamental technique for training any very deep network, regardless of architecture.

---

<p align="center">
  <a href="../README.md">← Back to all questions</a>
</p>
