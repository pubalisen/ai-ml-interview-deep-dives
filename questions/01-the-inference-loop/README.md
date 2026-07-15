# Part 1: The Inference Loop

> **The Question:** "A user prompts an LLM with: *'Write a polite email to my boss asking for Friday off.'* Walk me through exactly what happens under the hood at inference time to generate that response."

---

## What They're Actually Testing

The interviewer is checking if you deeply understand **autoregressive generation** and the lack of global state planning. They want to ensure you don't anthropomorphize the model into something that "thinks" or drafts an entire response before outputting it.

This question separates candidates who have used LLM APIs from candidates who understand what's happening inside them.

---

## The Technical Breakdown

When that prompt hits the API, the model doesn't formulate a complete email. It executes a **forward pass to predict a single token.**

### Step 1: Tokenization & Embedding

The input string is broken into tokens (subword units via BPE or SentencePiece), mapped to dense vectors, and combined with **positional embeddings** so the model understands sequence order.

```
"Write a polite email..." → [15043, 257, 30289, 3053, ...]
                           → [embed_15043 + pos_0, embed_257 + pos_1, ...]
```

The model doesn't see words. It sees a matrix of floating-point vectors.

### Step 2: The Forward Pass

These embeddings are passed through the transformer blocks (typically 32–128 layers in modern LLMs). The **self-attention mechanisms** calculate the contextual relevance of every token in the prompt against every other token.

Each attention head learns different relationships:
- One head might track syntactic dependencies ("email" → "polite")
- Another might track positional patterns ("boss" → subject of the email)
- Another might attend to formatting cues ("write" → task instruction)

### Step 3: Logits to Probabilities

The final layer outputs a vector of **logits** — one value for every token in the model's vocabulary (typically 32K–256K tokens). A softmax function converts this into a probability distribution for what the **first new token** should be.

```
P("Subject") = 0.12
P("Dear")    = 0.09
P("Hi")      = 0.07
P("I")       = 0.05
...
```

### Step 4: Sampling

The system **samples** from this distribution, modulated by decoding parameters:

| Parameter | What It Does | Typical Range |
|-----------|-------------|---------------|
| **Temperature** | Controls randomness. Lower = more deterministic, higher = more creative | 0.0 – 2.0 |
| **Top-k** | Only considers the top k most probable tokens | 10 – 100 |
| **Top-p (nucleus)** | Only considers tokens whose cumulative probability mass reaches p | 0.9 – 0.99 |

Let's say it selects **"Subject"**.

### Step 5: Autoregression

That single "Subject" token is **appended to the original prompt**. The entire forward pass runs again on this newly expanded sequence to predict the token after that (likely ":"). 

```
Pass 1: [prompt]              → "Subject"
Pass 2: [prompt + "Subject"]  → ":"
Pass 3: [prompt + "Subject:"] → " Request"
Pass 4: ...
```

This loop repeats — one token at a time — until the model outputs an **`<EOS>` (End of Sequence) token** or hits the max token limit.

> **Key insight:** The model never "sees" the complete email. It builds it one token at a time, each token conditioned entirely on everything before it.

---

## Why This Architecture Matters in Production

Understanding this sequential limitation explains the most common LLM failure modes:

### Error Compounding

Because earlier tokens strictly condition the probability distribution of later tokens, an early sampling anomaly (e.g., outputting "Hey" instead of "Dear") **mathematically forces** the rest of the generation down a casual, potentially unwanted trajectory. The model cannot recover gracefully from an early misstep.

### Lack of Lookahead

The model cannot backtrack, edit, or revise. It must resolve the sequence based solely on the current context window. There is no "draft → review → finalize" process happening internally.

### Context Window Pressure

As the generated sequence grows, older tokens in long prompts may get less attention weight (depending on position encoding scheme), which is why LLMs tend to lose coherence in very long outputs.

---

## The Curveball Follow-Up

> *"If it's strictly predicting the next token sequentially, how does it output a logically structured, multi-paragraph email?"*

### The Answer

Coherence is an **emergent property of representation learning**. During pre-training on trillions of tokens and alignment (RLHF/SFT), the model maps high-dimensional representations of human language — including structural templates, grammar, and logical flow.

When the model generates "Subject:", the probability distribution for the next token is already **heavily biased** toward email-structured continuations because the model has encoded millions of examples of that pattern.

### Why Chain-of-Thought Works

This autoregressive constraint is exactly why **Chain-of-Thought (CoT) prompting** works. By forcing the model to generate intermediate reasoning tokens first, you are **physically altering the context window**. 

When the model finally generates its answer, that final probability distribution is heavily conditioned by the logical steps it just appended to its own context, rather than attempting a zero-shot leap.

```
Without CoT: [question] → [answer]              ← one forward-pass leap
With CoT:    [question] → [step1, step2, step3] → [answer]  ← conditioned on reasoning
```

The reasoning tokens aren't for the model to "think" — they're for the model to **build a better context window** for the final prediction.

---

## Key Takeaways

1. **LLMs generate one token at a time** — there is no internal planning or drafting
2. **Each token is conditioned on the full preceding context** — early errors compound
3. **Coherence emerges from training, not from architecture** — the model learned structure, it doesn't enforce it
4. **CoT works by enriching the context window** — not by enabling "reasoning"
5. **Sampling parameters are engineering levers** — temperature, top-k, and top-p directly control the quality-creativity tradeoff

---

## Further Reading

- [Attention Is All You Need (Vaswani et al., 2017)](https://arxiv.org/abs/1706.03762)
- [Language Models are Few-Shot Learners (GPT-3 paper)](https://arxiv.org/abs/2005.14165)
- [Chain-of-Thought Prompting (Wei et al., 2022)](https://arxiv.org/abs/2201.11903)
- [Nucleus Sampling (Holtzman et al., 2020)](https://arxiv.org/abs/1904.09751)

---

<p align="center">
  <a href="../../README.md">← Back to all questions</a>
</p>
