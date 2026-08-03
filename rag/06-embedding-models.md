# Part 6: Embedding Models

> **The Question:** "What are embedding models, and how do they convert text to vectors?"

---

## The Technical Breakdown

### What Embedding Models Do

An embedding model converts text into a fixed-length dense vector (array of floating-point numbers) that captures semantic meaning:

```
"How do I reset my password?" → [0.023, -0.841, 0.127, ..., 0.445]
                                  ↑ 768 to 3072 dimensions
```

The key property: **semantically similar text produces similar vectors**. "How do I reset my password?" and "I forgot my login credentials" produce vectors that are close together in the embedding space, even though they share few words.

### How Embedding Models Work Internally

Most modern embedding models are **Transformer encoders** (like BERT) trained with a contrastive learning objective:

```
Training:
  Positive pair: ("reset password", "forgot login") → Push embeddings CLOSER
  Negative pair: ("reset password", "quarterly revenue") → Push embeddings APART

Architecture:
  Input tokens → Transformer Encoder → Token embeddings → Pooling → Single vector

Pooling strategies:
  [CLS] pooling:  Use the [CLS] token's embedding
  Mean pooling:   Average all token embeddings (most common)
  Max pooling:    Take max across each dimension
```

### The Embedding Pipeline in RAG

```
                    INDEXING
Document chunk ──→ Tokenizer ──→ Transformer ──→ Pooling ──→ Vector [768-dim]
                                                               │
                                                               ▼
                                                          Vector DB

                    QUERY TIME
User query ──→ Same Tokenizer ──→ Same Transformer ──→ Same Pooling ──→ Query Vector
                                                                            │
                                                                            ▼
                                                               Cosine Similarity Search
                                                                against stored vectors
```

### Key Embedding Models (2024-2025)

| Model | Dimensions | Max Tokens | Type | Performance |
|-------|-----------|------------|------|-------------|
| `text-embedding-3-small` (OpenAI) | 1536 | 8191 | API | Good quality, cheap |
| `text-embedding-3-large` (OpenAI) | 3072 | 8191 | API | Best quality (OpenAI) |
| `embed-v3` (Cohere) | 1024 | 512 | API | Excellent multilingual |
| `bge-large-en-v1.5` | 1024 | 512 | Open-source | Top open-source |
| `E5-mistral-7b-instruct` | 4096 | 32768 | Open-source | Best open-source, long context |
| `all-MiniLM-L6-v2` | 384 | 256 | Open-source | Fast, lightweight |
| `GTE-Qwen2` | 1024-1792 | 8192 | Open-source | Strong multilingual |

### Similarity Metrics

```python
# Cosine Similarity (most common for normalized embeddings)
cos_sim = dot(A, B) / (norm(A) * norm(B))
# Range: -1 to 1. Higher = more similar.

# Dot Product (if embeddings are already normalized)
dot_sim = dot(A, B)
# Faster — skip the normalization step.

# Euclidean Distance (L2)
l2_dist = sqrt(sum((A - B)^2))
# Lower = more similar. Less common in practice.
```

### Dimensionality Trade-offs

```
384 dims (MiniLM):  Fast search, low storage, ~5% quality drop
768 dims (BERT):    Standard quality, moderate storage
1536 dims (OpenAI): High quality, more storage
3072 dims (large):  Highest quality, most storage

Storage: 1M vectors × 1536 dims × 4 bytes = ~6 GB
Storage: 1M vectors × 384 dims × 4 bytes  = ~1.5 GB
```

---

## Accuracy Check
| Claim | Verified? |
|-------|-----------|
| Embedding models encode semantic meaning into dense vectors | ✅ Reimers & Gurevych (2019), Sentence-BERT |
| Contrastive learning trains embeddings to push similar text closer | ✅ Gao et al. (2021), SimCSE |
| Mean pooling generally outperforms [CLS] pooling | ✅ Sentence-Transformers benchmarks |
| Cosine similarity is the standard metric for embedding comparison | ✅ MTEB benchmark evaluation protocol |

## Scenario Examples
### A: An e-commerce product search system uses `text-embedding-3-small` (1536 dims) to embed 2 million product descriptions. Storage in Pinecone: ~12 GB. A customer searches "comfortable shoes for standing all day" — the embedding captures the intent (comfort + durability + standing) and retrieves products like "ergonomic nursing shoes" and "memory foam work boots" that share semantic meaning but few keywords.
### B: A multilingual customer support system serves customers in English, Spanish, and Japanese. They choose Cohere's `embed-v3` because it produces cross-lingual embeddings — a Spanish question "¿Cómo reseteo mi contraseña?" retrieves English support docs about password resets because the embeddings occupy the same region of the vector space regardless of language.

## Follow-Up Questions
### Q1: "What happens when you use different embedding models for indexing and querying?"
**Answer:** Complete retrieval failure. Different models produce different vector spaces — a vector from Model A has no meaningful similarity relationship with a vector from Model B. It's like comparing GPS coordinates from two different map projections. If you switch embedding models, you must re-embed your entire corpus. This is why the embedding model choice is a critical upfront decision.

### Q2: "How do you handle text longer than the embedding model's max token limit?"
**Answer:** Three approaches: (1) **Truncate** — cut at max_tokens (loses information at the end). (2) **Chunk first** — split the text into model-sized chunks and embed each separately. (3) **Use a long-context model** — E5-mistral supports 32K tokens. (4) **Pool across windows** — embed overlapping windows of the text and average the resulting vectors. Option 2 (chunk first) is the standard RAG approach.

### Q3: "Can you fine-tune embedding models for your domain?"
**Answer:** Yes, and it's often worth it. Generic embedding models may not understand domain-specific terminology. Fine-tuning with domain pairs (query, relevant_doc) using contrastive loss improves retrieval by 10-30% on domain-specific benchmarks. Tools: Sentence-Transformers `SentenceTransformerTrainer`, OpenAI fine-tuning API. You need ~1,000-10,000 training pairs. Common approach: use LLM to generate synthetic (query, passage) pairs from your documents.

---

<p align="center"><a href="../README.md">← Back to all questions</a></p>
