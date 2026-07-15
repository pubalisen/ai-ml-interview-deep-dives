# Part 5: The Transformer Architecture

> **The Question:** "What is the Transformer architecture and how does it work?"

---

## What They're Actually Testing

This is arguably the most important question in any ML interview. The interviewer wants to see if you can walk through the **data flow** of a Transformer — from input tokens to output probabilities — explaining each component mechanically. They're checking: Can you explain attention without hand-waving? Do you know why certain design choices were made?

---

## The Technical Breakdown

### The Big Picture

The Transformer (Vaswani et al., 2017) is a neural network architecture that processes sequences **in parallel** using **attention mechanisms** instead of the sequential processing used by RNNs and LSTMs.

```
Input tokens → Embeddings → [N × Transformer Blocks] → Output probabilities
```

Each Transformer block contains:
1. **Multi-Head Self-Attention** — lets each token attend to every other token
2. **Feed-Forward Network (FFN)** — applies non-linear transformations per position
3. **Layer Normalization** + **Residual Connections** — stabilize training

### Step-by-Step Data Flow

#### 1. Input Embedding + Positional Encoding

```
"The cat sat" → tokenize → [2, 1841, 2047]
                          → embed   → [[0.1, -0.3, ...], [0.5, 0.2, ...], [0.8, -0.1, ...]]
                          → + pos   → [[0.1+p0, -0.3+p0, ...], ...]
```

Each token is mapped to a **d-dimensional vector** (e.g., d=4096). Since attention processes all tokens in parallel (no inherent order), **positional encodings** are added so the model knows token positions.

#### 2. Self-Attention (The Core Innovation)

Self-attention allows each token to look at every other token and decide how much to "attend to" each one.

For each token, three vectors are computed via learned linear projections:
- **Query (Q)** — "What am I looking for?"
- **Key (K)** — "What do I contain?"
- **Value (V)** — "What information do I provide?"

The attention score between two tokens:

```
Attention(Q, K, V) = softmax(Q × Kᵀ / √dₖ) × V
```

| Step | Operation | Purpose |
|------|-----------|---------|
| `Q × Kᵀ` | Dot product of queries and keys | Measure relevance between token pairs |
| `/ √dₖ` | Scale by square root of key dimension | Prevent dot products from growing too large (which pushes softmax into saturation) |
| `softmax` | Normalize scores to probabilities | Attention weights sum to 1 |
| `× V` | Weighted sum of values | Aggregate information from attended tokens |

**Example:** In "The cat sat on the mat," when processing "sat," the attention mechanism might assign high weight to "cat" (subject of the verb) and lower weight to "the" (less informative).

#### 3. Multi-Head Attention

Instead of one attention computation, the model runs **h parallel attention heads** (e.g., h=32), each with its own Q, K, V projections:

```
head_i = Attention(Q_i, K_i, V_i)
MultiHead = Concat(head_1, ..., head_h) × W_O
```

Each head learns to attend to different types of relationships:
- Head 1 might track syntactic dependencies
- Head 2 might track coreference resolution
- Head 3 might track positional patterns

#### 4. Feed-Forward Network (FFN)

After attention, each token's representation passes through a **position-wise** FFN:

```
FFN(x) = W₂ · GELU(W₁ · x + b₁) + b₂
```

The FFN has a larger hidden dimension (typically 4× the model dimension). This is where the model stores **factual knowledge** — probing studies show that removing FFN layers causes the model to lose factual recall while retaining syntactic abilities.

#### 5. Residual Connections + Layer Norm

Each sub-layer (attention, FFN) uses:

```
output = LayerNorm(x + SubLayer(x))
```

- **Residual connections** allow gradients to flow directly through the network, enabling training of very deep models (96+ layers)
- **Layer normalization** stabilizes the distribution of activations across training

#### 6. Stacking Layers

A modern LLM stacks 32–128 identical Transformer blocks. Lower layers tend to capture syntactic features; higher layers capture semantic and reasoning patterns.

### Encoder vs. Decoder

| Variant | Attention Mask | Use Case | Examples |
|---------|---------------|----------|----------|
| **Encoder-only** | Bidirectional (sees all tokens) | Classification, embeddings | BERT, RoBERTa |
| **Decoder-only** | Causal (sees only past tokens) | Text generation | GPT, LLaMA, Claude |
| **Encoder-Decoder** | Encoder: bidirectional, Decoder: causal + cross-attention | Translation, summarization | T5, BART |

