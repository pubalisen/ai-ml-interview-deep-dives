# Part 33: Grouped-Query Attention (GQA)

> **The Question:** "What is Grouped-Query Attention (GQA), and how does it differ from Multi-Head Attention (MHA)?"

---

## What They're Actually Testing

They want you to explain GQA as an inference optimization that reduces KV cache memory by sharing KV heads.

---

## The Technical Breakdown

### The Spectrum

```
MHA:  32 Q heads, 32 KV heads (1:1 — each query has its own KV)
GQA:  32 Q heads, 8 KV heads  (4:1 — 4 query heads share 1 KV pair)
MQA:  32 Q heads, 1 KV head   (32:1 — all query heads share 1 KV pair)
```

| Architecture | KV heads | KV cache size | Quality | Used by |
|-------------|----------|---------------|---------|---------|
| **MHA** | = num_heads | Baseline | Best | BERT, GPT-2 |
| **GQA** | < num_heads | Reduced by group factor | Near-MHA | LLaMA 2 70B, LLaMA 3, Mistral |
| **MQA** | 1 | Minimized | Slightly lower | PaLM, StarCoder |

### Why GQA?

The KV cache is proportional to `num_kv_heads`. Reducing KV heads directly reduces:
- **KV cache memory** — serve more concurrent requests
- **Memory bandwidth** — read less data per token (decode speedup)
- **Inference latency** — fewer KV computations

GQA is the sweet spot: achieves 80-90% of MQA's memory savings while retaining MHA-level quality.

---

## Accuracy Check

| Claim | Verified? |
|-------|-----------|
| GQA groups query heads to share KV pairs | ✅ Ainslie et al. (2023), "GQA: Training Generalized Multi-Query Transformer Models" |
| LLaMA 2 70B uses GQA with 8 KV heads for 64 query heads | ✅ LLaMA 2 model card |
| GQA retains near-MHA quality | ✅ < 1% degradation in Ainslie et al. (2023) benchmarks |

---

## Scenario Examples

### Scenario 1: A team needs to serve 4× more concurrent requests on the same GPU. By converting from MHA (32 KV heads) to GQA (8 KV heads), KV cache drops 4×, fitting 4× more requests in the same memory.

### Scenario 2: An existing MHA model is converted to GQA post-training by mean-pooling adjacent KV head weights, then continued pre-training for 5% of the original tokens. This "uptrained" GQA model recovers 99% of original quality.

---

## Follow-Up Questions

### Q1: "How do you choose the number of KV groups?"

**Answer:** Common choices: 4 or 8 KV groups for 32 query heads. More groups = higher quality but less memory savings. Fewer groups = more savings but quality risk. LLaMA 3 uses 8 KV heads for 32 query heads (4:1 ratio) as the empirical sweet spot.

### Q2: "Can you convert MHA to GQA without retraining?"

**Answer:** You can merge KV heads by averaging weights, but quality degrades without continued training. The standard approach is "uptraining" — initializing GQA from MHA weights and training for a small fraction (2-5%) of the original training compute.

### Q3: "Does GQA affect training or only inference?"

**Answer:** GQA affects both. Training uses fewer KV projections (slightly faster training). But the primary benefit is inference — reduced KV cache memory and bandwidth. Some teams train with full MHA and convert to GQA for serving.

---

<p align="center">
  <a href="../../README.md">← Back to all questions</a>
</p>
