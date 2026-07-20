# Part 46: Large Reasoning Models (LRMs)

> **The Question:** "What are Large Reasoning Models (LRMs)?"

---

## What They're Actually Testing

They want to see if you understand the shift from "token prediction" to "thinking" models — how models like o1, o3, and DeepSeek-R1 use extended chain-of-thought and test-time compute.

---

## The Technical Breakdown

### What Are LRMs?

Large Reasoning Models are LLMs specifically trained to **spend more compute at inference time** through extended internal reasoning (chain-of-thought) before producing an answer. They "think" before responding.

```
Standard LLM:    prompt → answer (fast, may be wrong)
LRM:             prompt → [extended reasoning tokens] → answer (slower, more accurate)
```

### Key Examples

| Model | Approach | Key Innovation |
|-------|----------|---------------|
| **o1/o3** (OpenAI) | Extended CoT, RL-trained reasoning | Test-time compute scaling |
| **DeepSeek-R1** | GRPO-trained reasoning chains | Open-weight reasoning model |
| **Claude 3.5 (extended thinking)** | Internal deliberation tokens | Anthropic's approach |

### Test-Time Compute Scaling

The core insight: instead of making models bigger (training compute), make them **think longer** (inference compute).

```
Accuracy vs. "thinking" tokens:
100 thinking tokens → 60% accuracy on hard math
1000 thinking tokens → 78% accuracy
10000 thinking tokens → 92% accuracy
```

More thinking tokens = more intermediate reasoning steps = higher accuracy on complex problems. This is a fundamentally different scaling axis from model size.

### How LRMs Are Trained

1. **SFT on reasoning traces** — train on (problem, step-by-step-solution) pairs
2. **RL with verifiable rewards** — use GRPO/PPO where the reward is correctness (math problems, code tests)
3. **Process reward models** — reward each reasoning step, not just the final answer

---

## Accuracy Check

| Claim | Verified? |
|-------|-----------|
| o1 uses extended chain-of-thought | ✅ OpenAI o1 blog post and system card |
| Test-time compute improves accuracy | ✅ Snell et al. (2024), "Scaling LLM Test-Time Compute" |
| DeepSeek-R1 uses GRPO for reasoning training | ✅ DeepSeek-R1 technical report |
| Process reward models improve reasoning | ✅ Lightman et al. (2023), "Let's Verify Step by Step" |

---

## Scenario Examples

### Scenario 1: A math competition problem requires 12 reasoning steps. A standard LLM gets it wrong 70% of the time (skips steps). An LRM generates 2000 tokens of intermediate reasoning, systematically working through each step, achieving 90% accuracy. The cost is 20× more tokens (20× more expensive) but the accuracy gain is crucial.

### Scenario 2: A code debugging agent uses an LRM to analyze a complex bug. Instead of immediately suggesting a fix, it generates 500 tokens of analysis: "The error occurs in line 42... this is called from... the variable state at this point would be... the root cause is..." This deliberation produces a correct fix that a standard model would miss.

---

## Follow-Up Questions

### Q1: "What's the cost trade-off of LRMs?"

**Answer:** LRMs generate 10-100× more tokens per response (thinking tokens). For o1-pro, a single hard math problem might generate 50K+ tokens internally. This makes LRMs expensive for simple queries where a standard model would suffice. The optimal strategy: route simple queries to a fast model, complex reasoning tasks to an LRM.

### Q2: "Can you train your own LRM?"

**Answer:** Yes — DeepSeek-R1 showed that GRPO training with verifiable rewards (math correctness, code tests) can produce reasoning capabilities. The key ingredients: (1) a strong base model, (2) a verifiable reward signal, (3) enough training on reasoning traces. Open-source frameworks are emerging for this.

### Q3: "What are the hidden 'thinking' tokens?"

**Answer:** In models like o1, the extended reasoning tokens are generated internally but not shown to the user — only the final answer is displayed. This creates transparency concerns: the model's reasoning is opaque. Some implementations show a summary of the thinking process. DeepSeek-R1's open-weight approach makes the full reasoning chain visible.

---

<p align="center">
  <a href="../README.md">← Back to all questions</a>
</p>
