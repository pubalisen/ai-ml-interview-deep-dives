# Part 7: Choosing an Embedding Model

> **The Question:** "How do you choose an embedding model for your RAG system?"

---

## The Technical Breakdown

### The Decision Framework

Choosing an embedding model isn't about picking the "best" one — it's about matching the model to your constraints:

```
┌─────────────────────────────────────────────┐
│           EMBEDDING MODEL SELECTION          │
│                                              │
│  1. What language(s)?                        │
│     └─ English only → English-specific       │
│     └─ Multilingual → Cross-lingual model    │
│                                              │
│  2. Privacy constraints?                     │
│     └─ Can send data to API → OpenAI/Cohere  │
│     └─ Must stay on-prem → Open-source       │
│                                              │
│  3. Max chunk length?                        │
│     └─ <512 tokens → Most models work        │
│     └─ >512 tokens → Need long-context model │
│                                              │
│  4. Latency requirements?                    │
│     └─ Real-time (<50ms) → Small/local model │
│     └─ Batch OK (>200ms) → API model fine    │
│                                              │
│  5. Budget?                                  │
│     └─ Cost-sensitive → Open-source + GPU    │
│     └─ Low volume → API (pay per token)      │
└─────────────────────────────────────────────┘
```

### Evaluation with MTEB Benchmark

The Massive Text Embedding Benchmark (MTEB) ranks models across retrieval, classification, clustering, and semantic similarity:

| Rank | Model | Avg Score | Dims | Type |
|------|-------|-----------|------|------|
| Top tier | Cohere embed-v3 | 64.5 | 1024 | API |
| Top tier | OpenAI text-embedding-3-large | 64.6 | 3072 | API |
| Top tier | Voyage-3 | 67.3 | 1024 | API |
| Strong | bge-en-icl | 63.2 | 4096 | Open |
| Strong | GTE-Qwen2-7B-instruct | 62.8 | 3584 | Open |
| Good | E5-mistral-7b | 61.5 | 4096 | Open |
| Fast | all-MiniLM-L6-v2 | 56.3 | 384 | Open |

### Key Selection Criteria

#### Retrieval Quality
Don't trust overall MTEB scores alone — look at the **Retrieval** sub-task scores specifically. A model that excels at classification might underperform on retrieval.

#### Domain Fit
```
General text      → General-purpose models (OpenAI, Cohere)
Code/programming  → CodeBERT, Voyage-code-2
Medical/clinical  → PubMedBERT, BiomedBERT
Legal             → Legal-BERT fine-tuned variants
Financial         → FinBERT embeddings
```

#### Asymmetric vs Symmetric
```
Symmetric:   Both inputs are similar length (sentence ↔ sentence)
Asymmetric:  Short query ↔ long passage (most RAG use cases)

For RAG, you want asymmetric models — they're trained on
(short_query, long_relevant_passage) pairs.
```

#### Instruction-Prefixed Models
Some models accept task-specific instructions:
```python
# E5 models use query/passage prefixes
query_embedding = embed("query: How do I reset my password?")
doc_embedding = embed("passage: To reset your password, go to Settings > Security...")

# This helps the model understand asymmetric retrieval intent
```

### Cost Comparison

| Model | Cost per 1M tokens | 10M documents (500 tokens each) |
|-------|-------------------|---------------------------------|
| OpenAI small | $0.02 | $100 |
| OpenAI large | $0.13 | $650 |
| Cohere embed-v3 | $0.10 | $500 |
| Open-source (self-hosted) | GPU cost only | ~$50 (on A100 for ~2 hours) |

---

## Accuracy Check
| Claim | Verified? |
|-------|-----------|
| MTEB is the standard benchmark for embedding model comparison | ✅ Muennighoff et al. (2023), MTEB paper and leaderboard |
| Domain-specific embedding models outperform general ones on domain tasks | ✅ Gu et al. (2021), domain-adaptive pre-training benchmarks |
| Instruction-prefixed models improve asymmetric retrieval | ✅ Wang et al. (2024), E5-mistral paper |

## Scenario Examples
### A: A startup building a customer support bot for English-only users. Budget is tight, volume is 5K queries/day. Choice: OpenAI `text-embedding-3-small` — $0.02/1M tokens, 1536 dims, great quality, zero infrastructure. Total embedding cost: ~$1.50/month. The engineering time saved from not running GPU infrastructure far exceeds the API cost.
### B: A hospital building a medical literature search system. HIPAA requires all data stays on-premise — no external API calls. Choice: PubMedBERT fine-tuned for retrieval, self-hosted on an A10 GPU. The domain-specific model outperforms general-purpose OpenAI embeddings by 15% on medical queries because it understands terms like "myocardial infarction" = "heart attack" natively.

## Follow-Up Questions
### Q1: "Should you always pick the highest-ranked model on MTEB?"
**Answer:** No. MTEB scores are averages across diverse tasks. Your RAG use case might care only about retrieval, not clustering or classification. A model ranked #10 overall might be #3 on retrieval. Also, the top models are often massive (7B parameters) requiring expensive GPU inference — a smaller model with 95% of the quality at 10% of the latency is usually the better production choice.

### Q2: "Can you reduce embedding dimensions without re-training?"
**Answer:** Yes — OpenAI's `text-embedding-3` models support Matryoshka Representation Learning (MRL), where you truncate the vector to fewer dimensions (e.g., 3072 → 512) with minimal quality loss. Store 512-dim vectors for fast search, expand to full dimensions only for re-ranking. This reduces storage by 6x with only ~3% quality drop on retrieval benchmarks.

### Q3: "How do you evaluate an embedding model on YOUR data?"
**Answer:** Create a test set: 50-100 (query, relevant_document) pairs from your actual users. Embed everything, run retrieval, and measure recall@k (does the relevant doc appear in top-k results?). Compare 3-4 models on this test set. A model with 92% recall@5 on your data is better than one with 95% on MTEB but 85% on your data. Domain-specific evaluation always trumps benchmark scores.

---

<p align="center"><a href="../README.md">← Back to all questions</a></p>
