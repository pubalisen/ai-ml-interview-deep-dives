# Part 3: What Is a Large Language Model (LLM)?

> **The Question:** "What is a Large Language Model (LLM), and how does it work?"

---

## What They're Actually Testing

The interviewer is checking if you can explain LLMs **mechanically** — not just "it predicts the next word." They want to hear about the training objective, the architecture, how scale creates capability, and what the model actually learns during pre-training. This separates users of LLMs from builders of LLM systems.

---

## The Technical Breakdown

### Definition

A Large Language Model (LLM) is a neural network with **billions of parameters**, trained on **massive text corpora**, using a **self-supervised objective** (typically next-token prediction) to learn the statistical structure of language. Once trained, it can generate coherent text, answer questions, write code, reason about problems, and perform tasks it was never explicitly trained for.

### The Three Pillars

```
LLM = Architecture (Transformer) + Data (Trillions of tokens) + Scale (Billions of parameters)
```

| Pillar | Detail |
|--------|--------|
| **Architecture** | Decoder-only Transformer (GPT family) or encoder-decoder (T5, BART). Most modern LLMs are decoder-only. |
| **Data** | Web crawls (Common Crawl), books, code (GitHub), Wikipedia, academic papers. Typically 1-15 trillion tokens. |
| **Scale** | 7B to 405B+ parameters. Scaling laws (Chinchilla, Kaplan et al.) show that model performance improves predictably with more parameters and data. |

### How It Works: The Training Objective

The core training objective is **next-token prediction** (also called causal language modeling):

```
Input:  "The capital of France is"
Target: "Paris"
```

During pre-training, the model processes trillions of (input, target) pairs. For each position in a sequence, it predicts the next token and computes the **cross-entropy loss** between its prediction and the actual token. Backpropagation adjusts the model's billions of parameters to minimize this loss.

What the model actually learns during this process:
- **Syntax and grammar** — word order, agreement, punctuation rules
- **Semantics** — word meaning, analogies, relationships
- **World knowledge** — facts, dates, entities, relationships
- **Reasoning patterns** — if-then logic, mathematical steps, code patterns
- **Structural templates** — email formats, essay structures, code conventions

### How It Works: Inference

At inference (when you prompt it), the model:

1. **Tokenizes** your input into subword tokens
2. **Embeds** each token into a high-dimensional vector
3. **Passes** the sequence through transformer layers (self-attention + feed-forward networks)
4. **Outputs** a probability distribution over its entire vocabulary for the next token
5. **Samples** one token from this distribution
6. **Appends** that token to the context and **repeats** from step 3

This is **autoregressive generation** — one token at a time, each conditioned on everything before it.

### What Makes It "Large"

| Model | Parameters | Training Data |
|-------|-----------|---------------|
| GPT-2 (2019) | 1.5B | 40GB text |
| GPT-3 (2020) | 175B | 570GB text |
| LLaMA 2 (2023) | 7B–70B | 2T tokens |
| GPT-4 (2023) | Undisclosed (rumored ~1.8T MoE) | Undisclosed |
| LLaMA 3.1 (2024) | 8B–405B | 15T+ tokens |

"Large" refers to both the parameter count and the training data volume. The key insight from scaling laws: **there is a predictable, log-linear relationship between compute, data, model size, and performance**.

---

## Accuracy Check

| Claim | Verified? |
|-------|-----------|
| LLMs use next-token prediction as the primary training objective | ✅ GPT series, LLaMA series |
| Decoder-only Transformers dominate modern LLMs | ✅ GPT, LLaMA, Claude, Gemini all decoder-only |
| Scaling laws predict performance improvements | ✅ Kaplan et al. (2020), Chinchilla (Hoffmann et al., 2022) |
| LLMs learn world knowledge, reasoning, and structure from pre-training | ✅ Probing studies (Petroni et al., 2019; Clark et al., 2019) |
| Inference is autoregressive — one token at a time | ✅ Core Transformer architecture constraint |

