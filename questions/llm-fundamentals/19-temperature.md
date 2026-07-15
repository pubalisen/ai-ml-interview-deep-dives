# Part 20: Temperature in LLMs

> **The Question:** "What is temperature in the context of LLMs, and how does it affect output?"

---

## What They're Actually Testing

The interviewer wants you to explain temperature **mechanically** — how it modifies the softmax distribution over the vocabulary — and demonstrate understanding of when to use high vs. low temperature in production.

---

## The Technical Breakdown

### What Is Temperature?

Temperature is a scalar that controls the **randomness** of the model's output by scaling the logits before the final softmax:

```
P(token_i) = exp(logit_i / T) / Σ exp(logit_j / T)
```

Where T is the temperature parameter.

### How Temperature Affects the Distribution

```
Logits: [5.0, 2.0, 1.0, 0.5]  (for tokens: "Paris", "London", "Berlin", "Tokyo")

T = 0.1 (very low):  [0.9999, 0.0001, 0.0000, 0.0000]  → almost deterministic
T = 0.5 (low):       [0.9526, 0.0351, 0.0091, 0.0033]   → strongly peaked
T = 1.0 (default):   [0.7275, 0.1815, 0.0668, 0.0243]   → balanced
T = 2.0 (high):      [0.4375, 0.2184, 0.1321, 0.0984]   → flatter/more random
T → ∞:               [0.25,   0.25,   0.25,   0.25  ]    → uniform random
```

| Temperature | Effect | Use Case |
|-------------|--------|----------|
| **T → 0** | Greedy decoding — always picks highest probability token | Factual Q&A, code generation, structured output |
| **T = 0.3-0.5** | Low randomness — mostly picks top tokens with slight variation | Summarization, translation |
| **T = 0.7-1.0** | Moderate randomness — balanced creativity and coherence | General conversation, chatbots |
| **T = 1.2-2.0** | High randomness — more creative and unpredictable | Creative writing, brainstorming |
| **T > 2.0** | Very random — often incoherent | Rarely useful in production |

### Mathematical Intuition

Dividing logits by T before softmax:
- **Low T** → logits are amplified → differences between probabilities are magnified → distribution becomes peaky
- **High T** → logits are compressed → differences shrink → distribution becomes uniform
- **T = 1** → no modification → the model's learned distribution is used as-is

---

## Accuracy Check

| Claim | Verified? |
|-------|-----------|
| Temperature divides logits before softmax | ✅ Standard implementation in all LLM APIs |
| T=0 gives greedy/deterministic output | ✅ Most APIs implement T=0 as argmax |
| Higher temperature = more random/creative | ✅ Observable behavior, mathematically proven |
| T=1 uses the model's original distribution | ✅ Dividing by 1 = no change |

---

## Scenario Examples

### Scenario 1: Production JSON Extraction

A data pipeline uses an LLM to extract structured data from invoices into JSON. With temperature=0.7, the model occasionally generates creative field names ("total_due" vs "amount_total") or adds hallucinated fields. Setting temperature=0 + using structured output (JSON schema) ensures the model always produces the most probable, consistent JSON structure. In extraction/classification tasks, any randomness is pure noise.

### Scenario 2: A/B Testing Creative Copy

A marketing team generates ad copy using an LLM. With temperature=0, every prompt produces the same output — no variation to test. They set temperature=1.2 and generate 10 variations per prompt, then A/B test them. Higher temperature produces diverse, creative alternatives: "Unlock your potential" vs "Transform your tomorrow" vs "Start something extraordinary." The trade-off: some outputs at T=1.2 are unusable ("Catapult into dimensional success"), requiring human curation.

---

## Follow-Up Questions

### Q1: "What's the relationship between temperature, top-p, and top-k?"

**Answer:** All three control sampling randomness, but at different stages:
- **Temperature** modifies the **probability distribution** (reshapes all probabilities)
- **Top-k** truncates to the **k most probable** tokens (hard cutoff)
- **Top-p** (nucleus sampling) truncates to the smallest set of tokens whose cumulative probability ≥ p

They're typically combined: temperature reshapes the distribution first, then top-p/top-k truncates it. Example: T=0.8 + top_p=0.95 → slightly sharpen the distribution, then sample from tokens covering 95% of the probability mass.

### Q2: "Should you ever set temperature=0 in production?"

**Answer:** Yes — for deterministic tasks like classification, extraction, and code generation where consistency matters more than creativity. However, T=0 (greedy decoding) can occasionally get stuck in repetitive loops because it always picks the highest-probability next token. Some APIs implement T=0 with a small epsilon (e.g., 0.001) to avoid this. For production reliability, T=0 + structured output schemas is the gold standard for data extraction pipelines.

### Q3: "How does temperature interact with the training distribution?"

**Answer:** At T=1, you're sampling from the model's **trained distribution** — the probabilities it learned during pre-training and fine-tuning. At T<1, you're making the model **more confident** than it actually is (amplifying its preferences). At T>1, you're making it **less confident** (flattening its preferences). This means T<1 can amplify biases present in training data (the model becomes more extreme), while T>1 can surface lower-probability but potentially valid alternatives.

---

<p align="center">
  <a href="../../README.md">← Back to all questions</a>
</p>