Modern LLMs are almost exclusively **decoder-only** because they're optimized for autoregressive generation.

---

## Accuracy Check

| Claim | Verified? |
|-------|-----------|
| Transformers use self-attention to process sequences in parallel | ✅ Vaswani et al., "Attention Is All You Need" (2017) |
| Scaling by √dₖ prevents softmax saturation | ✅ Vaswani et al. (2017), Section 3.2.1 |
| Multi-head attention learns diverse attention patterns | ✅ Clark et al., "What Does BERT Look At?" (2019) |
| FFN layers store factual knowledge | ✅ Geva et al., "Transformer Feed-Forward Layers Are Key-Value Memories" (2021) |
| Residual connections enable training of deep networks | ✅ He et al., "Deep Residual Learning" (2015) |
| Modern LLMs are predominantly decoder-only | ✅ GPT-4, Claude, LLaMA, Gemini, Mistral |

---

## Scenario Examples

### Scenario 1: Why Transformers Replaced RNNs

A team was building a machine translation system using LSTMs. Training took 3 weeks on 8 GPUs because RNNs process tokens **sequentially** — token 50 can't be processed until tokens 1-49 are done. They switched to a Transformer, which processes all tokens **in parallel** during training. Training time dropped to 3 days. The Transformer also captured long-range dependencies better because attention connects every token to every other token directly, while RNNs must propagate information through many sequential steps (leading to vanishing gradients).

### Scenario 2: Debugging Attention in Production

A customer support chatbot kept ignoring the customer's account number mentioned early in a long conversation. By visualizing the attention weights, the team discovered that tokens beyond position ~2000 in the context received near-zero attention from the final layers. The model's positional encoding was effectively limiting its useful context window. They switched to a model using **RoPE (Rotary Position Embedding)**, which better handles longer sequences, and the issue resolved.

---

## Follow-Up Questions

### Q1: "Why is attention O(n²) in sequence length, and why does it matter?"

**Answer:** For a sequence of n tokens, self-attention computes a dot product between **every pair** of tokens to produce the n×n attention matrix. This means both compute and memory scale quadratically: doubling the sequence length quadruples the cost. For a 4K context, the attention matrix is 4K×4K = 16M entries per head per layer. For 128K context, it's 128K×128K = 16.4 billion entries. This quadratic scaling is the fundamental bottleneck for long-context models and has driven innovations like Flash Attention (fused kernels, tiled computation), sparse attention (only attend to subsets), and linear attention approximations.

### Q2: "What would happen if you removed the positional encoding?"

**Answer:** Without positional encoding, the Transformer would be a **bag-of-words model** — it would have no way to distinguish "The cat sat on the mat" from "The mat sat on the cat." Attention computes pairwise relevance between tokens but has no inherent notion of order. Positional encodings inject position information into the token embeddings so the model can learn position-dependent patterns. In practice, removing positional encoding collapses translation BLEU scores by 10+ points and makes language generation incoherent.

### Q3: "What is the difference between pre-norm and post-norm Transformer blocks?"

**Answer:** In the original Transformer (**post-norm**), layer normalization is applied **after** the residual connection: `x + LayerNorm(SubLayer(x))`. In **pre-norm** (used by GPT-2+, LLaMA), normalization is applied **before** the sub-layer: `x + SubLayer(LayerNorm(x))`. Pre-norm makes training more stable for very deep models (96+ layers) because the residual stream has a clear gradient path. Post-norm can produce better final performance but requires careful learning rate warmup. Most modern LLMs use pre-norm with RMSNorm (a simplified variant of layer norm).

---

## Further Reading

- [Attention Is All You Need (Vaswani et al., 2017)](https://arxiv.org/abs/1706.03762)
- [Transformer Feed-Forward Layers Are Key-Value Memories (Geva et al., 2021)](https://arxiv.org/abs/2012.14913)
- [What Does BERT Look At? — Attention Analysis (Clark et al., 2019)](https://arxiv.org/abs/1906.04341)

---

<p align="center">
  <a href="../../README.md">← Back to all questions</a>
</p>
