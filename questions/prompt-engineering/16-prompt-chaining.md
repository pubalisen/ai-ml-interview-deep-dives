# Part 16: Prompt Chaining

> **The Question:** "What is prompt chaining, and how do you design a chain of prompts for complex tasks?"

---

## The Technical Breakdown

### What Is Prompt Chaining?

Breaking a complex task into **sequential sub-tasks**, where each prompt's output feeds into the next prompt as input:

```
Query: "Analyze this company's financial health and generate an investment recommendation"

Prompt 1: "Extract key financial metrics from this report"
    → {"revenue": "$5B", "profit_margin": "12%", "debt_ratio": "0.4"}

Prompt 2: "Analyze these metrics against industry benchmarks: {metrics}"  
    → "Revenue growth is above average, but profit margin trails competitors..."

Prompt 3: "Based on this analysis, generate an investment recommendation: {analysis}"
    → "HOLD — Strong revenue offset by margin pressure. Rating: 6/10."
```

### Why Chain vs. Single Prompt?

| Single Prompt | Prompt Chain |
|--------------|-------------|
| One complex instruction | Multiple focused instructions |
| Error in one part ruins everything | Errors isolated to one step |
| Hard to debug | Debug step-by-step |
| Uses one model size | Can use different models per step |
| Limited to one context window | Each step gets full context |

### Chain Design Patterns

| Pattern | Description | Use Case |
|---------|-------------|----------|
| **Sequential** | A → B → C | Linear workflows |
| **Parallel** | A → (B, C, D) → E | Independent sub-tasks |
| **Conditional** | A → if X then B else C | Branching logic |
| **Iterative** | A → B → check → repeat B | Refinement loops |
| **Map-Reduce** | [A₁, A₂, ...Aₙ] → reduce | Processing multiple items |

---

## Scenario Examples
### A: A code review pipeline: (1) Extract code changes, (2) Check for security vulnerabilities, (3) Evaluate code quality, (4) Synthesize a review. Each step uses a specialized prompt. The security step uses a safety-focused model; the quality step uses a code-specialized model.
### B: A content pipeline: (1) Research topic (RAG), (2) Generate outline, (3) Human approves outline, (4) Write each section (parallel), (5) Edit for consistency, (6) Generate metadata/SEO.

## Follow-Up Questions
### Q1: "How do you handle errors mid-chain?"
**Answer:** Implement retry with error context at each step. If step 3 fails, retry with the error message appended: "Previous attempt failed because: {error}. Try again." Set max retries (typically 3). If still failing, fall back to a simpler approach or escalate.

### Q2: "How do you optimize chain latency?"
**Answer:** (1) Parallelize independent steps, (2) use smaller models for simple steps, (3) cache results for repeated inputs, (4) batch multiple items through the same step. A well-designed chain is often faster than a single complex prompt because each step is simpler and more cacheable.

### Q3: "What frameworks support prompt chaining?"
**Answer:** LangChain (LCEL), LlamaIndex (QueryPipeline), Semantic Kernel, DSPy. These provide abstractions for chaining, branching, error handling, and observability. For simple chains, plain Python with sequential API calls is often cleaner than a framework.

---

<p align="center"><a href="../../README.md">← Back to all questions</a></p>
