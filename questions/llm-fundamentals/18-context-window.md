# Part 19: The Context Window

> **The Question:** "What is the context window in LLMs, and why does it matter?"

---

## What They're Actually Testing

The interviewer wants you to explain the context window as an **engineering constraint**, not just a number. They want to hear about memory, compute costs, and the real-world implications for application design.

---

## The Technical Breakdown

### What Is the Context Window?

The context window is the **maximum number of tokens** the model can process in a single forward pass. Everything the model can "see" — system prompt, conversation history, retrieved documents, and the generated response — must fit within this window.

```
┌─────────────────── Context Window (e.g., 128K tokens) ───────────────────┐
│ [System Prompt] [Chat History] [RAG Context] [User Message] [Response]  │
└──────────────────────────────────────────────────────────────────────────┘
```

### Context Window Sizes

| Model | Context Window | Approximate Word Equivalent |
|-------|---------------|---------------------------|
| GPT-3.5 | 4K / 16K tokens | ~3K / ~12K words |
| GPT-4o | 128K tokens | ~96K words |
| Claude 3.5 Sonnet | 200K tokens | ~150K words |
| Gemini 1.5 Pro | 1M / 2M tokens | ~750K / 1.5M words |
| LLaMA 3.1 | 128K tokens | ~96K words |

### Why It's Limited

The context window is limited by three factors:

1. **Quadratic attention cost** — Self-attention computes an n×n matrix. At 128K tokens, that's 16.4 billion entries per head per layer.

2. **KV cache memory** — During generation, the model caches Key and Value matrices for all previous tokens. For a 70B model at 128K context: KV cache ≈ 40-80GB of GPU memory.

3. **Training data** — Models must be trained on sequences of at least the target context length. Longer training sequences require more memory and compute.

### Why It Matters for AI Engineers

```
Common failure modes:
├── Context overflow → Model silently drops oldest tokens
├── "Lost in the middle" → Model ignores information in middle of long context
├── Cost explosion → Longer contexts = more tokens = higher API cost
└── Latency increase → More tokens = slower prefill phase
```

---

## Accuracy Check

| Claim | Verified? |
|-------|-----------|
| Context window defines max tokens per forward pass | ✅ Fundamental architecture constraint |
| Attention is O(n²) in context length | ✅ Vaswani et al. (2017) |
| KV cache grows linearly with context length | ✅ Confirmed in vLLM, TensorRT-LLM docs |
| "Lost in the middle" is a documented phenomenon | ✅ Liu et al. (2023), "Lost in the Middle" |

---

## Scenario Examples

### Scenario 1: RAG Context Management

A legal AI system retrieves 20 relevant contract clauses (each ~500 tokens = 10K total) plus the system prompt (2K) and conversation history (5K). With a 16K context window, only 1K tokens remain for the response. The team must either truncate retrieval results, compress conversation history, or upgrade to a model with a larger context window. This is a daily constraint in production RAG systems.

### Scenario 2: Multi-Turn Chatbot Degradation

A customer support chatbot works well for 5-turn conversations but starts hallucinating at turn 15. Each turn averages 200 tokens (user + assistant), so 15 turns = 6K tokens. With the system prompt (2K) and tool definitions (3K), the context is 11K tokens. On turn 15, the system silently truncates early conversation turns, and the model loses context about the customer's original issue. Fix: implement conversation summarization — compress older turns into a summary to preserve critical information.

---

## Follow-Up Questions

### Q1: "How does the 'lost in the middle' problem affect long-context models?"

**Answer:** Liu et al. (2023) showed that even models with large context windows perform worse when relevant information is placed in the **middle** of the context. Models attend most strongly to information at the **beginning** (primacy bias) and **end** (recency bias) of the context. In a 128K context window, information at positions 40K-80K may receive significantly less attention. Mitigations: place critical information at the beginning or end, use structured formatting (headers, bullet points) to create attention anchors, or use retrieval to place relevant information closest to the query.

### Q2: "How do you estimate context window usage for cost planning?"

**Answer:** Total tokens per request = input tokens (prompt + context) + output tokens (response). For cost: `cost = (input_tokens × input_price) + (output_tokens × output_price)`. Key insight: input tokens are typically cheaper than output tokens (e.g., GPT-4o charges ~$2.50/M input, ~$10/M output). To optimize: minimize system prompt length, compress conversation history, limit RAG retrieval to top-k relevant chunks, and set max_tokens for responses.

### Q3: "What's the difference between context window and training sequence length?"

**Answer:** The **training sequence length** is the maximum sequence length used during pre-training. The **context window** at inference can potentially be longer if the model supports length extrapolation (via RoPE scaling, YaRN, etc.). However, quality typically degrades for lengths beyond what the model was trained on. A model trained on 8K sequences but serving 32K via RoPE interpolation will handle 32K contexts, but its performance on positions 8K-32K won't be as strong as on positions 0-8K.

---

<p align="center">
  <a href="../../README.md">← Back to all questions</a>
</p>
