# Part 22: Top-p and Top-k Sampling

> **The Question:** "Explain Top-p (nucleus) sampling and Top-k sampling. How do they differ?"

---

## What They're Actually Testing

The interviewer wants you to explain two decoding strategies mechanically and articulate when each is preferred.

---

## The Technical Breakdown

### Top-k Sampling

Select the **k most probable** tokens, redistribute probability among them, and sample:

```
Vocabulary probabilities: {Paris: 0.40, London: 0.25, Berlin: 0.15, Tokyo: 0.08, Mars: 0.02, ...}

Top-k=3: Keep only {Paris: 0.40, London: 0.25, Berlin: 0.15}
Renormalize: {Paris: 0.50, London: 0.31, Berlin: 0.19}
Sample from these 3 tokens
```

**Problem:** k is fixed regardless of the distribution shape. If the model is very confident (90% on one token), k=50 still includes 49 low-probability tokens. If the model is uncertain (many tokens at ~5%), k=5 might exclude valid options.

### Top-p (Nucleus) Sampling

Select the **smallest set of tokens** whose cumulative probability ≥ p, then sample:

```
Sorted probabilities: {Paris: 0.40, London: 0.25, Berlin: 0.15, Tokyo: 0.08, Rome: 0.05, ...}

Top-p=0.9: 
  Paris (0.40) → cumsum = 0.40
  London (0.25) → cumsum = 0.65
  Berlin (0.15) → cumsum = 0.80
  Tokyo (0.08) → cumsum = 0.88
  Rome (0.05) → cumsum = 0.93 ≥ 0.9 → STOP

Keep: {Paris, London, Berlin, Tokyo, Rome} → 5 tokens
Renormalize and sample
```

**Advantage:** The candidate set size **adapts** to the model's confidence. High confidence → fewer candidates. Low confidence → more candidates.

### Comparison

| Aspect | Top-k | Top-p |
|--------|-------|-------|
| **Candidate set** | Fixed (always k tokens) | Dynamic (adapts to distribution) |
| **High confidence** | Still considers k tokens (wastes probability on noise) | Narrows to few tokens |
| **Low confidence** | May miss valid tokens beyond rank k | Includes all tokens in the nucleus |
| **Typical values** | k=40-100 | p=0.9-0.95 |
| **Used by default** | Less common | ✅ Industry standard |

### Combined Usage

Most APIs combine both: apply top-k first (hard cutoff), then top-p within the remaining candidates:
```
Full vocab (50K) → Top-k=100 → Top-p=0.95 → Temperature sampling → Selected token
```

---

## Accuracy Check

| Claim | Verified? |
|-------|-----------|
| Top-k selects the k highest-probability tokens | ✅ Fan et al. (2018) |
| Top-p selects smallest set with cumulative probability ≥ p | ✅ Holtzman et al. (2020), "The Curious Case of Neural Text Degeneration" |
| Top-p is adaptive to distribution shape | ✅ Demonstrated in Holtzman et al. (2020) |
| Most production APIs default to top-p | ✅ OpenAI, Anthropic defaults use top-p=1.0 with temperature |

---

## Scenario Examples

### Scenario 1: Code Generation Precision

A code completion tool needs high precision — generating `return self.data` when that's clearly correct, but offering alternatives when multiple options are valid. Top-p=0.1 + temperature=0.2 creates a very focused distribution: when the model is 85% confident in one completion, only that token passes the nucleus filter. When it's uncertain (30% for option A, 25% for option B), both are included, allowing useful variation.

### Scenario 2: Creative Story Generation

A story writing app uses top-k=50 and the model generates repetitive narratives. Analysis shows that at narrative decision points, the model distributes probability across 200+ tokens (many valid story directions). With k=50, 150 valid continuations are excluded, causing the model to repeatedly choose from the same small set. Switching to top-p=0.95 allows the candidate set to expand at these decision points (sometimes including 200+ tokens) while staying focused during predictable passages.

---

## Follow-Up Questions

### Q1: "What is greedy decoding and when is it appropriate?"

**Answer:** Greedy decoding always picks the single highest-probability token (equivalent to top-k=1 or temperature=0). It produces deterministic output and is appropriate for tasks where there's one correct answer: classification, extraction, structured output, math, code. The downside: it can get stuck in repetitive loops because it never explores alternative paths.

### Q2: "What causes the 'degeneration' problem in text generation?"

**Answer:** Holtzman et al. (2020) showed that pure sampling (no truncation) from the full vocabulary produces degenerate text because the long tail of low-probability tokens collectively has significant probability mass. Even tokens with 0.01% probability occasionally get sampled, producing incoherent text. Top-p solves this by eliminating the long tail — only tokens in the high-probability nucleus are candidates.

### Q3: "How do you choose between top-p and top-k for a production system?"

**Answer:** Use **top-p** as the primary control because it adapts to the model's confidence level. Set top-p=0.9-0.95 for most applications. Add top-k as a **safety ceiling** (k=100-200) to prevent pathological cases where the distribution is extremely flat. Adjust temperature separately for creativity level. The combination `temperature=0.7, top_p=0.95, top_k=100` is a solid default for conversational applications.

---

<p align="center">
  <a href="../README.md">← Back to all questions</a>
</p>
