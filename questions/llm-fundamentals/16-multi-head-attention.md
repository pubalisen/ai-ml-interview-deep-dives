# Part 17: Multi-Head Attention

> **The Question:** "What are multi-head attention mechanisms? Why use multiple attention heads?"

---

## What They're Actually Testing

The interviewer wants to know why running attention **multiple times in parallel** (with different projections) is better than running it once, and what different heads learn to specialize in.

---

## The Technical Breakdown

### What Is Multi-Head Attention?

Instead of computing a single attention function with d_model-dimensional keys, queries, and values, multi-head attention runs **h parallel attention heads**, each with its own learned projections operating on a smaller dimension (d_model / h):

```
head_i = Attention(X @ W_Q_i, X @ W_K_i, X @ W_V_i)     # d_k = d_model / h
MultiHead(X) = Concat(head_1, head_2, ..., head_h) @ W_O  # Combine and project
```

### Why Multiple Heads?

A single attention head can only learn **one type of relationship** per layer. Multiple heads let the model simultaneously attend to different types of information:

```
Head 1: Syntactic dependencies     → "sat" attends to "cat" (subject-verb)
Head 2: Positional/local context   → each token attends to adjacent tokens
Head 3: Coreference               → "she" attends to "Sarah"
Head 4: Punctuation/structure     → tokens attend to sentence boundaries
```

### The Math

```python
# Standard: d_model = 768, h = 12 heads, d_k = d_v = 768/12 = 64

for i in range(num_heads):
    Q_i = X @ W_Q[i]    # [seq, 768] × [768, 64] → [seq, 64]
    K_i = X @ W_K[i]    # [seq, 64]
    V_i = X @ W_V[i]    # [seq, 64]
    head_i = softmax(Q_i @ K_i.T / sqrt(64)) @ V_i  # [seq, 64]

output = concat(head_1, ..., head_12) @ W_O  # [seq, 768] × [768, 768] → [seq, 768]
```

Total parameters: **same** as single-head attention with d_model dimensions. Multi-head is a reparameterization that enables specialization, not an increase in parameters.

### What Heads Actually Learn

Research (Clark et al., 2019; Voita et al., 2019) shows distinct specializations:

| Head Type | Behavior | % of Heads |
|-----------|----------|-----------|
| **Positional** | Attend to previous/next token | ~30% |
| **Syntactic** | Attend to syntactic dependencies | ~15% |
| **Rare token** | Attend to infrequent but informative words | ~10% |
| **Separator** | Attend to punctuation and delimiters | ~10% |
| **Redundant/dead** | Contribute minimally — prunable | ~35% |

---

## Accuracy Check

| Claim | Verified? |
|-------|-----------|
| Multi-head attention uses h parallel heads with d_k = d_model/h | ✅ Vaswani et al. (2017), Section 3.2.2 |
| Total parameters equal single-head attention | ✅ Mathematically equivalent parameter count |
| Different heads specialize in different relationship types | ✅ Clark et al. (2019), Voita et al. (2019) |
| ~35% of heads are prunable without quality loss | ✅ Voita et al. (2019), "Analyzing Multi-Head Self-Attention" |

---

## Scenario Examples

### Scenario 1: Head Pruning for Efficiency

A team deploys a 12-head model on edge devices. By analyzing attention patterns, they discover 4 heads consistently produce near-uniform attention distributions (attending equally to all tokens — contributing no useful information). They prune these 4 heads, reducing the model's attention computation by 33% with only 0.5% accuracy drop. This is possible precisely because multi-head attention provides **redundancy** — the remaining heads compensate.

### Scenario 2: Debugging a Translation Model

A translation model keeps getting word order wrong in German outputs. By visualizing per-head attention weights, the team finds that the head responsible for syntactic structure (verb-final patterns in German) has very diffuse attention — it's not learning the SOV pattern. They add synthetic examples with emphasized word order during fine-tuning, and that specific head's attention patterns sharpen on verb-position relationships.

---

## Follow-Up Questions

### Q1: "What is Multi-Query Attention (MQA) and how does it differ?"

**Answer:** In standard MHA, each head has its own Q, K, V projections. In **Multi-Query Attention (MQA)**, all heads share a **single K and V** while having separate Q projections. This dramatically reduces the **KV cache** size during inference (by h×), making inference faster and less memory-intensive. The trade-off is a small quality degradation (~0.5-1% on benchmarks). MQA was introduced by Shazeer (2019) and used in PaLM.

### Q2: "What is Grouped-Query Attention (GQA)?"

**Answer:** GQA is the middle ground between MHA and MQA. Instead of each head having its own KV (MHA) or all heads sharing one KV (MQA), GQA groups heads into **g groups**, each group sharing one set of KV projections. Example: 32 query heads with 8 KV groups → 4 query heads per KV group. This achieves most of MQA's inference speedup while recovering most of MHA's quality. LLaMA 2 70B, LLaMA 3, and Mistral use GQA.

### Q3: "If 35% of heads are prunable, why not train with fewer heads?"

**Answer:** Redundancy during training is actually **beneficial**. During pre-training, the model doesn't know in advance which relationship types will be most important. Extra heads provide exploration capacity — the model can try many relationship patterns and keep what works. Pruning is an **inference optimization**: you train with full capacity for maximum learning, then prune for deployment efficiency. Also, which heads are redundant depends on the downstream task — heads that are "dead" for one task may be important for another.

---

<p align="center">
  <a href="../../README.md">← Back to all questions</a>
</p>
