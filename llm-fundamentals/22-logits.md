# Part 23: Logits in Text Generation

> **The Question:** "What are logits, and how are they used in text generation?"

---

## What They're Actually Testing

They want to confirm you understand the raw model output (logits) vs. the probability distribution (after softmax), and how sampling decisions work on this output.

---

## The Technical Breakdown

### What Are Logits?

Logits are the **raw, unnormalized scores** output by the model's final linear layer — one score per token in the vocabulary. They represent the model's "confidence" in each token being the next one, **before** conversion to probabilities.

```
Final hidden state (d_model=4096) → Linear layer (4096 × vocab_size) → Logits (vocab_size)

Example for vocab_size = 5 (simplified):
Logits: [5.2, 2.1, 1.8, -0.3, -4.1]
         ↓ softmax
Probs:  [0.72, 0.03, 0.02, 0.003, 0.0001]
         Paris London Berlin Tokyo Mars
```

Logits can be **any real number** (positive, negative, or zero). Softmax converts them to a valid probability distribution (all positive, sum to 1).

### How Logits Flow Through Generation

```
Hidden state → Linear projection → Logits → Temperature scaling → Top-p/Top-k → Softmax → Sample
                                     ↓
                              (raw model output)
```

### Why Logits Matter in Production

1. **Log probabilities** — Many APIs return log-probs (log of softmax(logits)) for confidence scoring
2. **Classifier-free guidance** — Manipulate logits from multiple forward passes before softmax
3. **Constrained decoding** — Set logits of invalid tokens to -inf to force valid output (e.g., JSON)
4. **Token healing** — Adjust logits to fix tokenization artifacts

---

## Accuracy Check

| Claim | Verified? |
|-------|-----------|
| Logits are raw unnormalized scores from the final linear layer | ✅ Standard neural network terminology |
| Softmax converts logits to probability distribution | ✅ Mathematical definition |
| Temperature divides logits before softmax | ✅ OpenAI API, all inference frameworks |
| Constrained decoding sets invalid logits to -inf | ✅ Used in structured output (Outlines, guidance) |

---

## Scenario Examples

### Scenario 1: Confidence Scoring

An AI classification system categorizes customer emails. The team uses **log-probabilities** (derived from logits) to measure confidence: if the top token's log-prob is > -0.1 (high confidence), auto-route the email. If it's < -1.0 (low confidence), escalate to a human. Without access to logits/log-probs, you'd have no way to gauge how certain the model is about its output.

### Scenario 2: Forcing Valid JSON

A data pipeline requires valid JSON output. Using **logits manipulation**, after generating `{"name": "`, the system sets logits for all non-printable and non-JSON-safe characters to -inf, ensuring the model can only generate valid string content. After a closing `"`, logits for everything except `,`, `}`, or `:` are suppressed. Libraries like Outlines and guidance implement this via logit processors.

---

## Follow-Up Questions

### Q1: "What's the difference between logits and log-probabilities?"

**Answer:** Logits are raw scores before softmax. Log-probabilities are `log(softmax(logits))` — the natural log of the probability. Log-probs are useful because (1) they're numerically stable (probabilities can be tiny like 1e-10), (2) they're additive — the log-prob of a sequence is the sum of per-token log-probs, making sequence scoring easy, and (3) APIs commonly return them (OpenAI's `logprobs` parameter).

### Q2: "How do logit processors work in Hugging Face?"

**Answer:** Logit processors are functions that modify the logits tensor before sampling. They're applied in order: `processed_logits = processor(input_ids, logits)`. Common processors: TemperatureLogitsWarper (divides by T), TopPLogitsWarper (sets low-prob logits to -inf), RepetitionPenaltyLogitsProcessor (reduces logits for already-generated tokens). You can write custom processors for domain-specific constraints.

### Q3: "Can two different logit vectors produce the same probability distribution?"

**Answer:** Yes — softmax is invariant to constant shifts. Adding the same constant c to all logits produces the same probabilities: `softmax(z + c) = softmax(z)`. This means logit magnitudes are relative, not absolute. A logit of 10 doesn't mean "very confident" — it only means confident relative to the other logits. This is why temperature (which changes **relative** differences) works, while a constant offset would have no effect.

---

<p align="center">
  <a href="../README.md">← Back to all questions</a>
</p>
