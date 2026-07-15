# Part 13: Self-Attention

> **The Question:** "What is self-attention, and how does it work in Transformers?"

---

## What They're Actually Testing

The interviewer wants you to distinguish self-attention from other attention types, explain the computation end-to-end, and articulate why it's the core innovation that made Transformers work.

---

## The Technical Breakdown

### What Is Self-Attention?

Self-attention is a mechanism where each token in a sequence computes a **weighted relationship with every other token in the same sequence**. The "self" means Q, K, and V all come from the same input — the model attends to itself.

```
Input:  "The cat sat on the mat"
For each token → compute attention scores with ALL other tokens
                → aggregate information based on those scores
```

### The Full Computation

```python
# Input: X of shape [batch, seq_len, d_model]

# 1. Project to Q, K, V
Q = X @ W_Q  # [batch, seq_len, d_k]
K = X @ W_K  # [batch, seq_len, d_k]
V = X @ W_V  # [batch, seq_len, d_v]

# 2. Compute attention scores
scores = Q @ K.transpose(-2, -1)  # [batch, seq_len, seq_len]
scores = scores / math.sqrt(d_k)

# 3. Apply causal mask (for decoder-only)
scores = scores.masked_fill(mask == 0, float('-inf'))

# 4. Softmax → attention weights
attn_weights = softmax(scores, dim=-1)  # each row sums to 1

# 5. Weighted aggregation
output = attn_weights @ V  # [batch, seq_len, d_v]
```

### What the Attention Matrix Looks Like

For "The cat sat":
```
         The   cat   sat
The   [ 0.60  0.25  0.15 ]   ← "The" attends mostly to itself
cat   [ 0.10  0.50  0.40 ]   ← "cat" attends to itself and "sat"
sat   [ 0.05  0.70  0.25 ]   ← "sat" attends heavily to "cat" (its subject)
```

Each row sums to 1 (probability distribution). Each entry tells you how much one token "pays attention to" another.

### Why Self-Attention Beat RNNs

| Property | RNN/LSTM | Self-Attention |
|----------|----------|----------------|
| **Long-range dependencies** | Must propagate through O(n) steps | Direct connection in O(1) |
| **Parallelization** | Sequential — token n waits for token n-1 | Fully parallel — all tokens computed simultaneously |
| **Path length** | O(n) between distant tokens | O(1) between any two tokens |
| **Gradient flow** | Vanishing/exploding gradients over long sequences | Direct gradient path between all token pairs |
| **Compute** | O(n·d²) | O(n²·d) — quadratic in sequence length |

The trade-off: self-attention is **O(n²)** in sequence length (every token attends to every other), which limits context length. But the **parallelism** and **direct long-range connections** far outweigh this cost for modern hardware.

---

## Accuracy Check

| Claim | Verified? |
|-------|-----------|
| Self-attention computes pairwise relationships within the same sequence | ✅ Vaswani et al. (2017) |
| Each row of the attention matrix sums to 1 via softmax | ✅ Softmax normalization property |
| Self-attention provides O(1) path length between any two tokens | ✅ Vaswani et al. (2017), Table 1 |
| RNNs require O(n) sequential steps | ✅ Fundamental RNN architecture constraint |
| Self-attention compute is O(n²·d) | ✅ Vaswani et al. (2017), Section 4 |

---

## Scenario Examples

### Scenario 1: Resolving Ambiguity

In "The bank by the river was steep," self-attention determines that "bank" means "riverbank" (not "financial bank") by computing high attention weights between "bank" and "river." Without self-attention, a bag-of-words model couldn't disambiguate — it would see "bank" in isolation. The attention mechanism creates a **contextualized** representation of "bank" that incorporates information from "river."

### Scenario 2: Understanding Negation

In "The movie was not good," a simple embedding of "good" would encode positive sentiment. But self-attention allows "good" to attend to "not," producing a contextualized representation that captures the negation. The output embedding of "good" at the attention layer's output is dramatically different from the same word in "The movie was good" — because the attention weights shift to incorporate the negation signal.

---

## Follow-Up Questions

### Q1: "What's the difference between self-attention and cross-attention?"

**Answer:** In **self-attention**, Q, K, V all come from the same sequence — each token attends to tokens within its own sequence. In **cross-attention**, Q comes from one sequence (the decoder's current state) while K and V come from a different sequence (the encoder's output). Cross-attention is used in encoder-decoder architectures (T5, BART) to connect the encoder's representation to the decoder's generation. Decoder-only LLMs (GPT, LLaMA) use only self-attention with a causal mask.

### Q2: "How does the O(n²) complexity of self-attention become a practical bottleneck?"

**Answer:** For a sequence of n tokens, the attention matrix is n×n. At n=4096, that's 16M entries per head per layer. At n=128K (GPT-4's context), it's 16.4B entries. In FP16, that's ~32GB just for the attention matrix of a single layer with one head. This is why long-context models require innovations like **Flash Attention** (fused kernels that compute attention in tiles without materializing the full matrix), **sparse attention** (only attend to subset of tokens), and **linear attention** approximations.

### Q3: "Does every attention head learn the same patterns?"

**Answer:** No — different heads in the same layer specialize in different relationship types. Research (Voita et al., 2019; Clark et al., 2019) shows distinct head specializations: **positional heads** (attend to adjacent tokens), **syntactic heads** (attend to syntactic dependencies like subject-verb), **rare-word heads** (attend to infrequent but informative tokens), and **delimiter heads** (attend to punctuation and special tokens). This specialization is emergent — it's not programmed, but arises from the training objective. Some heads are even "dead" — contributing little and prunable without quality loss.

---

<p align="center">
  <a href="../../README.md">← Back to all questions</a>
</p>
