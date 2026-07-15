# Part 25: Open-Source vs. Closed-Source LLMs

> **The Question:** "What is the difference between open-source and closed-source LLMs? When would you choose one over the other?"

---

## What They're Actually Testing

The interviewer wants practical engineering judgment — not a philosophical debate about openness. They want to hear about trade-offs in cost, control, latency, privacy, and compliance.

---

## The Technical Breakdown

### Comparison

| Dimension | Open-Source (LLaMA, Mistral, Qwen) | Closed-Source (GPT-4, Claude, Gemini) |
|-----------|-------------------------------------|---------------------------------------|
| **Access** | Download weights, run anywhere | API access only |
| **Cost model** | GPU infrastructure cost (fixed) | Per-token pricing (variable) |
| **Customization** | Full fine-tuning, LoRA, custom tokenizer | Limited fine-tuning via API |
| **Privacy** | Data stays on your infrastructure | Data sent to third-party API |
| **Latency control** | Full control over hardware/batching | Dependent on provider's infrastructure |
| **Quality ceiling** | Generally lower (7B-405B) | Generally higher (GPT-4, Claude 3.5) |
| **Maintenance** | You manage infrastructure, updates | Provider manages everything |
| **Vendor lock-in** | None | High — tied to provider's API |

### When to Choose Open-Source

1. **Data privacy requirements** — Healthcare (HIPAA), finance (SOX), government (FedRAMP) where data can't leave your infrastructure
2. **High volume, predictable workloads** — 10M+ requests/month where per-token pricing is prohibitive
3. **Low latency requirements** — Self-hosted with dedicated GPUs gives consistent sub-100ms response times
4. **Customization needs** — Full fine-tuning, custom tokenizer, architecture modifications
5. **Offline/edge deployment** — No internet dependency

### When to Choose Closed-Source

1. **Maximum quality** — Tasks requiring state-of-the-art reasoning (complex code, nuanced analysis)
2. **Fast prototyping** — API call vs. setting up GPU infrastructure
3. **Low/variable volume** — Pay only for what you use
4. **No ML infrastructure team** — Don't want to manage GPUs, model updates, scaling

---

## Accuracy Check

| Claim | Verified? |
|-------|-----------|
| Open-source models allow full weight access | ✅ LLaMA, Mistral releases include weights |
| Closed-source models generally have higher quality ceiling | ✅ GPT-4, Claude 3.5 lead benchmarks (as of 2024) |
| Self-hosting eliminates per-token costs | ✅ Fixed GPU cost regardless of usage |
| Data privacy is a key driver for open-source adoption | ✅ Industry surveys, enterprise AI reports |

---

## Scenario Examples

### Scenario 1: Healthcare Startup

A telehealth platform needs LLM-powered medical note summarization. HIPAA prohibits sending patient data to external APIs without a BAA. They deploy LLaMA 3.1 70B on their own HIPAA-compliant infrastructure. Cost: $15K/month in GPU hosting. Equivalent API cost at their volume: $45K/month. They save 67% and maintain full compliance.

### Scenario 2: Early-Stage Startup

A 3-person startup building an AI writing assistant needs the best quality output. They use GPT-4o via API — no infrastructure to manage, no ML engineer needed, and they pay ~$500/month at their current scale. When they hit 100K users and API costs reach $20K/month, they'll evaluate switching to a fine-tuned open-source model.

---

## Follow-Up Questions

### Q1: "What does 'open-source' really mean for LLMs?"

**Answer:** There's a spectrum. **Truly open-source** (Apache 2.0, MIT) means weights, training code, and data are freely available with no restrictions (e.g., BLOOM, OLMo). **Open-weight** means weights are available but with usage restrictions (LLaMA's community license prohibits use by companies with 700M+ monthly users). **Open-weight, closed-data** is most common — you get the weights but not the training data or training code. Most "open-source" LLMs are actually open-weight.

### Q2: "How do you decide the break-even point between API and self-hosting?"

**Answer:** Calculate: `monthly_api_cost = total_tokens × price_per_token`. Compare against: `self_hosting_cost = gpu_rental + engineering_time + maintenance`. Rule of thumb: below 1M tokens/day, APIs are cheaper. Above 10M tokens/day, self-hosting is almost always cheaper. The crossover depends on your GPU pricing (cloud vs on-prem), whether you have ML engineers, and how much you value infrastructure simplicity.

### Q3: "Can you mix open and closed-source models in production?"

**Answer:** Yes — this is called **model routing**. Route simple requests (classification, extraction) to a fast, cheap open-source model (7B) and complex requests (reasoning, creative writing) to a powerful closed-source API (GPT-4). This optimizes cost while maintaining quality ceilings. Semantic routers classify query complexity and route accordingly. Many production systems use this hybrid approach.

---

<p align="center">
  <a href="../../README.md">← Back to all questions</a>
</p>
