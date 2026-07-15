# Part 6: Key Components of the Transformer

> **The Question:** "What are the key components of the Transformer architecture?"

---

## What They're Actually Testing

This is the "name the parts and explain what each does" version of the Transformer question. The interviewer wants a **structured enumeration** — not a deep dive into one component, but proof that you can identify and explain every building block and how they fit together.

---

## The Technical Breakdown

### The Component Map

```
┌──────────────────────────────────┐
│        TRANSFORMER BLOCK         │
│  ┌─────────────────────────────┐ │
│  │  Multi-Head Self-Attention  │ │  ← Learns token relationships
│  │  + Residual + LayerNorm     │ │  ← Gradient flow + stability
│  ├─────────────────────────────┤ │
│  │  Feed-Forward Network       │ │  ← Stores knowledge + transforms
│  │  + Residual + LayerNorm     │ │  ← Gradient flow + stability
│  └─────────────────────────────┘ │
│           × N layers             │
└──────────────────────────────────┘
          ▲                  │
    Token Embeddings    Output Head
    + Positional        (Linear + Softmax)
    Encoding
```

### Component-by-Component

| # | Component | What It Does | Why It's Needed |
|---|-----------|-------------|-----------------|
| 1 | **Token Embedding** | Maps discrete token IDs to dense vectors of dimension d | Neural networks operate on continuous vectors, not integers |
| 2 | **Positional Encoding** | Adds position information to embeddings | Attention is permutation-invariant — without position info, "cat sat" = "sat cat" |
| 3 | **Multi-Head Self-Attention** | Each token computes weighted relationships with all other tokens | Captures context, dependencies, and relationships across the sequence |
| 4 | **Scaled Dot-Product** | Q·Kᵀ / √dₖ computes attention scores | Measures pairwise relevance; scaling prevents gradient vanishing in softmax |
| 5 | **Feed-Forward Network** | Two linear layers with activation (GELU/SwiGLU) per position | Adds non-linearity; acts as a knowledge store for facts and patterns |
| 6 | **Layer Normalization** | Normalizes activations across the feature dimension | Stabilizes training, prevents internal covariate shift |
| 7 | **Residual Connections** | Adds input directly to sub-layer output: x + f(x) | Enables gradient flow through deep networks (32-128 layers) |
| 8 | **Output Head** | Linear projection to vocabulary size + softmax | Converts final hidden states to probability distribution over tokens |
| 9 | **Causal Mask** (decoder-only) | Masks future tokens so position i can only attend to positions ≤ i | Enforces autoregressive property — can't "look ahead" during generation |

### Modern Additions (Post-2017)

| Component | What Changed | Used By |
|-----------|-------------|---------|
| **RMSNorm** | Replaces LayerNorm — simpler, faster (no mean subtraction) | LLaMA, Mistral |
| **RoPE** | Rotary position embedding replaces fixed sinusoidal encoding | LLaMA, Mistral, Qwen |
| **SwiGLU** | Gated activation in FFN replaces ReLU/GELU — better performance | LLaMA 2/3, PaLM |
| **GQA** | Grouped-Query Attention — fewer KV heads, faster inference | LLaMA 2 70B, Mistral |
| **KV Cache** | Caches key/value matrices from previous tokens during generation | All production LLMs |

---

## Accuracy Check

| Claim | Verified? |
|-------|-----------|
| Transformers have 9 core components as listed | ✅ Vaswani et al. (2017) |
| RMSNorm replaces LayerNorm in modern LLMs | ✅ Zhang & Sennrich (2019); LLaMA (Touvron et al., 2023) |
| SwiGLU improves FFN performance | ✅ Shazeer (2020); adopted in PaLM, LLaMA |
| GQA reduces KV cache memory | ✅ Ainslie et al. (2023); used in LLaMA 2 70B |
| RoPE handles longer contexts better than sinusoidal | ✅ Su et al. (2021); adopted across industry |

---

## Scenario Examples

### Scenario 1: Optimizing Inference Memory

A team is serving a 13B model and running out of GPU memory during long-context inference. By analyzing the components, they identify the **KV cache** as the primary memory consumer — it stores key/value matrices for all past tokens across all layers and heads. For a 4096-token context with 40 layers and 40 heads, the KV cache alone consumes ~10GB in FP16. They switch to **Grouped-Query Attention (GQA)**, which shares KV heads across multiple query heads (e.g., 8 KV heads instead of 40), reducing KV cache memory by 5× without significant quality loss.

### Scenario 2: Debugging Gradient Instability

During fine-tuning of a 7B model, the team sees loss spikes and NaN values. They suspect the **layer normalization** placement. The model uses post-norm (LayerNorm after residual addition), which can cause instability in deep models. Switching to **pre-norm** (LayerNorm before the attention/FFN sub-layer) + using **RMSNorm** instead of full LayerNorm stabilizes training because the residual stream is no longer passed through normalization, maintaining a cleaner gradient path.

---

## Follow-Up Questions

### Q1: "Why does the FFN have a larger hidden dimension than the model dimension?"

**Answer:** The FFN typically expands the hidden dimension to 4× the model dimension (e.g., d_model=4096, d_ff=16384) before projecting back down. This expansion creates a **higher-dimensional space** where the model can learn more complex, non-linear transformations. Research (Geva et al., 2021) shows that FFN layers function as **key-value memories** — the first projection acts as keys matching input patterns, and the second projection acts as values storing associated information. The larger hidden dimension gives the model more "memory slots" to store facts and learned patterns.

### Q2: "What's the difference between self-attention and cross-attention?"

**Answer:** In **self-attention**, Q, K, and V all come from the **same sequence** — each token attends to all tokens in its own sequence. In **cross-attention**, Q comes from one sequence (e.g., the decoder) while K and V come from a **different sequence** (e.g., the encoder output). Cross-attention is how encoder-decoder models (like T5) connect the encoder's understanding of the input to the decoder's generation of the output. Decoder-only LLMs (GPT, LLaMA) don't use cross-attention — they only use causal self-attention.

### Q3: "If you had to remove one component to reduce model size, which would you remove and why?"

**Answer:** I'd reduce the **number of attention heads via GQA** rather than removing a structural component entirely. GQA shares key-value heads across multiple query heads — for example, using 8 KV heads instead of 32 for 32 query heads. This reduces the KV projection parameters and the KV cache memory at inference time by ~4×, with minimal quality degradation (< 1% on benchmarks). Removing entire components (like the FFN or attention) would break the architecture's fundamental ability to learn. GQA is a surgical optimization that preserves capability while reducing cost.

---

## Further Reading

- [Attention Is All You Need (Vaswani et al., 2017)](https://arxiv.org/abs/1706.03762)
- [GLU Variants Improve Transformer — SwiGLU (Shazeer, 2020)](https://arxiv.org/abs/2002.05202)
- [GQA: Training Generalized Multi-Query Transformer Models (Ainslie et al., 2023)](https://arxiv.org/abs/2305.13245)

---

<p align="center">
  <a href="../../README.md">← Back to all questions</a>
</p>
