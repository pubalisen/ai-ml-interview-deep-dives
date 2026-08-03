# Part 14: Self-RAG

> **The Question:** "Explain Self-RAG. How does the model decide when to retrieve?"

---

## The Technical Breakdown

### What Is Self-RAG?

Self-RAG trains the LLM itself to decide **when to retrieve**, **whether the retrieved content is relevant**, and **whether its own answer is supported** by the retrieved evidence. Instead of always retrieving (standard RAG) or never retrieving (standard LLM), the model dynamically controls the process.

```
Standard RAG: Always retrieve → Always use retrieved context → Generate
Self-RAG:     Think → "Do I need retrieval?" →
                ├─ No  → Generate directly
                └─ Yes → Retrieve → "Is this relevant?" →
                           ├─ No  → Discard, try different query
                           └─ Yes → Generate → "Is my answer supported?" →
                                      ├─ No  → Regenerate with different context
                                      └─ Yes → Return answer
```

### The Reflection Tokens

Self-RAG introduces special tokens that the model generates as part of its output:

| Token | Purpose | Values |
|-------|---------|--------|
| `[Retrieve]` | Should I retrieve? | `yes` / `no` |
| `[IsRel]` | Is retrieved context relevant? | `relevant` / `irrelevant` |
| `[IsSup]` | Is my answer supported by context? | `fully supported` / `partially` / `no support` |
| `[IsUse]` | Is my answer useful to the user? | `5` (best) to `1` (worst) |

### How Self-RAG Works Step by Step

```
Input: "What year was the Eiffel Tower built?"

Step 1: Model generates [Retrieve = yes]
  → Decides it needs external info for this factual question

Step 2: Retrieve passages about the Eiffel Tower

Step 3: Model generates [IsRel = relevant]
  → Confirms the retrieved passage is on-topic

Step 4: Model generates answer: "The Eiffel Tower was built in 1887-1889"

Step 5: Model generates [IsSup = fully supported]
  → Verifies its answer matches the retrieved evidence

Step 6: Model generates [IsUse = 5]
  → Self-rates the answer as highly useful

Output: "The Eiffel Tower was built between 1887 and 1889."
```

### When Self-RAG Skips Retrieval

```
Input: "Write a haiku about autumn."

Step 1: Model generates [Retrieve = no]
  → Creative task, no factual retrieval needed

Step 2: Model generates directly:
  "Crimson leaves descend
   Cool winds carry autumn's song
   Nature's quiet rest"

No retrieval cost, no latency overhead.
```

### Self-RAG vs Standard RAG vs No RAG

| Aspect | No RAG | Standard RAG | Self-RAG |
|--------|--------|-------------|----------|
| Retrieval | Never | Always | When needed |
| Relevance check | N/A | None (trust retrieval) | Model evaluates |
| Answer verification | None | None | Model self-checks |
| Latency (factual) | Fast | Medium | Medium |
| Latency (creative) | Fast | Medium (wasted retrieval) | Fast (skips retrieval) |
| Accuracy (factual) | Low | High | Highest |
| Hallucination | High | Medium | Low |

### Training Self-RAG

The model is fine-tuned on data annotated with reflection tokens:
1. Use a critic model (GPT-4) to annotate training data with reflection tokens
2. Fine-tune the base model (Llama 2) on this annotated data
3. The model learns to generate reflection tokens naturally during inference

---

## Accuracy Check
| Claim | Verified? |
|-------|-----------|
| Self-RAG uses reflection tokens for self-evaluation | ✅ Asai et al. (2023), "Self-RAG: Learning to Retrieve, Generate, and Critique" |
| Self-RAG outperforms standard RAG on factoid QA benchmarks | ✅ Asai et al. (2023), improvements on PopQA, TriviaQA, ASQA |
| Self-RAG reduces unnecessary retrieval for non-factual queries | ✅ Asai et al. (2023), selective retrieval experiments |

## Scenario Examples
### A: A general-purpose assistant gets a mix of questions: "What's the population of Tokyo?" (factual — needs retrieval) and "Write me a professional email declining a meeting" (compositional — no retrieval needed). Standard RAG wastes ~300ms retrieving irrelevant docs for the email request. Self-RAG skips retrieval for the email, reducing average latency by 40% across the mixed workload while maintaining accuracy on factual questions.
### B: A medical diagnosis assistant uses Self-RAG. For "What are the symptoms of appendicitis?", it retrieves medical literature and verifies its answer is fully supported. For "Can appendicitis pain be on the left side?", it retrieves, finds conflicting information (atypical presentation), and generates [IsSup = partially supported], prompting a more nuanced answer: "Typically right-sided, but atypical left-sided presentation occurs in ~2% of cases, particularly with a longer appendix or situs inversus."

## Follow-Up Questions
### Q1: "How does Self-RAG differ from CRAG (Corrective RAG)?"
**Answer:** Related but different approaches. Self-RAG is trained end-to-end with reflection tokens — the model itself evaluates retrieval quality. CRAG uses a separate lightweight evaluator model to assess retrieved documents and triggers web search as fallback when confidence is low. Self-RAG is more integrated (single model does everything), while CRAG is modular (separate retriever, evaluator, and generator). CRAG is easier to implement in production without fine-tuning.

### Q2: "Can you implement Self-RAG behavior without fine-tuning?"
**Answer:** Approximately, yes. Use prompt engineering to simulate reflection tokens: "Before answering, decide if you need to search for information. If you do, I'll provide search results. Then evaluate whether the results are relevant. Finally, verify your answer is supported by the evidence." This gives you ~70% of Self-RAG's benefit without fine-tuning. The gap: a fine-tuned Self-RAG model produces reflection tokens more reliably and efficiently.

### Q3: "What are the limitations of Self-RAG?"
**Answer:** Three main limitations: (1) **Requires fine-tuning** — you can't just use an off-the-shelf model; the reflection token behavior must be trained. (2) **Single-model dependency** — if the model misjudges [IsRel] or [IsSup], there's no external check. (3) **Training data quality** — the reflection token annotations (typically from GPT-4) can be noisy, and the fine-tuned model inherits those biases. In practice, many teams find that a well-tuned standard RAG pipeline with re-ranking and good prompting closes most of the gap.

---

<p align="center"><a href="../README.md">← Back to all questions</a></p>
