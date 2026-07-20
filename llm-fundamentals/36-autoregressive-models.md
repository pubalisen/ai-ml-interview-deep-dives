# Part 37: Autoregressive Models

> **The Question:** "What are Autoregressive Models?"

---

## What They're Actually Testing

They want you to explain the core generation paradigm: predicting one token at a time, conditioned on all previous tokens, and why this sequential process is both powerful and limiting.

---

## The Technical Breakdown

### Definition

An autoregressive model generates output **one token at a time**, where each token is conditioned on all previously generated tokens:

```
P(x₁, x₂, ..., xₙ) = P(x₁) × P(x₂|x₁) × P(x₃|x₁,x₂) × ... × P(xₙ|x₁,...,xₙ₋₁)
```

This factorization means the model can only look **backward** — it never sees future tokens during generation.

### The Generation Loop

```python
prompt = "The capital of France is"
tokens = tokenize(prompt)

while not done:
    logits = model(tokens)           # Forward pass
    next_token = sample(logits[-1])  # Sample from last position's distribution
    tokens.append(next_token)        # Append to context
    if next_token == EOS: break      # Stop at end-of-sequence
```

### Strengths and Limitations

| Strength | Limitation |
|----------|-----------|
| Natural fit for text generation | Sequential — can't parallelize generation |
| Each token conditions on full history | Slow — one forward pass per output token |
| Well-understood training objective | Can't "look ahead" or plan |
| Enables coherent long-form generation | Errors compound — one bad token affects everything after |

### All Modern LLMs Are Autoregressive

GPT-4, Claude, LLaMA, Gemini — every major chat model generates text autoregressively. The alternatives (parallel generation, diffusion LMs) are actively researched but not yet competitive for general-purpose language generation.

---

## Accuracy Check

| Claim | Verified? |
|-------|-----------|
| Autoregressive factorizes joint probability into conditionals | ✅ Standard probability chain rule |
| All major LLMs are autoregressive | ✅ GPT, Claude, LLaMA, Gemini |
| Generation is inherently sequential | ✅ Each token depends on previous ones |

---

## Scenario Examples

### Scenario 1: Error Propagation — A code generation model outputs `def calculate(x, y:` (missing closing parenthesis). Every subsequent token is conditioned on this malformed context, causing cascading syntax errors. The autoregressive nature means the model can't go back and fix the parenthesis.

### Scenario 2: Why Streaming Works — Because tokens are generated one at a time, they can be sent to the user **as they're produced**. This is why ChatGPT shows a "typing" effect — each token is streamed via SSE as soon as the model generates it, not buffered until complete.

---

## Follow-Up Questions

### Q1: "Can autoregressive models generate in parallel?"

**Answer:** Not natively. Speculative decoding approximates parallelism: a small draft model generates k candidate tokens, then the large model verifies them in one parallel forward pass. If the drafts are correct, you get k tokens for the cost of ~1 large model pass. But the fundamental generation is still autoregressive — just verified in batches.

### Q2: "What are non-autoregressive alternatives?"

**Answer:** Non-autoregressive models (NAR) generate all tokens simultaneously. Examples: Mask-predict (iteratively refine masked tokens), diffusion language models (denoise from random tokens). NAR models are faster but currently lower quality for open-ended generation. They work better for fixed-length tasks like translation where the output length is predictable.

### Q3: "Why can't autoregressive models plan ahead?"

**Answer:** Each token is sampled without knowledge of what will follow. The model can't "see" that the current word choice will lead to an incoherent sentence later. Chain-of-thought prompting partially addresses this by generating reasoning tokens first, giving the model more context before committing to a final answer. But true planning would require non-autoregressive or search-based generation (like beam search or tree-of-thought).

---

<p align="center">
  <a href="../README.md">← Back to all questions</a>
</p>
