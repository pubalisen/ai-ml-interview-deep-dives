# Scenario 10: Contradictory Sources

> **The Scenario:** "Your RAG system retrieves contradictory answers from different sources. How do you handle it?"

---

## Solutions

1. **Source prioritization** — Assign authority rankings to sources. Official documentation > blog posts > community forums. When contradictions arise, prefer the higher-authority source.
2. **Recency weighting** — When two sources contradict, prefer the more recent one. A policy updated in 2024 supersedes one from 2022.
3. **Explicit contradiction detection** — Use an NLI model to detect when two retrieved chunks contradict each other. Flag the contradiction to the user rather than picking one silently.
4. **Prompt the LLM to acknowledge the conflict** — "If the provided sources contain conflicting information, present both perspectives and note the contradiction."
5. **Source metadata signals** — Use metadata (official vs unofficial, author authority, document status) to resolve conflicts automatically.
6. **Human-in-the-loop escalation** — When contradictions are detected on critical topics (medical, legal, financial), escalate to a human reviewer instead of generating a potentially wrong answer.

```python
# Contradiction detection pipeline
def detect_and_resolve_contradictions(chunks):
    for i, chunk_a in enumerate(chunks):
        for chunk_b in chunks[i+1:]:
            nli_result = nli_model.predict(
                premise=chunk_a.text,
                hypothesis=chunk_b.text
            )
            if nli_result == "CONTRADICTION":
                # Resolve based on source priority and recency
                if chunk_a.metadata["authority"] > chunk_b.metadata["authority"]:
                    preferred = chunk_a
                elif chunk_a.metadata["updated"] > chunk_b.metadata["updated"]:
                    preferred = chunk_a
                else:
                    preferred = chunk_b
                    
                return preferred, {
                    "conflict_detected": True,
                    "resolved_by": "source_priority"
                }
```

## Scenario Examples
### A: An employee asks "How many vacation days do I get?" The old handbook (2022) says 15 days, and the new policy memo (2024) says 20 days. Both are in the index. Without conflict resolution, the LLM might average them or pick randomly. Fix: add `last_updated` metadata. The retriever detects the contradiction (NLI: "15 days" CONTRADICTS "20 days"), resolves by recency, and presents: "You receive 20 vacation days per year (per the updated 2024 policy). Note: this supersedes the previous policy of 15 days."
### B: A medical information system retrieves two guidelines about aspirin dosage — one from the AHA (American Heart Association) and one from a hospital's internal protocol that hasn't been updated. They contradict on recommended dosage for cardiac patients. The system detects the contradiction, escalates to a pharmacist for review, and responds: "I found conflicting dosage information from two sources. Please consult your physician for the current recommendation." Patient safety > automatic resolution.

## Follow-Up Questions
### Q1: "How do you prevent contradictions from appearing in the first place?"
**Answer:** Address it at ingestion time: (1) de-duplicate content to prevent old and new versions of the same document from coexisting, (2) mark superseded documents as archived, (3) implement a content review pipeline where new documents are checked against existing chunks for contradictions before indexing. Prevention is cheaper than runtime detection, but you can't prevent all contradictions (different authoritative sources may genuinely disagree).

### Q2: "Should the LLM pick a side or present both views?"
**Answer:** Depends on the domain. For factual/policy questions (refund policy, pricing): pick the most authoritative/recent source and state it clearly. The user needs ONE answer. For opinion/analysis questions (investment advice, treatment options): present both perspectives with source attribution. The user decides. For safety-critical questions: don't pick a side — flag the conflict and escalate to a human expert.

### Q3: "How do you handle contradictions when sources are equally authoritative and equally recent?"
**Answer:** This is the hardest case. Options: (1) **Present both** with full source attribution and let the user decide. (2) **Ask a clarifying question** — "I found two equally valid perspectives. Are you asking about X-context or Y-context?" (3) **Confidence scoring** — compare the cosine similarity of each source to the query. The one more relevant to the specific question is preferred. (4) **Default to the most conservative/safe answer** in regulated domains.

---

<p align="center"><a href="../README.md">← Back to all questions</a></p>
