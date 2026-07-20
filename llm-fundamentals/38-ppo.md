# Part 39: Proximal Policy Optimization (PPO)

> **The Question:** "Explain Proximal Policy Optimization (PPO)."

---

## What They're Actually Testing

They want you to explain PPO's role in RLHF — how a reward model trains the LLM to align with human preferences, and why PPO specifically is used.

---

## The Technical Breakdown

### PPO in the RLHF Pipeline

```
Step 1: Train a reward model on human preference data (which response is better?)
Step 2: Use PPO to optimize the LLM to maximize the reward model's score
Step 3: Add a KL penalty to prevent the LLM from diverging too far from the base model
```

### How PPO Works

PPO is a **policy gradient** reinforcement learning algorithm. In the LLM context:
- **Policy** = the LLM (generates text)
- **Action** = each generated token
- **Reward** = reward model score for the complete response
- **Environment** = the conversation/prompt

PPO updates the policy to increase the probability of high-reward responses while constraining updates to be small (the "proximal" part):

```
Loss = -min(r(θ) × Â, clip(r(θ), 1-ε, 1+ε) × Â)

r(θ) = π_new(a|s) / π_old(a|s)   # probability ratio
Â = advantage estimate              # how much better than expected
ε = clip range (typically 0.2)      # limits update size
```

The clipping prevents catastrophically large policy updates — the model can't change too much in one step.

### Why PPO for LLMs

| Property | Why It Matters |
|----------|---------------|
| **Clipped updates** | Prevents reward hacking (model exploiting reward model loopholes) |
| **Sample efficiency** | Reuses experience for multiple gradient steps |
| **Stability** | Converges reliably with proper hyperparameters |
| **KL penalty** | `reward - β × KL(π_new || π_ref)` keeps model close to supervised fine-tuned version |

---

## Accuracy Check

| Claim | Verified? |
|-------|-----------|
| PPO is used in RLHF for aligning LLMs | ✅ InstructGPT (Ouyang et al., 2022) |
| PPO uses clipped probability ratios | ✅ Schulman et al. (2017) |
| KL penalty prevents divergence from base model | ✅ Standard RLHF implementation |

---

## Scenario Examples

### Scenario 1: An LLM trained with PPO-based RLHF learns to give long, detailed responses because the reward model slightly prefers longer answers. The KL penalty prevents this from going too far — without it, the model would generate infinitely verbose responses to maximize reward (reward hacking).

### Scenario 2: A team fine-tunes a medical chatbot with PPO. The reward model is trained on doctor-preference data. After RLHF, the model refuses to diagnose (appropriately cautious) and recommends seeing a professional — behavior learned from the reward model, not from the pre-training data.

---

## Follow-Up Questions

### Q1: "What are the main challenges of PPO-based RLHF?"

**Answer:** (1) Requires training 4 models simultaneously (policy, reference, reward, value), (2) reward model quality bottlenecks the entire system, (3) training is unstable and requires careful hyperparameter tuning, (4) reward hacking — the model finds shortcuts to maximize reward without being genuinely helpful.

### Q2: "What is the reward model?"

**Answer:** A separate model trained on human preference data: given two responses to the same prompt, which is better? It learns to assign scalar scores that predict human preference. This reward signal then trains the LLM via PPO. The reward model is typically the same size as the LLM or smaller.

### Q3: "Why has DPO been replacing PPO?"

**Answer:** DPO eliminates the need for a separate reward model and RL training loop, making alignment much simpler. PPO requires training 4 models with complex RL dynamics. DPO achieves comparable alignment quality with a single supervised fine-tuning step. See the DPO deep-dive.

---

<p align="center">
  <a href="../README.md">← Back to all questions</a>
</p>
