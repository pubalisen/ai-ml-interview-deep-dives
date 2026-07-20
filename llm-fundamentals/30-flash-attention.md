# Part 31: Flash Attention

> **The Question:** "What is Flash Attention?"

---

## What They're Actually Testing

They want you to explain that Flash Attention is an **IO-aware** implementation optimization — it computes the exact same attention but avoids materializing the full n×n matrix in GPU HBM.

---

## The Technical Breakdown

### The Problem

Standard attention computes: `softmax(QK^T / √d_k) × V`

This requires materializing the full **n×n attention matrix** in GPU High Bandwidth Memory (HBM):
- 4K context → 16M entries → 32MB (FP16)
- 128K context → 16.4B entries → 32GB (FP16) — exceeds most GPU memory

The bottleneck isn't compute — GPUs have plenty of FLOPs. The bottleneck is **memory bandwidth** — reading/writing the huge attention matrix from/to slow HBM.

### Flash Attention Solution

Flash Attention (Dao et al., 2022) uses **tiling** — processes attention in small blocks that fit in fast GPU SRAM (shared memory):

```
Instead of: Compute full QK^T (n×n) → store in HBM → softmax → multiply by V
            (3 HBM round trips)

Flash does: For each tile of Q:
              For each tile of K, V:
                Compute partial attention in SRAM
                Accumulate result (online softmax)
            (1 HBM round trip, tiles stay in SRAM)
```

### Results

| Metric | Standard Attention | Flash Attention |
|--------|-------------------|-----------------|
| **Memory** | O(n²) | O(n) — never materializes full matrix |
| **HBM reads/writes** | O(n²·d) | O(n²·d² / M) where M = SRAM size |
| **Wall-clock speedup** | Baseline | 2-4× faster |
| **Supports longer contexts** | Limited by memory | ✅ Enables 128K+ contexts |

### Flash Attention 2 and 3

- **FA-2** (2023): Better work partitioning across GPU warps → additional 2× speedup
- **FA-3** (2024): Leverages Hopper GPU features (FP8, warp-level scheduling) → another 1.5-2× on H100

---

## Accuracy Check

| Claim | Verified? |
|-------|-----------|
| Standard attention materializes n×n matrix in HBM | ✅ Standard PyTorch implementation |
| Flash Attention uses tiling to avoid n×n materialization | ✅ Dao et al. (2022), "FlashAttention: Fast and Memory-Efficient Exact Attention" |
| Reduces memory from O(n²) to O(n) | ✅ Core contribution |
| Achieves 2-4× wall-clock speedup | ✅ Benchmarks in Dao et al. (2022) |

---

## Scenario Examples

### Scenario 1: Enabling Long Context

A team wants to serve a model with 128K context. With standard attention, the 128K × 128K attention matrix would require 32GB per layer — impossible. Flash Attention never materializes this matrix, computing it in tiles that fit in 20MB of SRAM. This makes 128K (and even 1M+) context windows practically feasible.

### Scenario 2: Training Throughput

During pre-training, Flash Attention reduces training time by 40% — not because the model is doing less computation, but because it makes fewer memory round trips. The GPU's compute units spend more time computing and less time waiting for data from HBM. For a multi-billion dollar training run, this translates to millions saved.

---

## Follow-Up Questions

### Q1: "Does Flash Attention change the attention output?"

**Answer:** No — Flash Attention computes **mathematically exact** standard attention. It's a pure implementation optimization. The softmax is computed using an online algorithm (Milakov & Gimelshein, 2018) that processes tiles incrementally while maintaining numerical equivalence. The output is bit-for-bit identical to standard attention (up to floating-point precision).

### Q2: "What is the 'online softmax' trick?"

**Answer:** Standard softmax requires knowing the maximum value across all elements (for numerical stability). With tiling, you don't see all elements at once. Online softmax tracks a running maximum and rescales previous partial results when a new maximum is found. This lets softmax be computed incrementally without seeing the full row, enabling the tiling approach.

### Q3: "Is Flash Attention always faster?"

**Answer:** Not for very short sequences (< 512 tokens) where the standard attention matrix fits comfortably in memory — the tiling overhead can make Flash Attention marginally slower. It's most beneficial for sequences > 2K tokens. In practice, it's used by default in all modern inference frameworks (vLLM, TGI, TensorRT-LLM) because most production use cases involve sufficiently long contexts.

---

<p align="center">
  <a href="../README.md">← Back to all questions</a>
</p>