---

## Scenario Examples

### Scenario 1: Building a Legal Document Analyzer

Your company wants an AI that reads 50-page legal contracts and extracts key clauses (termination, liability, payment terms). 

**How the LLM works here:** The model doesn't "understand" law — it has encoded statistical patterns from millions of legal documents in its training data. When you prompt it with a contract and ask "Extract the termination clause," the model's attention mechanisms identify which tokens in the document are contextually relevant to "termination" and generate a response conditioned on those high-attention-weight regions. If the contract uses unusual legal language not well-represented in training data, the model may hallucinate or miss clauses — which is why you'd pair it with RAG (retrieval from a legal database) and human review.

### Scenario 2: Code Generation at Scale

A developer asks an LLM to write a Python function that merges two sorted arrays. The model generates correct code because during pre-training, it processed millions of code repositories containing this exact pattern. But the model doesn't "know" algorithms — it has learned that when the context contains "merge two sorted arrays" + "Python," the highest-probability token sequence corresponds to the two-pointer merge pattern. If you ask it to implement a novel algorithm it's never seen, performance degrades because it lacks training examples to condition on.

---

## Follow-Up Questions

### Q1: "If LLMs just predict the next token, how can they perform reasoning?"

**Answer:** Reasoning in LLMs is an **emergent property of pattern matching at scale**. During pre-training on trillions of tokens — including textbooks, proofs, code, and step-by-step explanations — the model learns sequential patterns that *look like* reasoning. When you use chain-of-thought prompting, you force the model to generate intermediate reasoning tokens, which physically alter the context window and bias subsequent token predictions toward logically consistent conclusions. The model doesn't "reason" in the human sense — it generates the most probable continuation of a reasoning pattern. This is why LLMs can solve math problems they've seen patterns of, but fail on genuinely novel logical puzzles.

### Q2: "What's the difference between pre-training, fine-tuning, and alignment?"

**Answer:** 
- **Pre-training** is the initial, expensive phase where the model learns language by predicting the next token on trillions of tokens of raw text. This produces a base model that can complete text but isn't useful as an assistant.
- **Fine-tuning** (SFT — Supervised Fine-Tuning) trains the base model on curated (instruction, response) pairs to teach it to follow instructions and be helpful.
- **Alignment** (RLHF/DPO) further adjusts the model using human preference data to make it safe, honest, and harmless. A reward model learns what humans prefer, and the LLM is optimized to maximize that reward.

The pipeline: `Pre-training → SFT → RLHF/DPO = Chat model`

### Q3: "How do you choose between using a large model (e.g., 70B) vs. a small model (e.g., 7B)?"

**Answer:** The decision depends on four factors:
1. **Task complexity** — Simple classification or extraction can work with 7B models. Complex multi-step reasoning, creative writing, or code generation benefits from 70B+.
2. **Latency requirements** — 7B models are 5-10x faster at inference. If you need sub-200ms responses, smaller models are necessary.
3. **Cost** — Serving a 70B model requires 4-8 GPUs; a 7B model can run on a single GPU or even CPU with quantization.
4. **Fine-tuning opportunity** — A fine-tuned 7B model on your specific domain can often match or beat a general-purpose 70B model on that domain, at a fraction of the serving cost.

Rule of thumb: Start with the smallest model that meets your quality bar, then scale up only if needed.

---

## Further Reading

- [Scaling Laws for Neural Language Models (Kaplan et al., 2020)](https://arxiv.org/abs/2001.08361)
- [Training Compute-Optimal Large Language Models — Chinchilla (Hoffmann et al., 2022)](https://arxiv.org/abs/2203.15556)
- [Language Models are Unsupervised Multitask Learners — GPT-2 (Radford et al., 2019)](https://cdn.openai.com/better-language-models/language_models_are_unsupervised_multitask_learners.pdf)

---

<p align="center">
  <a href="../../README.md">← Back to all questions</a>
</p>
