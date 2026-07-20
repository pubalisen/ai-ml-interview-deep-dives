# Part 38: Autoregressive vs. Masked Language Modeling

> **The Question:** "Explain the difference between autoregressive and masked language modeling."

---

## What They're Actually Testing

They want you to compare the two dominant pre-training objectives and explain why each produces models suited for different tasks.

---

## The Technical Breakdown

| Property | Autoregressive (CLM) | Masked Language Modeling (MLM) |
|----------|---------------------|-------------------------------|
| **Objective** | Predict the next token | Predict randomly masked tokens (15%) |
| **Attention** | Causal (unidirectional) | Bidirectional |
| **Context used** | Only past tokens | Full surrounding context |
| **Training signal** | Every token contributes to loss | Only masked tokens (~15%) contribute |
| **Best for** | Text generation | Text understanding (classification, NER) |
| **Models** | GPT, LLaMA, Claude | BERT, RoBERTa, DeBERTa |

### Autoregressive (Causal Language Modeling)

```
Input:  "The cat sat on the ___"
Target: "mat"
Context: Only "The cat sat on the" (left context)
```

The model learns to predict each token from its left context. Training uses 100% of tokens.

### Masked Language Modeling

```
Input:  "The [MASK] sat on the mat"
Target: "cat"
Context: "The ___ sat on the mat" (full bidirectional context)
```

The model learns to predict masked tokens from both left AND right context. Training uses only ~15% of tokens.

### Why This Matters

- **Generation**: Autoregressive is natural for generation — the model generates left-to-right. MLM can't generate text naturally because it needs future tokens that don't exist yet.
- **Understanding**: MLM produces better representations for classification because bidirectional attention captures richer context. "Bank" in "I went to the bank by the river" gets context from both "went to" AND "by the river."
- **Efficiency**: Autoregressive training uses every token as a target (100% utilization). MLM only trains on masked tokens (15% utilization), needing more data to achieve equivalent exposure.

---

## Accuracy Check

| Claim | Verified? |
|-------|-----------|
| CLM predicts next token; MLM predicts masked tokens | ✅ GPT (Radford et al., 2018), BERT (Devlin et al., 2019) |
| MLM uses only 15% of tokens for training signal | ✅ BERT paper — 15% mask rate |
| Bidirectional attention produces better representations for understanding | ✅ BERT outperforms GPT on GLUE benchmarks |
| Autoregressive is more natural for generation | ✅ Fundamental architecture constraint |

---

## Scenario Examples

### Scenario 1: A team needs a sentiment classifier. BERT (MLM-trained) achieves 94% accuracy because bidirectional attention captures the full sentence context. GPT-2 (autoregressive) achieves only 89% — the causal mask prevents it from seeing future words that provide crucial context for understanding sentiment.

### Scenario 2: A team needs a code completion tool. GPT/Codex (autoregressive) excels because code completion is inherently left-to-right — you type code and the model predicts what comes next. BERT can't do this naturally because it's trained to fill in masked positions, not extend sequences.

---

## Follow-Up Questions

### Q1: "Can you combine both objectives?"

**Answer:** Yes — models like XLNet use permutation language modeling (all possible orderings), and UL2/PaLM-2 use a mixture of objectives. T5 uses a span corruption objective (mask spans of text and predict them). These hybrid approaches aim to get both generation and understanding capabilities, but decoder-only CLM has emerged as the dominant approach for LLMs.

### Q2: "Why did decoder-only (autoregressive) win over encoder-only (MLM)?"

**Answer:** Scaling laws favor autoregressive models. With CLM, you get training signal from every token (100% utilization). MLM only uses 15%. At trillion-token scale, this efficiency difference is massive. Additionally, autoregressive models can do both generation AND understanding (via prompting), while MLM models can only do understanding.

### Q3: "What is the 'pre-training objective mismatch' problem?"

**Answer:** BERT is pre-trained with [MASK] tokens, but at fine-tuning/inference, there are no [MASK] tokens — creating a distribution mismatch. The model never saw un-masked text during pre-training. Mitigations: keep 10% of "masked" tokens unchanged, replace 10% with random tokens. Autoregressive models don't have this problem — training and inference use the same format.

---

<p align="center">
  <a href="../README.md">← Back to all questions</a>
</p>
