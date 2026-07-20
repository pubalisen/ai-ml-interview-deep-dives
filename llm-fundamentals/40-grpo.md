# Part 41: GRPO (Group Relative Policy Optimization)

> **The Question:** "Explain Group Relative Policy Optimization (GRPO)."

---

## What They're Actually Testing

They want you to explain how GRPO (used by DeepSeek) eliminates the value model from PPO, using group-level comparisons instead.

---

## The Technical Breakdown

### GRPO vs PPO

PPO requires a **value model** (critic) to estimate advantages. GRPO eliminates it by generating **multiple responses per prompt** and computing advantages **relative to the group**:

```
For each prompt:
  Generate G responses: [y₁, y₂, ..., y_G]
  Score each: [r₁, r₂, ..., r_G]
  Compute advantage: Â_i = (r_i - mean(r)) / std(r)   ← group-relative
```

A response is "good" if it scores above the group average and "bad" if below. No separate value model needed.

| Aspect | PPO | GRPO |
|--------|-----|------|
| **Value model** | ✅ Required (separate neural network) | ❌ Not needed |
| **Advantage estimate** | From value function | Relative to group of responses |
| **Memory** | 4 models in memory | 2 models (policy + reference) |
| **Used by** | InstructGPT, ChatGPT | DeepSeek-R1, DeepSeek-V2 |

### Why GRPO Matters

By removing the value model, GRPO reduces the GPU memory requirements of RLHF training by ~25-30%. It's simpler to implement and showed strong results in DeepSeek's reasoning models.

---

## Accuracy Check

| Claim | Verified? |
|-------|-----------|
| GRPO removes the value model from PPO | ✅ Shao et al. (2024), DeepSeek-Math |
| Advantages are computed relative to group mean | ✅ GRPO algorithm specification |
| Used by DeepSeek-R1 | ✅ DeepSeek-R1 technical report |

---

## Scenario Examples

### Scenario 1: A team training a math reasoning model generates 8 solutions per problem, scores them by correctness, and uses GRPO to reinforce correct solutions while suppressing incorrect ones — no reward model needed, just a verifiable answer checker.

### Scenario 2: For code generation, GRPO generates 16 code completions per prompt, runs unit tests to score them (pass=1, fail=0), and optimizes toward passing solutions. The group-relative advantage naturally handles varying difficulty across prompts.

---

## Follow-Up Questions

### Q1: "How many responses per group (G) are typically needed?"

**Answer:** Typically G=8-64. Larger G gives more stable advantage estimates but requires more generation compute per training step. G=16 is a common sweet spot.

### Q2: "Can GRPO work with binary rewards (pass/fail)?"

**Answer:** Yes — and this is where GRPO shines. For verifiable tasks (math, code), binary rewards from automated checkers work perfectly. The group normalizes these binary signals into continuous advantages.

### Q3: "How does GRPO compare to DPO?"

**Answer:** DPO requires offline preference data. GRPO generates its own training data online (on-policy). GRPO is better for tasks with verifiable rewards (math, code) where you can score responses automatically. DPO is better for subjective tasks (helpfulness, tone) where human preferences are needed.

---

<p align="center">
  <a href="../README.md">← Back to all questions</a>
</p>
