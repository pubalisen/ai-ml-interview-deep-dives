# Part 11: Embeddings

> **The Question:** "What are embeddings?"

---

## What They're Actually Testing

The interviewer wants you to explain how **discrete symbols** (words, tokens, images, products) are represented as **continuous vectors** in a way that captures semantic meaning. They're checking if you understand the geometry of embedding spaces — why "king - man + woman = queen" works — and how embeddings are used in production systems (not just inside LLMs, but in search, RAG, and recommendations).

---

## The Technical Breakdown

### What Are Embeddings?

An embedding is a **dense, low-dimensional vector representation** of a discrete object (word, token, sentence, image, product, user) in a continuous vector space, where **geometric relationships encode semantic relationships**.

```
"cat"    → [0.22, -0.41, 0.89, 0.03, ...]   # 768-dimensional vector
"dog"    → [0.25, -0.38, 0.85, 0.07, ...]   # similar vector (close in space)
"rocket" → [-0.72, 0.56, -0.11, 0.94, ...]  # dissimilar vector (far in space)
```

### Why Not One-Hot Encoding?

| Approach | "cat" | "dog" | "rocket" | Dimension |
|----------|-------|-------|----------|-----------|
| **One-hot** | [1,0,0,...,0] | [0,1,0,...,0] | [0,0,1,...,0] | ~50,000 (vocab size) |
| **Embedding** | [0.22, -0.41, ...] | [0.25, -0.38, ...] | [-0.72, 0.56, ...] | 768 (fixed) |

One-hot problems:
1. **No semantic information** — cosine similarity between any two one-hot vectors is exactly 0
2. **Dimensionality** — vectors are as large as the vocabulary (50K+)
3. **Sparsity** — 99.99% of entries are zeros

Embeddings solve all three: semantically similar items are close together, dimensions are manageable (256-4096), and vectors are dense.

### How Embeddings Are Learned

#### Inside LLMs (Token Embeddings)

The embedding layer is a **lookup table** — a matrix of shape `[vocab_size × d_model]`:

```python
# Simplified
embedding_matrix = nn.Embedding(50000, 768)  # 50K tokens, 768 dimensions
token_ids = [2, 1841, 2047]  # "The cat sat"
vectors = embedding_matrix[token_ids]  # Look up 3 vectors, each 768-dim
```

These vectors are learned during pre-training via backpropagation. The training objective (next-token prediction) forces the model to position tokens with similar contexts close together in the embedding space.

#### Sentence/Document Embeddings (for RAG & Search)

Embedding models like `text-embedding-3-small`, `BGE`, or `E5` produce a single vector for an entire text:

```python
text = "How to fix a memory leak in Python"
vector = embedding_model.encode(text)  # → [0.12, -0.45, 0.78, ...] (1536-dim)
```

These models are trained with **contrastive learning**: similar texts (question-answer pairs, paraphrases) are pushed close together; dissimilar texts are pushed apart.

### The Geometry of Embedding Spaces

The famous "king − man + woman ≈ queen" example demonstrates that embeddings capture **relational structure**:

```
embed("king") - embed("man") + embed("woman") ≈ embed("queen")
```

This works because the model has learned that the **difference vector** between "king" and "man" represents the concept of "royalty" — and adding that difference to "woman" lands near "queen."

### Measuring Similarity

| Method | Formula | Range | Use Case |
|--------|---------|-------|----------|
| **Cosine similarity** | (A·B) / (‖A‖·‖B‖) | [-1, 1] | Standard for text search. Ignores magnitude, measures direction. |
| **Dot product** | A·B | (-∞, +∞) | When magnitude matters (e.g., confidence-weighted similarity) |
| **Euclidean distance** | ‖A-B‖₂ | [0, +∞) | When absolute position in space matters |

**Cosine similarity** is the industry standard for semantic search because it's **scale-invariant** — documents of different lengths produce vectors of different magnitudes, but their direction (meaning) is what matters.

---

## Accuracy Check

| Claim | Verified? |
|-------|-----------|
| Embeddings are dense, low-dimensional vectors encoding semantic meaning | ✅ Mikolov et al., Word2Vec (2013); Pennington et al., GloVe (2014) |
| King - man + woman ≈ queen demonstrates relational structure | ✅ Mikolov et al. (2013), Section 4 |
| LLM token embeddings are learned lookup tables | ✅ Core Transformer architecture |
| Contrastive learning trains sentence embedding models | ✅ SimCSE (Gao et al., 2021), E5 (Wang et al., 2022) |
| Cosine similarity is scale-invariant | ✅ Mathematical property |

