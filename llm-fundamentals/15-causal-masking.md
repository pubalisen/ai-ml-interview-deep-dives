# Part 16: Causal Masking

> **The Question:** "What is causal masking?"

---

## What They're Actually Testing

The interviewer wants you to explain why decoder-only models must prevent tokens from "seeing the future" and how the mask is implemented mechanically.

---

## The Technical Breakdown

### What Is Causal Masking?

Causal masking (also called autoregressive masking) is a technique that **prevents each token from attending to tokens that come after it** in the sequence. Position i can only attend to positions 0, 1, ..., i — never to i+1, i+2, etc.

```
                Attending TO →
            pos0  pos1  pos2  pos3
From pos0 [  ✅    ❌    ❌    ❌  ]
From pos1 [  ✅    ✅    ❌    ❌  ]
From pos2 [  ✅    ✅    ✅    ❌  ]
From pos3 [  ✅    ✅    ✅    ✅  ]
```

### Why It's Needed

During **training**, the model processes the entire sequence at once (for efficiency). But at **inference**, it generates one token at a time — it doesn't know future tokens. Without causal masking, the model would "cheat" during training by looking at the answer while predicting it, and then fail at inference when future tokens aren't available.

```python
# Implementation
mask = torch.triu(torch.ones(seq_len, seq_len), diagonal=1).bool()
# Upper triangular matrix = True for future positions

scores = Q @ K.T / math.sqrt(d_k)
scores.masked_fill_(mask, float('-inf'))  # Future positions → -inf
attn_weights = softmax(scores)  # -inf → 0 after softmax
```

Setting future positions to `-inf` before softmax ensures they get **zero attention weight** — the model cannot extract any information from future tokens.

### Causal vs. Bidirectional

| Model Type | Masking | What Each Token Sees | Example |
|-----------|---------|---------------------|---------|
| **Decoder-only** (GPT, LLaMA) | Causal mask | Only past + self | Text generation |
| **Encoder-only** (BERT) | No mask (bidirectional) | All tokens | Classification, embeddings |
| **Encoder-decoder** (T5) | Encoder: bidirectional; Decoder: causal | Encoder sees all; decoder sees past | Translation |

---

## Accuracy Check

| Claim | Verified? |
|-------|-----------|
| Causal mask prevents attending to future positions | ✅ Vaswani et al. (2017); GPT (Radford et al., 2018) |
| Implemented by setting future scores to -inf before softmax | ✅ Standard implementation in PyTorch, JAX |
| Without masking, training would leak future information | ✅ This would violate the autoregressive training objective |
| BERT uses bidirectional (no mask) | ✅ Devlin et al. (2019) |

---

## Scenario Examples

### Scenario 1: Training vs. Inference Consistency

During training on "The cat sat on the mat", when computing the loss for predicting "sat" (position 2), the model must NOT access "on", "the", or "mat" (positions 3-5). Causal masking ensures this. At inference, the model literally doesn't have future tokens yet — the mask ensures training matches this constraint. Without it, the model would learn patterns that depend on future context and fail catastrophically at generation time.

### Scenario 2: Prefix Caching Optimization

In production serving, when a user sends a long system prompt + short question, the model computes KV cache for the system prompt. The causal mask ensures that the system prompt's representations are **independent of the user's question** (the prompt can't attend to future user tokens). This means the prompt's KV cache can be **pre-computed and reused** across different user queries — a critical optimization called **prefix caching** or **prompt caching** that reduces TTFT by 50-80%.

---

## Follow-Up Questions

### Q1: "How does the causal mask differ from the padding mask?"

**Answer:** The **causal mask** prevents attending to future positions (upper-triangular blocking). The **padding mask** prevents attending to padding tokens (variable-length sequences are padded to the same length in a batch). Both are applied before softmax. In practice, they're combined: `combined_mask = causal_mask | padding_mask`. The causal mask is fixed for a given sequence length; the padding mask varies per sample in the batch.

### Q2: "Why doesn't BERT use causal masking?"

**Answer:** BERT's training objective is **masked language modeling (MLM)** — it predicts randomly masked tokens within a sequence. To predict a masked token accurately, the model benefits from seeing tokens **both before and after** the mask. Causal masking would prevent this, degrading BERT's ability to capture bidirectional context. The trade-off is that BERT cannot be used for autoregressive generation — it's designed for understanding (classification, extraction), not generation.

### Q3: "What is prefix LM masking?"

**Answer:** Prefix LM masking is a hybrid: a **prefix portion** of the sequence uses bidirectional attention (no mask), and the remaining **generation portion** uses causal masking. This lets the model fully contextualize the input prompt (bidirectional) while still generating autoregressively. Models like PaLM-2 and UL2 use this pattern. It's more efficient than encoder-decoder (single model, shared parameters) while preserving some benefits of bidirectional encoding for the input.

---

<p align="center">
  <a href="../README.md">← Back to all questions</a>
</p>
