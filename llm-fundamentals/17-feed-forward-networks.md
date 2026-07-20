# Part 18: Feed-Forward Networks in LLMs

> **The Question:** "What are Feed-Forward Networks in LLMs?"

---

## What They're Actually Testing

The interviewer wants you to explain the FFN's role beyond "adds non-linearity." They want to hear that FFN layers function as **knowledge stores** and how the expansion-compression pattern works.

---

## The Technical Breakdown

### What Is the FFN in a Transformer?

Each Transformer block contains a **position-wise feed-forward network** — two linear layers with a non-linear activation applied independently to each token position:

```python
FFN(x) = W₂ · activation(W₁ · x + b₁) + b₂

# Dimensions:
# W₁: [d_model, d_ff]     → expansion (e.g., 4096 → 16384)
# W₂: [d_ff, d_model]     → compression (e.g., 16384 → 4096)
```

The FFN **expands** to a larger hidden dimension (typically 4× d_model), applies a non-linearity, then **compresses** back. This expansion creates a high-dimensional space for learning complex transformations.

### FFN as Knowledge Storage

Research (Geva et al., 2021) shows FFN layers function as **key-value memories**:
- **W₁** rows act as **keys** — pattern matchers for specific input features
- **W₂** columns act as **values** — the associated output information
- The activation function acts as a **gate** — only "matching" patterns pass through

```
Input: "The capital of France is ___"
W₁ detects: "capital of France" pattern → high activation in specific neuron
W₂ maps: that neuron's output → probability boost for "Paris"
```

### Modern FFN Variants

| Variant | Formula | Activation | Used By |
|---------|---------|-----------|---------|
| **Original** | W₂ · ReLU(W₁x) | ReLU | Original Transformer |
| **GELU** | W₂ · GELU(W₁x) | GELU | GPT-2, BERT |
| **SwiGLU** | W₂ · (Swish(W₁x) ⊙ W₃x) | Swish + Gate | LLaMA, PaLM, Mistral |

**SwiGLU** adds a **gating mechanism** — a third projection W₃ that controls which neurons pass information. This improves performance at the cost of one additional parameter matrix.

### Attention vs. FFN: Division of Labor

| Component | Role | Analogy |
|-----------|------|---------|
| **Attention** | Routes information between tokens | The **routing network** — decides which tokens talk to which |
| **FFN** | Transforms individual token representations | The **processing unit** — applies knowledge and transformations |

A token might attend to "France" and "capital" (attention), then the FFN transforms the combined representation into one that encodes "this context points to Paris" (knowledge retrieval).

---

## Accuracy Check

| Claim | Verified? |
|-------|-----------|
| FFN expands to 4× d_model then compresses back | ✅ Vaswani et al. (2017); standard in GPT, LLaMA |
| FFN layers function as key-value memories | ✅ Geva et al. (2021), "Transformer Feed-Forward Layers Are Key-Value Memories" |
| SwiGLU outperforms ReLU/GELU in FFN | ✅ Shazeer (2020); adopted by LLaMA, PaLM |
| Attention routes, FFN processes | ✅ Supported by ablation studies and probing experiments |

---

## Scenario Examples

### Scenario 1: Knowledge Editing

Researchers want to update the model's knowledge that "The president of the US is Biden" → "The president of the US is [new president]" without retraining. They locate the specific FFN neurons that fire for "president of the US" queries and directly edit the W₂ weights for those neurons to output the new answer. This technique (ROME, MEMIT) works because FFN layers literally store factual associations in their weight matrices.

### Scenario 2: Model Compression

When compressing a 70B model, the team analyzes FFN activation patterns and discovers that 40% of neurons in the FFN layers rarely activate (< 0.1% of inputs). They prune these neurons (structured pruning), reducing FFN parameters by 40% with only 2% quality drop. The FFN's over-parameterized expansion dimension provides natural redundancy that enables aggressive compression.

---

## Follow-Up Questions

### Q1: "Why is the FFN applied independently to each position?"

**Answer:** The FFN is "position-wise" — the same weights are applied to every token independently. Cross-token interaction is handled by attention. This separation is by design: attention handles **what information to gather** from other tokens, and the FFN handles **how to transform** each token's representation using that gathered information. Making FFN position-wise also enables parallelism — every position can be computed simultaneously.

### Q2: "What happens if you remove the FFN entirely?"

**Answer:** Without FFN, the Transformer becomes a series of linear mixing operations (attention is essentially a weighted average). The model loses its ability to store factual knowledge and perform non-linear transformations. Experiments show removing FFN layers causes massive drops in factual recall (knowledge stored in FFN weights is lost) while syntactic understanding (handled more by attention) is partially preserved.

### Q3: "Why does SwiGLU use three matrices instead of two?"

**Answer:** SwiGLU splits the original W₁ into two matrices — one produces the main activations, the other produces a **gate**. The gate controls which neurons pass information: `output = Swish(W₁x) ⊙ (W₃x)`. The element-wise multiplication (⊙) means the gate can shut off noisy or irrelevant neurons, producing sparser, more selective activations. This improves the signal-to-noise ratio of the FFN output. The third matrix adds ~50% more parameters to the FFN, but the quality improvement justifies the cost.

---

<p align="center">
  <a href="../README.md">← Back to all questions</a>
</p>
