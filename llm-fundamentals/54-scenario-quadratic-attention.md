# Scenario 9: Quadratic Attention Scaling

> **The Scenario:** "Your Transformer runs out of memory on long documents due to quadratic self-attention. How do you scale it?"

---

## Solutions
1. **Flash Attention** — Compute exact attention in tiles without materializing the n×n matrix. O(n) memory instead of O(n²). Drop-in replacement.
2. **Sparse attention** — Only attend to a subset of tokens: local windows + global tokens (BigBird, Longformer).
3. **Linear attention** — Approximate attention with kernel methods for O(n) complexity (Performer, RWKV).
4. **Chunked cross-attention** — Process long inputs in chunks with cross-attention between them.
5. **Ring Attention** — Distribute the sequence across GPUs, each computing a chunk of attention, passing KV blocks in a ring.

## Scenario Examples
### A: Flash Attention enables a model trained at 4K context to serve 128K contexts without architecture changes — same math, much less memory.
### B: A document search system processes 100K-token legal filings using Longformer's sliding window attention (O(n)) for encoding, reducing memory from 40GB to 4GB.

## Follow-Up Questions
### Q1: "Which approach has the best quality/efficiency trade-off?"
**Answer:** Flash Attention — it's exact (no quality loss) and 2-4× faster. For truly massive sequences (>1M tokens), sparse or linear attention may be necessary, accepting some quality trade-off.

### Q2: "What is Ring Attention?"
**Answer:** Ring Attention distributes the sequence across multiple GPUs. Each GPU holds a block of Q and iteratively receives KV blocks from other GPUs in a ring pattern. This enables training/inference on sequences longer than what fits on a single GPU's memory.

### Q3: "Are there attention-free architectures?"
**Answer:** Yes — Mamba (structured state spaces), RWKV (linear RNN), and RetNet replace attention with recurrent/convolutional operations that are O(n) in sequence length. They show competitive results on some benchmarks but haven't yet matched Transformers at frontier scale.

---

<p align="center"><a href="../README.md">← Back to all questions</a></p>
