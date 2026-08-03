# Part 8: Agentic RAG

> **The Question:** "Explain Agentic RAG."

---

## The Technical Breakdown

### What Is Agentic RAG?

Agentic RAG gives the LLM **control over the retrieval process** instead of using a fixed retrieve-then-generate pipeline. The model decides:
- **When** to retrieve (not every query needs retrieval)
- **What** to retrieve (query reformulation)
- **Where** to retrieve from (which knowledge source)
- **Whether to retrieve again** (multi-step retrieval)

```
Traditional RAG:
  Query → Always Retrieve → Generate

Agentic RAG:
  Query → Agent Decides:
    ├─ "I know this already" → Generate directly
    ├─ "I need docs" → Retrieve → Read → Generate
    ├─ "These docs aren't enough" → Rewrite query → Retrieve again → Generate
    └─ "I need different sources" → Query DB + API + Docs → Combine → Generate
```

### The Architecture

```
┌──────────────────────────────────────────┐
│              AGENTIC RAG LOOP            │
│                                          │
│  User Query                              │
│      │                                   │
│      ▼                                   │
│  ┌──────────┐                            │
│  │  PLANNER  │ ← LLM decides strategy    │
│  └────┬─────┘                            │
│       │                                  │
│       ├── Tool: vector_search(query)     │
│       ├── Tool: sql_query(table, filter) │
│       ├── Tool: web_search(query)        │
│       ├── Tool: api_call(endpoint)       │
│       └── Tool: rewrite_query(feedback)  │
│                                          │
│       ▼                                  │
│  ┌──────────┐                            │
│  │ EVALUATOR │ ← "Is this enough?"       │
│  └────┬─────┘                            │
│       │                                  │
│       ├── Yes → Generate final answer    │
│       └── No  → Loop back to PLANNER     │
└──────────────────────────────────────────┘
```

### Key Agentic RAG Patterns

#### 1. Routing RAG
The agent routes queries to different knowledge sources:
```python
def route_query(query):
    if is_about_pricing(query):
        return search_pricing_db(query)
    elif is_about_technical_docs(query):
        return search_vector_db(query)
    elif is_about_recent_events(query):
        return web_search(query)
    else:
        return search_general_kb(query)
```

#### 2. Adaptive Retrieval
The agent decides whether retrieval is even needed:
```
Query: "What is 2 + 2?"         → No retrieval needed (general knowledge)
Query: "What's our Q3 revenue?" → Retrieve from financial docs
Query: "Summarize this PDF"     → Document already in context, no retrieval
```

#### 3. Iterative Retrieval
The agent retrieves, reads, and retrieves again if needed:
```
Step 1: Search "diabetes treatment guidelines"
Step 2: Read retrieved docs → "I see metformin mentioned but need dosage info"
Step 3: Search "metformin dosage recommendations type 2 diabetes"
Step 4: Read → "Now I have enough to answer completely"
Step 5: Generate comprehensive answer
```

#### 4. Multi-Source Orchestration
The agent queries multiple sources and synthesizes:
```
Query: "Should we use Redis or Memcached for our caching layer?"

Agent:
  1. Search internal architecture docs (what do we already use?)
  2. Search technical blog posts (benchmark comparisons)
  3. Query Stack Overflow API (community recommendations)
  4. Synthesize all sources into a recommendation
```

### Agentic RAG vs Traditional RAG

| Aspect | Traditional RAG | Agentic RAG |
|--------|----------------|-------------|
| Retrieval trigger | Always | Agent decides |
| Query used | Original query | Agent may rewrite |
| Sources | Single vector DB | Multiple sources |
| Iterations | One-shot | Multi-step loop |
| Failure recovery | None | Agent retries with new strategy |
| Complexity | Low | High |
| Latency | Fast (one retrieval) | Slower (multiple steps) |
| Cost | Predictable | Variable (depends on iterations) |

---

## Accuracy Check
| Claim | Verified? |
|-------|-----------|
| Agentic RAG gives the model control over when and how to retrieve | ✅ LlamaIndex "Agentic RAG" pattern, industry terminology |
| Iterative retrieval improves answer quality for complex questions | ✅ Jiang et al. (2023), "Active Retrieval Augmented Generation" |
| Multi-source retrieval enables richer answers than single-source | ✅ Standard in production systems (Perplexity, ChatGPT with browsing) |

## Scenario Examples
### A: A research assistant gets the question "What are the latest FDA-approved treatments for Alzheimer's and how do they compare in efficacy?" Traditional RAG does one search and might miss either the FDA data or the efficacy comparisons. Agentic RAG: (1) searches FDA database for approved treatments, (2) searches PubMed for clinical trial results, (3) searches internal medical knowledge base for drug comparisons, (4) synthesizes all three sources into a comprehensive comparison table.
### B: A coding assistant gets "Why is our API returning 500 errors?" Agentic RAG: (1) searches error logs (Datadog API), (2) finds the relevant service → searches its source code docs, (3) finds a recent deployment → searches the deployment changelog, (4) identifies the root cause: a config change in the last deploy broke the database connection string.

## Follow-Up Questions
### Q1: "How do you prevent Agentic RAG from looping forever?"
**Answer:** Three safeguards: (1) **Max iterations** — hard cap at 3-5 retrieval rounds. (2) **Token budget** — stop when total tokens (retrieved docs + generations) exceed a threshold. (3) **Diminishing returns detection** — if the last retrieval didn't add new relevant information (measured by semantic similarity to previous results), stop and generate with what you have.

### Q2: "How does Agentic RAG relate to AI agents with tool use?"
**Answer:** Agentic RAG is essentially a specialized agent where the primary tools are retrieval operations (vector search, SQL query, API calls). The same agent architecture (ReAct loop, tool calling, planning) applies. The difference is scope: a general AI agent might also write code, send emails, or modify databases. An Agentic RAG agent is focused specifically on information retrieval and synthesis.

### Q3: "When should you use traditional RAG over Agentic RAG?"
**Answer:** Traditional RAG is better when: (1) queries are simple and uniform ("What's the return policy?"), (2) latency is critical (agentic adds 2-5x latency per iteration), (3) cost must be predictable (agentic has variable token consumption), (4) your knowledge base is a single source with good coverage. Agentic RAG shines for complex, multi-faceted questions that require reasoning about what information is missing.

---

<p align="center"><a href="../README.md">← Back to all questions</a></p>
