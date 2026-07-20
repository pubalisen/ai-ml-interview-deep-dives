# Part 35: Layer Normalization

> **The Question:** "Explain Layer Normalization."

---

## What They're Actually Testing

They want you to explain how LayerNorm stabilizes training by normalizing activations, and how it differs from Batch Normalization.

---

## The Technical Breakdown

### What Is Layer Normalization?

LayerNorm normalizes activations **across the feature dimension** for each token independently:

```python
LayerNorm(x) = γ × (x - μ) / √(σ² + ε) + β

μ = mean(x)       # mean across feature dimension
σ² = var(x)       # variance across feature dimension
γ, β              # learnable scale and shift parameters
ε                 # small constant for numerical stability (e.g., 1e-5)
```

### LayerNorm vs BatchNorm

| Property | LayerNorm | BatchNorm |
|----------|----------|-----------|
| **Normalizes across** | Feature dimension (per token) | Batch dimension (across samples) |
| **Batch dependency** | ❌ Independent of batch | ✅ Requires batch statistics |
| **Works with batch=1** | ✅ Yes | ❌ No (undefined statistics) |
| **Sequence models** | ✅ Standard for Transformers | ❌ Problematic for variable-length sequences |

### Pre-Norm vs Post-Norm

**Post-norm** (original Transformer): `x = LayerNorm(x + SubLayer(x))`
**Pre-norm** (modern LLMs): `x = x + SubLayer(LayerNorm(x))`

Pre-norm is more stable for deep models because the residual stream bypasses normalization, maintaining cleaner gradient flow.

---

## Accuracy Check

| Claim | Verified? |
|-------|-----------|
| LayerNorm normalizes across feature dimension | ✅ Ba et al. (2016) |
| Pre-norm is more stable for deep models | ✅ Xiong et al. (2020), "On Layer Normalization in the Transformer Architecture" |
| Most modern LLMs use pre-norm | ✅ GPT-2+, LLaMA, Mistral |

---

## Scenario Examples

### Scenario 1: A 96-layer Transformer using post-norm fails to converge — loss oscillates and spikes. Switching to pre-norm stabilizes training immediately because the residual connection provides an unimpeded gradient path (normalization is before the sub-layer, not wrapping it).

### Scenario 2: During inference with batch_size=1, BatchNorm would fail (can't compute batch statistics). LayerNorm works perfectly because it normalizes within each token independently.

---

## Follow-Up Questions

### Q1: "Why do modern LLMs use RMSNorm instead of LayerNorm?"

**Answer:** RMSNorm removes the mean-centering step, computing only the root-mean-square normalization. This is ~10-15% faster with equivalent quality. See the RMSNorm deep-dive for details.

### Q2: "Where is LayerNorm placed in a Transformer block?"

**Answer:** In pre-norm: before each sub-layer (before attention, before FFN). In post-norm: after each sub-layer (after residual addition). A final LayerNorm is also applied to the last layer's output before the linear output head.

### Q3: "What happens without any normalization?"

**Answer:** Without normalization, activation magnitudes grow or shrink through layers, causing training instability. Gradients become too large or too small, learning rates must be set extremely carefully, and deep networks often fail to converge entirely.

---

<p align="center">
  <a href="../README.md">← Back to all questions</a>
</p>
