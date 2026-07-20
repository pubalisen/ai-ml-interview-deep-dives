# Part 27: KV Cache

> **The Question:** "What is KV cache, and how does it speed up inference?"

---

## What They're Actually Testing

This is a production inference question. They want you to explain why caching key-value pairs eliminates redundant computation during autoregressive generation.

---

## The Technical Breakdown

### The Problem Without KV Cache

During autoregressive generation, at step t, the model generates token t based on all previous tokens [0, 1, ..., t-1]. Without caching, the model **recomputes** the Key and Value vectors for ALL previous tokens at every step:

```
Step 1: Compute K,V for tokens [0] → generate token 1
Step 2: Compute K,V for tokens [0,1] → generate token 2        ← recomputes token 0
Step 3: Compute K,V for tokens [0,1,2] → generate token 3      ← recomputes tokens 0,1
...
Step n: Compute K,V for tokens [0,...,n-1] → generate token n   ← recomputes everything
```

Total computation: O(n²) forward passes worth of Key/Value computation.

### The Solution: KV Cache

Cache the Key and Value vectors from each token's computation. At step t, only compute K,V for the **new token** and append to the cache:

```
Step 1: Compute K₀,V₀ → cache {K₀,V₀} → generate token 1
Step 2: Compute K₁,V₁ → cache {K₀,V₀,K₁,V₁} → attend new Q₁ to all cached K,V
Step 3: Compute K₂,V₂ → cache {K₀,V₀,K₁,V₁,K₂,V₂} → attend Q₂ to all cached K,V
```

Total computation: O(n) forward passes — each step only processes one new token.

### Memory Cost

```
KV cache size = 2 × num_layers × num_kv_heads × d_head × seq_len × dtype_size

Example (LLaMA 2 70B, 4K context, FP16):
= 2 × 80 layers × 8 KV heads × 128 d_head × 4096 seq_len × 2 bytes
= 2 × 80 × 8 × 128 × 4096 × 2
≈ 13.4 GB per request
```

The KV cache is often the **dominant memory consumer** during inference, exceeding the model weights for long contexts.

---

## Accuracy Check

| Claim | Verified? |
|-------|-----------|
| Without KV cache, K/V are recomputed for all previous tokens | ✅ Fundamental autoregressive computation |
| KV cache reduces recomputation from O(n²) to O(n) | ✅ Standard inference optimization |
| KV cache is often the largest memory consumer | ✅ vLLM documentation, Pope et al. (2022) |

---

## Scenario Examples

### Scenario 1: Memory-Limited Serving

A team serves a 13B model on a single A100 (80GB). The model weights take 26GB (FP16). With KV cache for 128K context across 4 concurrent requests, they'd need 4 × 50GB = 200GB — far exceeding available memory. Solutions: (1) reduce max context length, (2) use GQA to reduce KV heads, (3) use PagedAttention (vLLM) to manage KV cache like virtual memory, (4) quantize KV cache to INT8.

### Scenario 2: Prefix Caching

100 requests share the same 2K-token system prompt. Without optimization, each request computes and caches the same 2K KV pairs independently. With **prefix caching**, the KV cache for the shared prefix is computed once and shared across all requests — saving 99% of the prefill compute for that prefix.

---

## Follow-Up Questions

### Q1: "What is PagedAttention?"

**Answer:** PagedAttention (Kwon et al., 2023, used in vLLM) manages KV cache like **virtual memory pages**. Instead of pre-allocating a contiguous block for each request's full context length, it allocates small fixed-size pages on demand. This eliminates memory fragmentation and waste — a request using 500 tokens doesn't reserve 128K tokens of memory. PagedAttention typically improves throughput by 2-4× by packing more concurrent requests into GPU memory.

### Q2: "How does GQA reduce KV cache size?"

**Answer:** Standard MHA has num_heads KV pairs per layer. GQA groups multiple query heads to share one KV pair — e.g., 32 query heads with 8 KV groups reduces KV cache by 4×. This directly reduces the `num_kv_heads` term in the cache size formula. LLaMA 2 70B uses GQA (8 KV heads vs 64 query heads), reducing KV cache by 8× compared to full MHA.

### Q3: "Can you quantize the KV cache?"

**Answer:** Yes — KV cache quantization (e.g., FP16 → INT8 or INT4) reduces memory by 2-4× with minimal quality loss. The key insight is that KV values don't need full precision — they're intermediate representations, not final outputs. Libraries like vLLM and TensorRT-LLM support KV cache quantization. Combined with GQA and PagedAttention, this can reduce KV cache memory by 16-32×.

---

<p align="center">
  <a href="../README.md">← Back to all questions</a>
</p>
