# Part 18: Meta-Prompts

> **The Question:** "What are meta-prompts, and how can they be used to generate prompts?"

---

## The Technical Breakdown

### What Are Meta-Prompts?

A meta-prompt is a **prompt that generates other prompts**. Instead of writing prompts manually, you describe the task and let an LLM create the optimal prompt:

```
Meta-prompt:
"I need a prompt for a customer support chatbot that handles refund requests for 
an e-commerce company. The bot should be empathetic, follow company refund policy 
(30-day window, receipt required), and escalate complex cases. 
Generate the optimal system prompt."

Output:
"You are a friendly customer support agent for ShopCo. Your primary role is to 
handle refund requests with empathy and efficiency. Follow these guidelines:
1. Acknowledge the customer's frustration before discussing policy
2. Verify: purchase date (within 30 days?) and receipt availability
3. If eligible: process the refund immediately
4. If ineligible: explain why clearly and offer alternatives
5. If complex (damaged goods, disputes): escalate to a human agent
..."
```

### Use Cases

| Use Case | How |
|----------|-----|
| **Prompt generation** | Describe task → LLM generates system prompt |
| **Prompt optimization** | "Improve this prompt for accuracy on these failing examples" |
| **Prompt translation** | Adapt prompts for different models (GPT-4 → Claude) |
| **A/B variant generation** | "Generate 5 variations of this prompt for testing" |

### OPRO (Optimization by PROmpting)

Google's OPRO uses an LLM to iteratively optimize prompts:
```
1. Start with a baseline prompt + eval scores
2. Ask the LLM: "Here are prompts and their scores. Generate a better prompt."
3. Evaluate the new prompt
4. Feed results back → repeat
```

---

## Scenario Examples
### A: A team needs prompts for 50 different customer intents. Instead of writing 50 prompts manually, a meta-prompt generates all 50 based on intent descriptions and company guidelines. Human review refines the top 10 highest-volume prompts.
### B: An OPRO-style optimizer improves a math prompt from 72% → 84% accuracy by discovering non-obvious phrasings: "Take a deep breath and work through this step by step" outperformed human-written instructions.

## Follow-Up Questions
### Q1: "Can LLMs really write better prompts than humans?"
**Answer:** Sometimes — OPRO and DSPy have found prompts that outperform human-designed ones by 5-15% on specific benchmarks. LLMs can explore phrasings humans wouldn't try. But humans are better at understanding context, constraints, and edge cases. Best approach: LLM generates candidates, humans select and refine.

### Q2: "How do you validate meta-prompt outputs?"
**Answer:** Always evaluate generated prompts on your eval dataset before deployment. Meta-prompts produce candidates, not solutions. A prompt that sounds good may not perform well — evaluation is mandatory.

### Q3: "What is self-refine applied to prompts?"
**Answer:** Generate a prompt → test it on examples → identify failures → ask the LLM to improve the prompt based on the failures → repeat. This is the meta-prompt + evaluation loop that converges on high-quality prompts with minimal human intervention.

---

<p align="center"><a href="../../README.md">← Back to all questions</a></p>
