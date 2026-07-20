# Part 45: Small Language Models (SLMs)

> **The Question:** "What are Small Language Models (SLMs)?"

---

## What They're Actually Testing

They want you to discuss the trend of capable smaller models (1-7B) and when they're preferable to larger models in production.

---

## The Technical Breakdown

### What Are SLMs?

Small Language Models are Transformer-based models with **1B-7B parameters** that are optimized for efficiency while retaining strong performance on focused tasks. Examples: Phi-3 (3.8B), Gemma 2 (2B), LLaMA 3.2 (1B/3B), Mistral 7B.

### Why SLMs Matter

| Property | Large LLM (70B+) | SLM (1-7B) |
|----------|------------------|------------|
| **Inference speed** | 10-50 tok/s | 100-500 tok/s |
| **Memory** | 140GB+ (FP16) | 2-14GB (FP16) |
| **Hardware** | Multi-GPU cluster | Single GPU or laptop |
| **Cost/request** | ~$0.01-0.10 | ~$0.0001-0.001 |
| **On-device** | ❌ Impossible | ✅ Phone/laptop viable |
| **General reasoning** | ✅ Excellent | ⚠️ Limited |
| **Narrow tasks** | Overkill | ✅ Excellent after fine-tuning |

### When SLMs Win

1. **High volume, narrow tasks** — classification, extraction, routing (where quality requirements are met)
2. **On-device/edge** — mobile apps, embedded systems, offline scenarios
3. **Cost-sensitive** — 100× cheaper per request than frontier models
4. **Low latency** — sub-50ms response times for real-time applications
5. **Privacy** — data never leaves the device

### The Quality Gap Is Shrinking

Modern SLMs achieve surprising quality through:
- **High-quality training data** (curated, filtered, synthetic)
- **Knowledge distillation** from larger models
- **Architecture optimizations** (GQA, SwiGLU, RoPE)
- **Training on more tokens** than scaling laws suggest (over-training)

---

## Accuracy Check

| Claim | Verified? |
|-------|-----------|
| SLMs (1-7B) can run on single GPU or laptop | ✅ Quantized 7B fits in 4GB VRAM |
| Phi-3 (3.8B) approaches GPT-3.5 on some benchmarks | ✅ Microsoft Phi-3 technical report |
| Over-training improves small model quality | ✅ Chinchilla-suboptimal but effective |

---

## Scenario Examples

### Scenario 1: A mobile keyboard app uses a 1B SLM for on-device autocomplete. Running locally means zero latency (no API call), zero cost per prediction, and full privacy (keystrokes never leave the phone). Quality is lower than GPT-4 but more than sufficient for next-word prediction.

### Scenario 2: An API gateway uses a 3B model as a **router** — classifying incoming requests by complexity (simple/medium/complex) and routing them to appropriate backend models. The router needs to be fast and cheap, not brilliant. An SLM handles this at 500 requests/second on a single GPU.

---

## Follow-Up Questions

### Q1: "How small can you go before quality degrades unacceptably?"

**Answer:** For narrow, well-defined tasks (sentiment, classification, extraction): 0.5-1B is often sufficient after fine-tuning. For conversational AI: 3B is the minimum for coherent responses. For reasoning tasks: 7B with CoT is the practical floor. Below 1B, models struggle with multi-step reasoning.

### Q2: "How does quantization extend SLM deployment?"

**Answer:** A 7B model at FP16 = 14GB. At INT4 quantization = 3.5GB — fits on a smartphone. Quality loss from INT4 quantization is typically 1-3% on benchmarks. Combined with SLM size, this enables genuine on-device LLM inference.

### Q3: "Is it better to use a fine-tuned SLM or a prompted large model?"

**Answer:** If you have task-specific data, a fine-tuned SLM often matches or beats a prompted large model on that specific task while being 100× cheaper. The large model wins when you need generalization across diverse tasks or when you don't have fine-tuning data. Rule of thumb: use prompted large models for prototyping, then distill/fine-tune into an SLM for production.

---

<p align="center">
  <a href="../README.md">← Back to all questions</a>
</p>
