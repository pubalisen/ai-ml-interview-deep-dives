# Part 3: Chain-of-Thought (CoT) Prompting

> **The Question:** "What is chain-of-thought (CoT) prompting, and when should you use it?"

---

## What They're Actually Testing

They want you to explain that CoT elicits intermediate reasoning steps, why this improves accuracy on complex tasks, and when it actually helps vs. hurts.

---

## The Technical Breakdown

### What Is CoT?

Chain-of-thought prompting instructs the model to **show its reasoning step by step** before producing a final answer:

```
Standard prompt:
"Roger has 5 tennis balls. He buys 2 cans of 3 balls each. How many balls?" → "11"

CoT prompt:
"Roger has 5 tennis balls. He buys 2 cans of 3 balls each. How many balls?
Let's think step by step."
→ "Roger starts with 5 balls. He buys 2 cans × 3 balls = 6 new balls. Total: 5 + 6 = 11"
```

### Why CoT Works

1. **Decomposes complexity** — Breaks a hard problem into manageable sub-problems
2. **Allocates compute** — More output tokens = more "thinking" compute at inference time
3. **Reduces errors** — Each step can be verified; errors are localized to one step
4. **Leverages pre-training** — Models saw step-by-step reasoning during training

### CoT Variants

| Variant | Description | When to Use |
|---------|-------------|-------------|
| **Zero-shot CoT** | "Let's think step by step" | Simple reasoning tasks |
| **Few-shot CoT** | Provide examples with reasoning steps | Complex domain-specific reasoning |
| **Auto-CoT** | Model generates its own reasoning examples | When you can't write good examples |
| **Self-consistency** | Generate N reasoning chains, majority vote | When accuracy is critical |

### When CoT Helps vs. Hurts

| ✅ CoT Helps | ❌ CoT Hurts |
|-------------|-------------|
| Math word problems | Simple factual recall |
| Multi-step reasoning | Classification tasks |
| Code debugging | Translation |
| Logic puzzles | Short creative text |
| Planning tasks | Sentiment analysis |

**Rule of thumb:** If a human would need scratch paper to solve it, use CoT. If a human answers instantly, skip it.

---

## Accuracy Check

| Claim | Verified? |
|-------|-----------|
| CoT improves accuracy on reasoning tasks | ✅ Wei et al. (2022), "Chain-of-Thought Prompting Elicits Reasoning" |
| "Let's think step by step" is effective zero-shot CoT | ✅ Kojima et al. (2022) |
| CoT is less effective for simple tasks | ✅ Wei et al. (2022), shown for small models and simple tasks |
| CoT effectiveness scales with model size | ✅ Emergent capability in models >~10B parameters |

---

## Scenario Examples

### Scenario 1: Math Problem Accuracy

A tutoring app asks GPT-4 to solve algebra problems. Without CoT: 68% accuracy. With "Show your work step by step": 92% accuracy. The reasoning steps also let the app pinpoint where a student went wrong by comparing their steps to the model's.

### Scenario 2: CoT Overhead in Production

A real-time classification API adds "Let's think step by step" to improve accuracy. Latency jumps from 200ms to 1.2s (the model generates 200 reasoning tokens before the answer). Accuracy improves only 2% (classification doesn't benefit much from CoT). The team removes CoT and uses few-shot examples instead — same accuracy, 6× faster.

---

## Follow-Up Questions

### Q1: "What is self-consistency, and how does it improve CoT?"

**Answer:** Generate N independent reasoning chains (with temperature > 0), extract the final answer from each, and take the **majority vote**. Different reasoning paths may have different intermediate errors, but they often converge on the correct final answer. Self-consistency (Wang et al., 2023) improves CoT accuracy by 5-15% on math benchmarks at the cost of N× more inference.

### Q2: "Does CoT work on small models?"

**Answer:** Poorly. CoT is an **emergent capability** that appears reliably in models with >10B parameters. Smaller models often generate plausible-sounding but logically incorrect reasoning chains, which can actually degrade accuracy compared to direct answering. Below 7B, use few-shot examples with answers rather than CoT.

### Q3: "How do you verify that CoT reasoning is correct?"

**Answer:** Use **process reward models** that evaluate each reasoning step (not just the final answer). Or implement programmatic verification: for math, parse and execute the intermediate calculations; for code, run unit tests on intermediate outputs. The reasoning chain is only valuable if it's actually correct — a fluent but wrong chain is dangerous.

---

<p align="center">
  <a href="../README.md">← Back to all questions</a>
</p>
