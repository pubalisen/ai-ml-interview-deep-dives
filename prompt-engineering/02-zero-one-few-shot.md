# Part 2: Zero-Shot, One-Shot, and Few-Shot Prompting

> **The Question:** "Explain zero-shot, one-shot, and few-shot prompting with examples."

---

## What They're Actually Testing

They want you to explain the spectrum of providing examples to an LLM, when each is appropriate, and the quality/cost trade-offs.

---

## The Technical Breakdown

### Zero-Shot Prompting

No examples — just instructions. The model relies entirely on its pre-training knowledge:

```
Classify the sentiment of this review as Positive, Negative, or Neutral:
"The battery life is amazing but the camera quality is disappointing."

→ Answer: Neutral
```

**When to use:** Simple, well-understood tasks (sentiment, translation, summarization) where the model's pre-trained knowledge is sufficient.

### One-Shot Prompting

One example demonstrating the expected behavior:

```
Classify the sentiment:
Review: "Best phone I've ever owned!" → Positive

Now classify:
Review: "The battery life is amazing but the camera quality is disappointing." → 
```

**When to use:** When you need to demonstrate a specific format or interpretation that differs from the model's default.

### Few-Shot Prompting

Multiple examples (typically 3-10) demonstrating the task:

```
Extract the product and issue from customer complaints:

Complaint: "My iPhone screen cracked after one drop"
→ {"product": "iPhone", "issue": "cracked screen"}

Complaint: "The AirPods keep disconnecting from my Mac"
→ {"product": "AirPods", "issue": "bluetooth disconnection"}

Complaint: "MacBook Pro overheats during video calls"
→ {"product": "MacBook Pro", "issue": "overheating"}

Complaint: "My Apple Watch band broke after 2 months"
→ 
```

**When to use:** Complex tasks, specific output formats, domain-specific patterns, or when zero-shot accuracy is insufficient.

### Comparison

| Approach | Examples | Context Cost | Accuracy | Best For |
|----------|----------|-------------|----------|----------|
| **Zero-shot** | 0 | Minimal | 60-80% | Simple, well-known tasks |
| **One-shot** | 1 | Low | 70-85% | Format demonstration |
| **Few-shot** | 3-10 | Medium-High | 80-95% | Complex extraction, specific formats |

### Key Principles for Few-Shot Examples

1. **Diversity** — Cover edge cases, not just easy examples
2. **Representativeness** — Match the distribution of real inputs
3. **Order matters** — Most relevant examples last (recency bias)
4. **Consistency** — All examples must follow the same format exactly

---

## Accuracy Check

| Claim | Verified? |
|-------|-----------|
| Few-shot outperforms zero-shot on complex tasks | ✅ Brown et al. (2020), GPT-3 paper |
| Example order affects output quality | ✅ Lu et al. (2022), "Fantastically Ordered Prompts" |
| 3-10 examples is the typical few-shot range | ✅ Industry practice, limited by context window |

---

## Scenario Examples

### Scenario 1: Classification Accuracy

A zero-shot email classifier achieves 72% accuracy. Adding 5 diverse examples (including tricky edge cases like "sarcastic complaint" and "feature request disguised as bug report") improves accuracy to 89% — a 17-point gain from 5 examples, no fine-tuning needed.

### Scenario 2: Few-Shot vs. Fine-Tuning Cost

A company needs a custom entity extractor. Few-shot approach: write 8 examples, test in 1 hour, achieve 85% accuracy. Fine-tuning approach: label 5000 examples (2 weeks), train model (2 days), achieve 92% accuracy. For a prototype, few-shot wins. For production at scale, fine-tuning justifies the investment.

---

## Follow-Up Questions

### Q1: "How do you select the best few-shot examples?"

**Answer:** Use **dynamic few-shot selection** — embed all candidate examples, then at runtime, retrieve the most similar examples to the current input using cosine similarity. This ensures each query gets the most relevant examples. Libraries like LangChain and DSPy support this pattern.

### Q2: "Does more examples always mean better?"

**Answer:** No — returns diminish after 5-8 examples for most tasks. Beyond that, you're spending context window tokens with marginal accuracy gain. Also, too many examples can cause the model to **overfit to the examples** — copying patterns too literally instead of generalizing. The optimal count depends on task complexity and model size.

### Q3: "What is in-context learning, and how does it relate to few-shot?"

**Answer:** In-context learning (ICL) is the phenomenon where LLMs learn task patterns from examples in the prompt — without gradient updates. Few-shot prompting is the practical application of ICL. Research shows ICL works because the pre-trained model has learned to recognize and apply patterns from demonstrations during pre-training on diverse text.

---

<p align="center">
  <a href="../README.md">← Back to all questions</a>
</p>
