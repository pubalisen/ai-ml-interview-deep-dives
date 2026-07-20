# Part 21: Why the First Token Is Slower

> **The Question:** "Why is the first token slower than the rest in an LLM?"

---

## What They're Actually Testing

This is a systems/inference question. They want you to explain the **prefill vs. decode** dichotomy and the different computational bottlenecks at each phase.

---

## The Technical Breakdown

### Two Phases of LLM Inference

| Phase | What Happens | Bottleneck | Speed |
|-------|-------------|-----------|-------|
| **Prefill** | Process ALL input tokens simultaneously through the Transformer → build KV cache → generate first output token | **Compute-bound** (matrix multiplications on full input) | Slow (1-3 seconds for long prompts) |
| **Decode** | Generate tokens one at a time, each using the KV cache from previous tokens | **Memory-bandwidth-bound** (reading large KV cache from GPU memory) | Fast (~20-100 tokens/sec) |

### Why Prefill Is Compute-Bound

During prefill, the model computes self-attention over the **entire input sequence**:
```
Attention matrix: [prompt_len × prompt_len] — e.g., 4000 × 4000 = 16M entries
FFN computation: [prompt_len × d_ff] for every layer
```

This involves massive matrix multiplications that saturate the GPU's compute units.

### Why Decode Is Memory-Bandwidth-Bound

During decode, only **one new token** is processed per step. The computation itself is tiny — but the model must read the entire KV cache from GPU memory for each attention head at each layer:
```
KV cache read per step: 2 × num_layers × num_heads × seq_len × d_head × dtype_size
For a 70B model at 4K context: ~10-20 GB of memory reads per token
```

The GPU's arithmetic units are mostly idle — they're waiting for data to be fetched from memory.

### Time to First Token (TTFT) vs. Inter-Token Latency (ITL)

```
User prompt (2000 tokens) → [TTFT: 800ms] → First token → [ITL: 30ms] → [30ms] → [30ms] → ...
```

TTFT = prefill time (proportional to prompt length)
ITL = decode time (relatively constant per token)

---

## Accuracy Check

| Claim | Verified? |
|-------|-----------|
| Prefill processes all tokens simultaneously | ✅ Standard Transformer inference; vLLM documentation |
| Prefill is compute-bound | ✅ Pope et al. (2022), "Efficiently Scaling Transformer Inference" |
| Decode is memory-bandwidth-bound | ✅ Pope et al. (2022); Williams et al. roofline model |
| TTFT is proportional to prompt length | ✅ Measurable in production LLM systems |

---

## Scenario Examples

### Scenario 1: Optimizing User Experience

A chatbot has a 4K-token system prompt. Users complain about a 2-second delay before the response starts typing. The TTFT is dominated by prefilling the 4K system prompt. Solutions: (1) **prefix caching** — pre-compute and cache the KV states for the system prompt, so only the user's message needs prefill, (2) shorten the system prompt, (3) use a smaller, faster model for initial response generation.

### Scenario 2: Batching Strategy

An API serving 100 concurrent requests must balance TTFT and throughput. If all requests prefill simultaneously, TTFT is low but GPU utilization during decode is poor (compute-bound phase followed by memory-bound phase). Modern serving frameworks like vLLM use **continuous batching** — new requests can begin prefill while existing requests are in the decode phase, keeping the GPU busy during both phases.

---

## Follow-Up Questions

### Q1: "How does prefix caching reduce TTFT?"

**Answer:** If many requests share the same prefix (e.g., system prompt), the KV cache for that prefix can be computed once and reused. Subsequent requests only need to prefill the **new tokens** after the cached prefix. For a 4K system prompt + 200-token user message, prefix caching reduces prefill from 4200 tokens to 200 tokens — a ~20× TTFT improvement.

### Q2: "What is speculative decoding and how does it address the decode bottleneck?"

**Answer:** Speculative decoding uses a small, fast "draft" model to generate several candidate tokens in parallel, then the large model **verifies** them in a single forward pass. Since verification is like a short prefill (compute-bound, parallelizable), it's much faster than sequential generation. If the draft model guessed correctly (which it often does for predictable text), you get multiple tokens for the cost of one large-model forward pass. Typical speedup: 2-3×.

### Q3: "How does Flash Attention affect TTFT?"

**Answer:** Flash Attention primarily speeds up the **prefill phase** by computing attention without materializing the full n×n attention matrix in GPU HBM. It uses a tiled algorithm that keeps data in fast SRAM, reducing memory reads by 5-20×. This reduces TTFT significantly, especially for long prompts. For the decode phase, Flash Attention provides more modest improvements since the bottleneck is reading the KV cache, not computing attention.

---

<p align="center">
  <a href="../README.md">← Back to all questions</a>
</p>