---

## Scenario Examples

### Scenario 1: Building a Semantic Search System

A company has 500,000 support articles and wants users to search in natural language ("my laptop won't charge") instead of keyword matching. They use an embedding model to convert every article into a 1536-dimensional vector and store them in a vector database (Pinecone/Weaviate). At query time, the user's question is embedded into the same space, and the system finds the nearest article vectors using cosine similarity. "My laptop won't charge" matches articles about "battery issues," "power adapter troubleshooting," and "charging port repair" — even though those articles never contain the exact phrase "won't charge." This is the foundation of **RAG pipelines**.

### Scenario 2: Detecting Semantic Duplicates in a Knowledge Base

A company's internal wiki has 10,000 articles written by different teams. Many articles cover the same topic with different wording ("How to reset your password" vs. "Steps for password recovery" vs. "Can't log in — password help"). By embedding all articles and computing pairwise cosine similarities, the team identifies clusters of near-duplicate content (similarity > 0.92). They merge 1,200 duplicate articles, reducing knowledge base noise and improving RAG retrieval accuracy by 15% (fewer redundant chunks competing for the top-k slots).

---

## Follow-Up Questions

### Q1: "What's the difference between token embeddings inside an LLM and sentence embeddings from a model like text-embedding-3?"

**Answer:** 
- **Token embeddings** are the raw lookup vectors from the LLM's embedding layer — one vector per token. They represent a token's meaning **in isolation** before any attention processing. They're not useful for comparing sentences because they don't incorporate context.
- **Sentence embeddings** are produced by specialized models that take the full text, process it through Transformer layers, and output a single vector (usually by pooling the final layer's token representations). This vector captures the **meaning of the entire text** in context. Sentence embeddings are what you use for semantic search, RAG, and similarity comparison.

An LLM's token embedding for "bank" is the same regardless of whether the sentence is about rivers or finance. A sentence embedding model produces different vectors for "I went to the river bank" vs. "I deposited money at the bank."

### Q2: "How do you handle the case where two texts are semantically similar but your embedding model produces distant vectors?"

**Answer:** This is an **embedding model quality issue**. Solutions:
1. **Try a better embedding model** — newer models like `text-embedding-3-large`, `BGE-M3`, or `E5-mistral` generally produce better representations.
2. **Fine-tune the embedding model** on your domain data using contrastive learning with (query, relevant_document) pairs from your specific use case.
3. **Use hybrid search** — combine vector similarity with keyword (BM25) search. If the embedding misses a match, keyword overlap may catch it.
4. **Use a re-ranker** — retrieve top-100 with embeddings (high recall), then re-rank with a cross-encoder model (high precision). Cross-encoders compare two texts jointly and produce better similarity scores than embedding comparison.

### Q3: "Why are embedding dimensions typically 768, 1024, or 1536? What happens if you use fewer or more?"

**Answer:** These dimensions are tied to the Transformer's `d_model` parameter — the embedding dimension equals the model width. The choice is a trade-off:
- **Fewer dimensions** (256-384): Cheaper to store and search (less memory, faster similarity computation), but may not capture enough nuance for complex semantics. Works for simple tasks.
- **Standard dimensions** (768-1536): Good balance of quality and cost. Most embedding models target this range.
- **More dimensions** (3072+): Diminishing returns in quality but much higher storage and compute costs. A vector database with 1M documents at 3072 dimensions uses ~12GB in float32 vs. ~6GB at 1536.

Practically, **Matryoshka embedding models** (like text-embedding-3) let you truncate dimensions at inference time — you can use 256 dimensions for cheap search and 1536 for high-quality search with the same model.

---

## Further Reading

- [Efficient Estimation of Word Representations in Vector Space — Word2Vec (Mikolov et al., 2013)](https://arxiv.org/abs/1301.3781)
- [GloVe: Global Vectors for Word Representation (Pennington et al., 2014)](https://nlp.stanford.edu/pubs/glove.pdf)
- [SimCSE: Simple Contrastive Learning of Sentence Embeddings (Gao et al., 2021)](https://arxiv.org/abs/2104.08821)
- [Matryoshka Representation Learning (Kusupati et al., 2022)](https://arxiv.org/abs/2205.13147)

---

<p align="center">
  <a href="../../README.md">← Back to all questions</a>
</p>
