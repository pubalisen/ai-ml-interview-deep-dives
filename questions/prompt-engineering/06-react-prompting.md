# Part 6: ReAct (Reasoning + Acting) Prompting

> **The Question:** "What is ReAct (Reasoning + Acting) prompting, and how does it work?"

---

## The Technical Breakdown

### What Is ReAct?

ReAct interleaves **reasoning** (thinking about what to do) with **acting** (executing actions like tool calls) in a loop:

```
Thought: I need to find the population of Tokyo to answer this question.
Action: search("population of Tokyo 2024")
Observation: Tokyo has a population of approximately 13.96 million.
Thought: Now I have the population. The question asks about the metro area.
Action: search("Tokyo metropolitan area population")
Observation: The Greater Tokyo Area has approximately 37.4 million people.
Thought: I now have enough information to answer.
Answer: The Tokyo metropolitan area has approximately 37.4 million people.
```

### The ReAct Loop

```
┌──────────────┐
│   Thought    │ ← Model reasons about current state
└──────┬───────┘
       ↓
┌──────────────┐
│   Action     │ ← Model decides which tool to call
└──────┬───────┘
       ↓
┌──────────────┐
│ Observation  │ ← Tool returns result
└──────┬───────┘
       ↓
    Repeat until answer found
```

### ReAct vs. CoT vs. Act-Only

| Approach | Reasoning | Action | Quality |
|----------|----------|--------|---------|
| **Act-only** | ❌ | ✅ | Calls tools without planning — misses steps |
| **CoT-only** | ✅ | ❌ | Reasons well but can't access external info |
| **ReAct** | ✅ | ✅ | Reasons AND accesses external tools |

---

## Accuracy Check
| Claim | Verified? |
|-------|-----------|
| ReAct interleaves reasoning and action | ✅ Yao et al. (2023), "ReAct: Synergizing Reasoning and Acting" |
| ReAct outperforms CoT-only on knowledge-intensive tasks | ✅ HotpotQA, FEVER benchmarks |
| ReAct outperforms Act-only on multi-step tasks | ✅ Fewer hallucinations due to grounded observations |

## Scenario Examples
### A: A research assistant asked "Is drug X approved for condition Y?" uses ReAct: Thought (need to check FDA database) → Action (query FDA API) → Observation (drug X approved for condition Z, not Y) → Thought (need to check off-label studies) → Action (search PubMed) → Answer (not FDA-approved but 3 clinical trials show efficacy).
### B: A customer support agent asked about a refund: Thought (need order details) → Action (query order DB) → Observation (order #123, shipped) → Thought (check refund policy for shipped orders) → Action (lookup policy) → Answer (eligible for return within 30 days).

## Follow-Up Questions
### Q1: "How does ReAct relate to modern agent frameworks?"
**Answer:** ReAct is the **foundational pattern** behind frameworks like LangChain agents, AutoGPT, and tool-using LLMs. When an LLM calls a function and processes the result before deciding the next step, it's implementing ReAct. Modern function-calling APIs (OpenAI, Anthropic) implement this pattern natively.

### Q2: "What happens when the ReAct loop doesn't converge?"
**Answer:** Set a maximum number of iterations (typically 5-10). If the model hasn't found an answer, it should either provide a best-effort response with caveats or escalate. Common failure modes: infinite tool-calling loops, incorrect tool selection, and misinterpreting observations.

### Q3: "Can you combine ReAct with self-consistency?"
**Answer:** Yes — generate N independent ReAct traces and vote on final answers. This is expensive (N × multiple tool calls) but robust for high-stakes decisions. Each trace may use different tools or reasoning paths, converging on the correct answer.

---

<p align="center"><a href="../../README.md">← Back to all questions</a></p>
