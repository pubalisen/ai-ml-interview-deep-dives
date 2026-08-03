# Part 23: Citation and Source Attribution

> **The Question:** "How do you implement citation and source attribution in RAG?"

---

## The Technical Breakdown

### Why Citations Matter

Without citations, a RAG answer is just another LLM output — users can't verify it. Citations transform RAG from "the AI said" to "according to Document X, page 3."

```
Without citation: "The refund window is 30 days."
With citation:    "The refund window is 30 days. [Source: Return Policy v2.1, Section 3.2, Updated: Nov 2024]"
```

### Citation Implementation Strategies

#### 1. Chunk-Level Attribution
Attach source metadata to each retrieved chunk and instruct the LLM to cite:

```python
context = ""
for i, chunk in enumerate(retrieved_chunks):
    context += f"[Source {i+1}: {chunk.metadata['source']}, "
    context += f"Page {chunk.metadata['page']}]\n"
    context += f"{chunk.text}\n\n"

prompt = f"""Answer the question using ONLY the sources below.
After each claim, cite the source in brackets like [Source 1].

Sources:
{context}

Question: {query}"""
```

#### 2. Inline Citations
The LLM embeds citations within its response:

```
Output: "The enterprise plan costs $499/month [Source 1] and includes
unlimited API calls [Source 1]. For teams larger than 50, a custom
quote is required [Source 3]."
```

#### 3. Post-Generation Attribution
Generate the answer first, then map each claim to its source:

```python
# Step 1: Generate answer
answer = llm.generate(query, context)

# Step 2: For each sentence in the answer, find which chunk supports it
for sentence in answer.split('. '):
    supporting_chunk = find_most_similar_chunk(sentence, retrieved_chunks)
    if supporting_chunk.similarity > 0.8:
        citations.append({
            "claim": sentence,
            "source": supporting_chunk.metadata,
            "confidence": supporting_chunk.similarity
        })
    else:
        citations.append({
            "claim": sentence,
            "source": None,
            "warning": "UNSUPPORTED — may be hallucinated"
        })
```

#### 4. NLI-Based Verification
Use a Natural Language Inference model to verify each claim:

```
Premise (chunk):   "The refund window is 30 days for all products."
Hypothesis (claim): "Electronics can be returned within 30 days."

NLI result: ENTAILED → Citation is valid ✅
```

### Citation Quality Metrics

| Metric | What It Measures |
|--------|-----------------|
| **Citation precision** | % of citations that actually support the claim |
| **Citation recall** | % of claims that have a valid citation |
| **Hallucination rate** | % of claims with no supporting chunk |
| **Source coverage** | How many unique sources were cited |

### Common Citation Formats

```
Academic style:  "According to Smith et al. (2024)..."
Footnote style:  "The policy applies to all users.¹"
Inline bracket:  "Returns are accepted within 30 days [Policy Doc, p.3]"
Link style:      "Returns are accepted within 30 days (view source)"
```

---

## Accuracy Check
| Claim | Verified? |
|-------|-----------|
| Inline citations improve user trust in RAG answers | ✅ User studies across enterprise RAG deployments |
| NLI models can verify claim-source alignment | ✅ Honovich et al. (2022), TRUE benchmark for factual consistency |
| Post-generation attribution catches unsupported claims | ✅ Rashkin et al. (2023), attribution evaluation methods |

## Scenario Examples
### A: A medical information system where every claim MUST be traceable to a published source. Each sentence in the LLM's response is verified against the retrieved PubMed abstracts using NLI. Claims with NLI score < 0.7 are flagged: "⚠️ This claim could not be verified against the provided sources." Doctors trust the system because they can click any citation and read the original paper.
### B: A customer support bot cites the specific help article that answers the question: "You can upgrade your plan at any time through Settings > Billing. [Source: help.company.com/billing/upgrade, last updated 2 days ago]." If the user disagrees, they can click the link and verify. This reduces escalation to human agents by 35% because users trust the cited source.

## Follow-Up Questions
### Q1: "How do you handle when the LLM generates claims not supported by any retrieved chunk?"
**Answer:** Three approaches: (1) **Block** — don't show unsupported claims. Risk: incomplete answers. (2) **Flag** — show the claim with a warning: "⚠️ This information could not be verified from our knowledge base." (3) **Regenerate** — prompt the LLM again with stricter instructions: "Only state facts that appear verbatim in the provided sources." The right choice depends on your domain's risk tolerance.

### Q2: "How accurate are LLM-generated citations?"
**Answer:** LLMs frequently hallucinate citations — they'll cite "Source 3" when the information actually came from Source 1, or cite a source for a hallucinated fact. Accuracy varies: ~70-80% with good prompting, ~90%+ with post-generation NLI verification. Never trust LLM-generated citations without verification. The post-generation attribution approach (verify each claim against each chunk) is more reliable than asking the LLM to self-cite.

### Q3: "How do you make citations clickable in a UI?"
**Answer:** Store chunk metadata with unique IDs linking to the original document location. In the UI: (1) render citations as clickable links, (2) on click, open the source document and highlight the relevant section, (3) show a preview tooltip on hover with the source text. Implementation: return a structured response with `{answer, citations: [{text, source_url, page, highlight_text}]}` and render it in your frontend with anchor links.

---

<p align="center"><a href="../README.md">← Back to all questions</a></p>
