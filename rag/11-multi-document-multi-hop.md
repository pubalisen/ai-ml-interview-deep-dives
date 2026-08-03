# Part 11: Multi-Document and Multi-Hop Questions

> **The Question:** "How do you handle multi-document and multi-hop questions in RAG?"

---

## The Technical Breakdown

### What Are Multi-Hop Questions?

Multi-hop questions require combining information from **multiple sources** to produce an answer. A single document retrieval isn't enough.

```
Single-hop: "What is the refund policy?"
  → One chunk answers this completely.

Multi-hop: "Can I get a refund on an item I bought with a gift card during a promotional sale?"
  → Requires: refund policy + gift card policy + promotional sale terms
  → Three different documents, combined reasoning.
```

### Why Naive RAG Fails on Multi-Hop

```
Query: "Which company has higher revenue — the one that acquired Instagram or the one that acquired YouTube?"

Naive RAG retrieves:
  Chunk 1: "Meta (Facebook) acquired Instagram in 2012 for $1B"
  Chunk 2: "Google acquired YouTube in 2006 for $1.65B"
  Chunk 3: "Alphabet (Google's parent) reported $307B revenue in 2023"

Missing: Meta's revenue! The retrieval found acquisition info but only one company's revenue.
```

### Strategies for Multi-Hop RAG

#### 1. Query Decomposition
Break the complex question into sub-questions:

```python
# LLM decomposes the query:
original = "Which acquired company's parent has higher revenue — Instagram or YouTube?"

sub_questions = [
    "Who acquired Instagram?",        # → Meta/Facebook
    "What is Meta's annual revenue?",  # → $135B
    "Who acquired YouTube?",           # → Google/Alphabet
    "What is Alphabet's annual revenue?", # → $307B
]

# Retrieve for each sub-question, then synthesize
```

#### 2. Iterative Retrieval
Retrieve → Read → Identify gaps → Retrieve more:

```
Step 1: Retrieve for original query
  → Found: Instagram acquired by Meta, YouTube acquired by Google
  → Missing: Revenue figures

Step 2: Retrieve for "Meta annual revenue 2023"
  → Found: $135B

Step 3: Retrieve for "Alphabet annual revenue 2023"
  → Found: $307B

Step 4: Synthesize: "Alphabet (YouTube's parent) has higher revenue at $307B vs Meta's $135B"
```

#### 3. Chain-of-Thought Retrieval
Let the LLM reason step-by-step, retrieving as needed:

```
Thought: I need to find who acquired Instagram and YouTube.
Retrieve: "Instagram acquisition" → Meta acquired Instagram
Retrieve: "YouTube acquisition" → Google acquired YouTube
Thought: Now I need both companies' revenue.
Retrieve: "Meta revenue 2023" → $135B
Retrieve: "Alphabet revenue 2023" → $307B
Answer: Alphabet ($307B) has higher revenue than Meta ($135B).
```

#### 4. Graph-Based Retrieval
Build a knowledge graph connecting entities across documents:

```
[Meta] --acquired--> [Instagram]
[Meta] --revenue--> [$135B]
[Alphabet] --acquired--> [YouTube]
[Alphabet] --revenue--> [$307B]

Query traverses the graph: Instagram → acquired_by → Meta → revenue → $135B
```

### Multi-Document Synthesis

When the answer spans multiple documents, the LLM must synthesize — not just concatenate:

```
Document A: "Drug X reduces blood pressure by 15% in clinical trials"
Document B: "Drug X has side effects including dizziness and nausea"
Document C: "Drug X is contraindicated with blood thinners"

Synthesized answer: "Drug X effectively reduces blood pressure by 15% but
has side effects (dizziness, nausea) and should not be taken with blood
thinners. Consult your doctor before use."
```

---

## Accuracy Check
| Claim | Verified? |
|-------|-----------|
| Multi-hop questions require information from multiple documents | ✅ Yang et al. (2018), HotpotQA dataset |
| Query decomposition improves multi-hop QA accuracy | ✅ Press et al. (2023), "Measuring and Narrowing the Compositionality Gap" |
| Iterative retrieval outperforms single-shot for complex questions | ✅ Jiang et al. (2023), FLARE; Trivedi et al. (2023), IRCoT |

## Scenario Examples
### A: A compliance officer asks "Do our data retention policies comply with both GDPR and CCPA for customers in California who opted out of data sharing?" This requires: (1) internal data retention policy, (2) GDPR requirements, (3) CCPA requirements, (4) California-specific opt-out rules. Query decomposition breaks this into four sub-queries. Each retrieves from different policy documents. The LLM cross-references all four to produce a gap analysis.
### B: A venture capital analyst asks "Compare the unit economics of the top 3 AI startups that raised Series B in 2024." Iterative retrieval: (1) find Series B AI startups → retrieves funding news, (2) for each company, retrieve revenue/cost data, (3) calculate and compare unit economics. No single document has all this information — the system must chain three rounds of retrieval.

## Follow-Up Questions
### Q1: "How do you know when a question is multi-hop vs single-hop?"
**Answer:** Use an LLM classifier: before retrieval, ask the model "Does this question require information from multiple topics or sources? If so, list the sub-topics." Multi-hop indicators: comparative questions ("compare X and Y"), conditional questions ("if A then what about B?"), temporal questions ("how has X changed from 2022 to 2024?"), and questions with multiple entities. Route single-hop to standard RAG, multi-hop to the decomposition pipeline.

### Q2: "What's the cost of multi-hop RAG vs single-hop?"
**Answer:** Significantly higher. A 3-hop question requires 3x the retrieval calls, 3x the embedding API calls, and the synthesis prompt is longer (more retrieved context). Typical cost: single-hop = 1 retrieval + 1 generation (~$0.005). 3-hop = 3-5 retrievals + 1-3 intermediate generations + 1 synthesis (~$0.03-0.05). Budget 5-10x the cost of single-hop for multi-hop queries. Route accordingly.

### Q3: "How do you evaluate multi-hop RAG accuracy?"
**Answer:** Standard retrieval metrics aren't enough. You need: (1) **Sub-question coverage** — did the system identify all required sub-questions? (2) **Per-hop retrieval accuracy** — was the correct chunk retrieved for each sub-question? (3) **Synthesis accuracy** — did the LLM correctly combine the retrieved facts? Datasets like HotpotQA and MuSiQue provide multi-hop evaluation benchmarks with annotated reasoning chains.

---

<p align="center"><a href="../README.md">← Back to all questions</a></p>
