# Part 29: Mixture of Experts (MoE)

> **The Question:** "What is Mixture of Experts (MoE), and how does it work in models like Mixtral?"

---

## What They're Actually Testing

They want you to explain how MoE scales model capacity without proportionally scaling compute, and the engineering trade-offs (memory vs compute, load balancing).

---

## The Technical Breakdown

### What Is MoE?

MoE replaces each Transformer's dense FFN with **multiple parallel FFN "experts"**, where a **router** selects which experts process each token. Only a subset of experts activate per token, so total compute is much less than the full parameter count suggests.

```
Dense model: Every token → same FFN (all parameters active)
MoE model:   Every token → router selects top-k experts → only those FFN experts activate

Mixtral 8x7B: 8 expert FFNs per layer, router selects top-2
Total parameters: ~47B (all experts)
Active parameters per token: ~13B (2 experts + shared layers)
```

### How the Router Works

```python
router_logits = token_hidden @ router_weights    # [d_model] × [d_model, num_experts]
router_probs = softmax(router_logits)             # probability per expert
top_k_experts = topk(router_probs, k=2)          # select 2 experts

output = Σ (router_prob_i × expert_i(token))      # weighted sum of expert outputs
```

### Why MoE Matters

| Property | Dense 70B | MoE 8×7B (Mixtral) |
|----------|-----------|---------------------|
| **Total parameters** | 70B | 47B |
| **Active parameters/token** | 70B | ~13B |
| **Inference compute** | High | ~13B equivalent (3-5× faster) |
| **Memory** | 140GB (FP16) | 94GB (FP16) — all experts must be in memory |
| **Quality** | Baseline | Comparable or better (more total knowledge stored) |

### The Load Balancing Problem

Without constraints, the router might send all tokens to 1-2 "favorite" experts, leaving others untrained. Solutions:
- **Auxiliary loss** — adds a penalty for uneven expert utilization
- **Expert capacity** — limits how many tokens each expert can process per batch

---

## Accuracy Check

| Claim | Verified? |
|-------|-----------|
| Mixtral uses 8 experts with top-2 routing | ✅ Mixtral technical report (Jiang et al., 2024) |
| MoE reduces active compute while maintaining capacity | ✅ Shazeer et al. (2017), Switch Transformer (Fedus et al., 2021) |
| Load balancing loss prevents expert collapse | ✅ Standard MoE training technique |
| All expert parameters must be in memory | ✅ Hardware constraint for inference |

---

## Scenario Examples

### Scenario 1: High-Throughput API

A company needs to serve millions of requests at GPT-3.5 quality. A dense 70B model gives great quality but requires 4× A100s and is slow. Mixtral 8x7B matches quality on most tasks but activates only ~13B parameters per token, running 3× faster on fewer GPUs. The trade-off: total memory is still large (all experts in VRAM), but compute per request is dramatically lower.

### Scenario 2: Expert Specialization

Analysis of Mixtral's trained experts reveals that different experts specialize: Expert 2 activates heavily for code tokens, Expert 5 for mathematical expressions, Expert 7 for conversational text. The router has learned to dispatch tokens to domain-appropriate experts — emergent specialization from the routing mechanism, not explicit programming.

---

## Follow-Up Questions

### Q1: "Why is MoE memory-expensive despite lower compute?"

**Answer:** All expert parameters must be loaded into GPU memory, even though only k experts are active per token. For Mixtral 8×7B, the 8 expert FFN sets total ~35B parameters that all need to be in memory. During inference, different tokens in a batch may route to different experts, so you can't predict which experts to offload. This creates a paradox: MoE is compute-efficient but memory-intensive.

### Q2: "How does expert parallelism work for serving MoE models?"

**Answer:** Expert parallelism distributes different experts across different GPUs. When a token routes to Expert 3, its hidden state is sent to the GPU hosting Expert 3, processed, and the result is sent back. This requires **all-to-all communication** between GPUs, which can be a latency bottleneck. Efficient MoE serving requires high-bandwidth interconnects (NVLink, InfiniBand).

### Q3: "What is a 'shared expert' in recent MoE architectures?"

**Answer:** Some MoE designs (like DeepSeek-MoE) include a **shared expert** that processes every token alongside the routed experts. The shared expert captures general knowledge common to all inputs, while routed experts capture specialized knowledge. Formula: `output = shared_expert(x) + Σ router_prob_i × routed_expert_i(x)`. This improves quality because not all knowledge needs specialization.

---

<p align="center">
  <a href="../../README.md">← Back to all questions</a>
</p>
