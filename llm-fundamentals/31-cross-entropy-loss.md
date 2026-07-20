# Part 32: Cross-Entropy Loss

> **The Question:** "What is Cross-Entropy Loss?"

---

## What They're Actually Testing

They want you to explain the training objective of LLMs mechanically — how the model's predicted distribution is compared to the actual next token.

---

## The Technical Breakdown

### What Is Cross-Entropy Loss?

Cross-entropy loss measures the **difference between the model's predicted probability distribution and the actual (one-hot) target distribution**. For LLMs, it measures how well the model predicts the next token:

```
Predicted: P("Paris") = 0.72, P("London") = 0.15, P("Berlin") = 0.08, ...
Actual:    "Paris" (one-hot: 1 for Paris, 0 for everything else)

Cross-entropy = -log(P("Paris")) = -log(0.72) = 0.33
```

Lower loss = higher probability assigned to the correct token = better model.

### The Formula

```
CE(y, ŷ) = -Σ y_i × log(ŷ_i)

For one-hot targets (language modeling):
CE = -log(ŷ_correct)    # Only the correct token contributes
```

### Why Cross-Entropy and Not MSE?

| Property | Cross-Entropy | Mean Squared Error |
|----------|--------------|-------------------|
| **Gradient behavior** | Strong gradients when prediction is wrong | Weak gradients when prediction is far off |
| **Probability output** | Natural pairing with softmax | Doesn't assume probability distribution |
| **Classification** | ✅ Standard choice | ❌ Poor convergence |

### Perplexity: The Interpretable Metric

**Perplexity = e^(cross-entropy)**. Intuition: the average number of equally likely tokens the model is "confused" between.

```
Perplexity 1 = perfect (always predicts the right token)
Perplexity 10 = model is choosing between ~10 equally likely tokens
Perplexity 50000 = random guessing over entire vocabulary
```

---

## Accuracy Check

| Claim | Verified? |
|-------|-----------|
| LLMs are trained with cross-entropy loss | ✅ GPT, LLaMA, all autoregressive models |
| CE = -log(predicted probability of correct token) | ✅ Standard loss function |
| Perplexity = exp(cross-entropy) | ✅ Standard NLP metric |

---

## Scenario Examples

### Scenario 1: Training Monitoring

During pre-training, a team monitors cross-entropy loss. It starts at ~10 (perplexity ~22K — near random) and drops to ~2.5 (perplexity ~12) after 1 epoch. A sudden loss spike at step 50K indicates a bad data batch — they inspect and find corrupted training data.

### Scenario 2: Model Comparison

Team compares two fine-tuned models. Model A: perplexity 8.2 on their test set. Model B: perplexity 6.1. Model B assigns higher probability to correct tokens on average, suggesting it has better language understanding for this domain. But perplexity alone doesn't capture output quality — they also run human evaluations.

---

## Follow-Up Questions

### Q1: "Why is lower perplexity not always better for downstream tasks?"

**Answer:** Perplexity measures how well the model predicts the next token in general. But downstream tasks (summarization, code generation) have specific quality criteria. A model with low perplexity might generate fluent but factually wrong text (high perplexity improvement from memorizing frequent patterns, not from understanding). Task-specific evaluation metrics (ROUGE, pass@k, human evaluation) are essential alongside perplexity.

### Q2: "How does cross-entropy relate to the training objective of GPT vs BERT?"

**Answer:** GPT uses cross-entropy on **every** token (next-token prediction). BERT uses cross-entropy only on **masked tokens** (~15% of tokens). This means GPT gets training signal from the entire sequence, while BERT gets signal from a small subset. GPT's approach is more compute-efficient per token of training data.

### Q3: "What is label smoothing and why is it used?"

**Answer:** Instead of one-hot targets (1 for correct token, 0 for all others), label smoothing distributes a small probability ε to incorrect tokens: correct = 1-ε, each incorrect = ε/(V-1). This prevents the model from becoming overconfident and improves generalization. It softens the cross-entropy target, regularizing training.

---

<p align="center">
  <a href="../README.md">← Back to all questions</a>
</p>
