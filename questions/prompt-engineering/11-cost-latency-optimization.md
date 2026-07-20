# Part 11: Optimizing Prompts for Cost and Latency

> **The Question:** "How do you optimize prompts for cost and latency?"

---

## The Technical Breakdown

### Cost Drivers

`Total cost = (input_tokens × input_price) + (output_tokens × output_price)`

| Optimization | Technique | Impact |
|-------------|-----------|--------|
| **Reduce input tokens** | Remove verbose instructions, compress few-shot examples | 20-50% cost reduction |
| **Reduce output tokens** | Set max_tokens, request concise responses | 30-60% cost reduction |
| **Use cheaper models** | Route simple queries to smaller/cheaper models | 5-20× cost reduction |
| **Cache responses** | Semantic caching for repeated/similar queries | 80-95% cost reduction for cache hits |
| **Batch requests** | Group multiple queries into one prompt | 30-50% cost reduction |

### Latency Drivers

```
Total latency = TTFT (time to first token) + (output_tokens × time_per_token)
```

| Optimization | Technique | Impact |
|-------------|-----------|--------|
| **Shorter prompts** | Fewer input tokens = faster prefill | Linear improvement |
| **Fewer output tokens** | Shorter responses = less decode time | Linear improvement |
| **Streaming** | Return tokens as generated | Better perceived latency |
| **Parallel calls** | Independent sub-tasks in parallel | Near-linear speedup |
| **Prompt caching** | Reuse KV cache for shared prompt prefixes | 50-80% TTFT reduction |

### Practical Strategies

1. **Tiered model routing** — Simple queries → GPT-4o-mini ($0.15/M), complex → GPT-4o ($2.50/M)
2. **Prompt compression** — LLMLingua compresses prompts 2-5× with minimal quality loss
3. **Prefix caching** — Long system prompts cached across requests (Anthropic, vLLM)
4. **Remove unnecessary CoT** — Skip reasoning steps for simple tasks

---

## Scenario Examples
### A: A classification API processes 1M requests/day. Switching from GPT-4o ($2.50/M tokens) to GPT-4o-mini ($0.15/M) for the 80% of queries that are "easy" saves $3,600/day with < 2% accuracy loss.
### B: A RAG system has a 2000-token system prompt. Anthropic's prompt caching caches this prefix, reducing TTFT from 800ms to 200ms and input cost by 90% for the cached portion.

## Follow-Up Questions
### Q1: "What is semantic caching?"
**Answer:** Instead of exact-match caching, embed queries and cache responses for semantically similar queries. "What's the weather in NYC?" and "NYC weather today?" hit the same cache entry. Libraries like GPTCache implement this with configurable similarity thresholds.

### Q2: "How do you decide which model to route to?"
**Answer:** Train a lightweight classifier on (query, complexity_label) pairs. Or use a heuristic: query length, keyword detection, domain classification. Route simple queries (< 50 tokens, factual) to small models; complex queries (multi-step reasoning, code generation) to large models.

### Q3: "What's the ROI of prompt optimization?"
**Answer:** At scale, prompt optimization has massive ROI. A company spending $50K/month on LLM APIs can typically reduce costs to $10-15K through model routing + caching + prompt compression — a 70% reduction that pays for itself in days.

---

<p align="center"><a href="../../README.md">← Back to all questions</a></p>
