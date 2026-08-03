# Part 5: Chunking Comparison

> **The Question:** "Compare fixed-size chunking, semantic chunking, and recursive chunking."

---

## The Technical Breakdown

### Head-to-Head Comparison

| Aspect | Fixed-Size | Recursive | Semantic |
|--------|-----------|-----------|----------|
| **How it splits** | Every N tokens/characters | Tries progressively smaller separators | Embedding similarity between adjacent segments |
| **Speed** | ⚡ Fastest | ⚡ Fast | 🐢 Slowest (requires embedding each sentence) |
| **Quality** | Basic — cuts mid-thought | Good — respects paragraphs/sentences | Best — detects topic boundaries |
| **Complexity** | Trivial to implement | Low — configurable separators | High — needs embedding model + threshold tuning |
| **Cost** | Free | Free | Embedding API calls for every sentence |
| **Best for** | Prototyping, uniform data | Most production use cases | High-value domains (legal, medical) |

### Fixed-Size Chunking in Detail

```
Input: "Paragraph about pricing. Paragraph about features. Paragraph about support."
                    ↓ (chunk_size=100 chars)
Chunk 1: "Paragraph about pricing. Paragra"  ← Cuts mid-word!
Chunk 2: "ph about features. Paragraph abo"
Chunk 3: "ut support."
```

**The problem:** No awareness of content structure. A sentence about pricing gets merged with one about features.

### Recursive Character Splitting in Detail

```python
# Try splitting by the largest separator first
separators = ["\n\n", "\n", ". ", " "]

Input: "Section 1: Pricing\n\nOur plans start at...\n\nSection 2: Features\n\nWe offer..."
                    ↓ (split on "\n\n" first)
Chunk 1: "Section 1: Pricing\nOur plans start at..."
Chunk 2: "Section 2: Features\nWe offer..."
```

**The advantage:** Preserves paragraph boundaries. Only falls back to sentence/word splitting when paragraphs exceed the max chunk size.

### Semantic Chunking in Detail

```
Sentence 1: "Our enterprise plan costs $499/month."          → embedding_1
Sentence 2: "It includes unlimited API calls."               → embedding_2
Sentence 3: "All plans come with 24/7 support."             → embedding_3
Sentence 4: "Our company was founded in 2019."              → embedding_4

Similarity(1,2) = 0.89  ← Same topic (pricing)
Similarity(2,3) = 0.72  ← Related (plan features)
Similarity(3,4) = 0.31  ← Topic shift! Split here.

Chunk 1: Sentences 1-3 (pricing and features)
Chunk 2: Sentence 4+ (company history)
```

### When to Use Each

```
Decision Tree:

Is this a prototype?
  └─ Yes → Fixed-size (just get it working)
  └─ No
      ├─ Does your data have natural structure (headers, paragraphs)?
      │   └─ Yes → Recursive splitting (respect that structure)
      │   └─ No → Semantic chunking (find topic boundaries)
      └─ Is retrieval quality critical (medical, legal, financial)?
          └─ Yes → Semantic chunking + manual review of results
```

### Production Recommendation

Most teams use **recursive splitting** as the default because it's the best quality-to-complexity ratio. Semantic chunking is reserved for high-value domains where the 10-20% quality improvement justifies the extra cost and complexity.

---

## Accuracy Check
| Claim | Verified? |
|-------|-----------|
| Fixed-size chunking can split mid-sentence, degrading retrieval quality | ✅ Empirically validated across benchmarks |
| Recursive splitting tries paragraph → sentence → word boundaries | ✅ LangChain RecursiveCharacterTextSplitter implementation |
| Semantic chunking uses embedding similarity to find topic boundaries | ✅ Greg Kamradt's semantic chunking, LlamaIndex SemanticSplitterNodeParser |
| Recursive splitting offers the best complexity-to-quality ratio | ✅ Industry consensus — default in LangChain, LlamaIndex |

## Scenario Examples
### A: A news aggregator indexes 10,000 articles daily. Articles have clear paragraph structure (headline, lead, body paragraphs). Recursive splitting on `\n\n` boundaries works perfectly — each chunk is a coherent paragraph or group of paragraphs. Semantic chunking would cost $50/day in embedding API calls for marginal improvement. Fixed-size would cut headlines from their context.
### B: A research lab indexes 500 dense scientific papers with no consistent formatting — some use section headers, others don't. Paragraphs mix methods, results, and discussion unpredictably. Semantic chunking is essential here — it detects when the paper shifts from "methods" to "results" even without explicit headers. The 3x cost is justified because incorrect retrieval could lead to wrong experimental conclusions.

## Follow-Up Questions
### Q1: "Can you combine multiple chunking strategies?"
**Answer:** Yes, and production systems often do. A common pattern: (1) Use document-structure-aware splitting first (by headers/sections), (2) then apply recursive splitting within each section if it exceeds the max chunk size. Another pattern: use semantic chunking for the initial split, then enforce a max size constraint — if a semantic chunk is too large, recursively split it.

### Q2: "How does chunk overlap work with each strategy?"
**Answer:** Fixed-size: overlap is straightforward (slide window by `chunk_size - overlap`). Recursive: overlap is added by including the last N tokens of the previous chunk at the start of the next. Semantic: overlap is less common because chunks are already semantically coherent, but you can include the last sentence of the previous chunk as context. Typical overlap: 10-20% of chunk size.

### Q3: "What's the impact of chunking strategy on embedding quality?"
**Answer:** Huge impact. Embedding models produce better vectors when the input is coherent and topically focused. A fixed-size chunk that mixes pricing info with support info produces a "muddled" embedding that partially matches both topics but strongly matches neither. Semantic chunks produce embeddings that clearly represent one topic, leading to sharper retrieval. This is why semantic chunking improves retrieval precision by 10-20% in benchmarks.

---

<p align="center"><a href="../README.md">← Back to all questions</a></p>
