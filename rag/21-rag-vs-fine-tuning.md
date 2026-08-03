# Part 21: RAG vs Fine-Tuning

> **The Question:** "Compare RAG vs fine-tuning. When would you use each?"

---

## The Technical Breakdown

### Core Difference

```
RAG:          "Here's the relevant info — now answer the question."
Fine-tuning:  "Learn this knowledge permanently so you always know it."

RAG puts knowledge IN THE PROMPT (external, at inference time).
Fine-tuning puts knowledge IN THE WEIGHTS (internal, at training time).
```

### Head-to-Head Comparison

| Dimension | RAG | Fine-Tuning |
|-----------|-----|-------------|
| **Knowledge type** | Factual, frequently changing | Style, behavior, domain vocabulary |
| **Update speed** | Instant (update docs) | Hours-days (re-train) |
| **Cost to update** | ~$0 (re-index docs) | $50-$5,000+ per training run |
| **Hallucination** | Lower (grounded in docs) | Higher (memorized, may confabulate) |
| **Citations** | Easy (point to source doc) | Hard (knowledge in weights) |
| **Latency** | Higher (retrieval step) | Lower (no retrieval) |
| **Data privacy** | Docs stay in your infra | Training data used by provider |
| **Setup complexity** | Medium (vector DB, pipeline) | Low-medium (training script) |
| **Scales to** | Millions of documents | Limited by training data size |

### When to Use RAG

```
✅ RAG is the right choice when:
  - Knowledge changes frequently (docs, policies, prices)
  - You need source attribution ("According to section 3.2...")
  - Large knowledge base (>1000 documents)
  - Data is private and can't be sent for training
  - You want fast iteration (update docs, not retrain)
  - Accuracy and faithfulness are critical
```

### When to Use Fine-Tuning

```
✅ Fine-tuning is the right choice when:
  - You need a specific output style or format
  - Domain-specific vocabulary and reasoning patterns
  - Consistent behavior across all queries (brand voice)
  - Latency-critical applications (can't afford retrieval step)
  - Small, stable knowledge set that rarely changes
  - Task-specific optimization (classification, extraction)
```

### When to Use Both (RAG + Fine-Tuning)

The most powerful production systems **combine both**:

```
Fine-tune the model to:
  - Follow your output format consistently
  - Understand domain terminology
  - Reason in domain-specific ways

Use RAG to:
  - Inject current, factual knowledge
  - Provide citations
  - Handle the long tail of specific questions
```

### The Decision Framework

```
Does the knowledge change frequently?
├── Yes → RAG
└── No
    ├── Do you need citations? → RAG
    └── No
        ├── Is it about style/behavior? → Fine-tuning
        └── Is it about factual knowledge?
            ├── Small knowledge set (<100 docs) → Fine-tuning OR RAG
            └── Large knowledge set (>100 docs) → RAG
```

---

## Accuracy Check
| Claim | Verified? |
|-------|-----------|
| RAG enables real-time knowledge updates without retraining | ✅ Fundamental RAG advantage, Lewis et al. (2020) |
| Fine-tuning is better for style/format/behavior changes | ✅ Industry consensus, OpenAI fine-tuning docs |
| Combining RAG + fine-tuning produces the best results | ✅ Demonstrated in production systems, Anthropic and OpenAI recommendations |

## Scenario Examples
### A: A tax preparation assistant needs to answer questions about the latest tax laws (change annually) AND format responses as structured tax advice. Solution: Fine-tune the model on 10,000 examples of well-formatted tax advice (teaches style and format). Use RAG to inject the current year's tax code and IRS guidelines (provides up-to-date factual knowledge). The fine-tuned model formats answers correctly, and RAG ensures the numbers and rules are current.
### B: A company wants its chatbot to always respond in their brand voice (friendly, concise, emoji-sprinkled). They fine-tune GPT-4o-mini on 5,000 examples of ideal responses. For factual questions about their products ("Does Plan X include feature Y?"), they use RAG against their product docs. Result: every answer sounds on-brand AND is factually correct. Without fine-tuning, RAG answers are accurate but generic. Without RAG, fine-tuned answers sound right but may contain outdated product info.

## Follow-Up Questions
### Q1: "Is fine-tuning becoming less necessary with better prompting?"
**Answer:** Partially. Modern LLMs (GPT-4o, Claude 3.5) follow complex instructions well enough that many style/format requirements can be achieved via system prompts alone. However, fine-tuning still wins for: (1) consistent behavior across thousands of queries (prompts drift), (2) reducing prompt length (fine-tuned behavior doesn't need instructions), (3) cost reduction (shorter prompts = fewer tokens = lower cost at scale), (4) domain-specific reasoning that prompting can't teach.

### Q2: "Can fine-tuning replace RAG by training on all your documents?"
**Answer:** No, for three reasons: (1) **Knowledge cutoff** — the model only knows what it learned during training. New docs require re-training. (2) **Hallucination** — fine-tuned models can't distinguish "I was trained on this" from "I'm making this up." RAG's retrieved context provides a grounding check. (3) **Scale** — fine-tuning on 1 million documents is impractical and may cause catastrophic forgetting of other capabilities. RAG scales to millions of docs trivially.

### Q3: "What about cost comparison at scale?"
**Answer:** At low volume (<1K queries/day): RAG is cheaper (no training cost). At high volume (>100K queries/day): fine-tuning may be cheaper because it eliminates retrieval costs and uses shorter prompts (no retrieved context). Break-even calculation: if RAG adds 2000 tokens of context per query at $0.01/1K tokens, that's $0.02 per query extra. At 100K queries/day = $2,000/day. A fine-tuning run costs $500-5000 once. If knowledge is stable, fine-tuning amortizes better at scale.

---

<p align="center"><a href="../README.md">← Back to all questions</a></p>
