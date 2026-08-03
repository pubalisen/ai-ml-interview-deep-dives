# Part 4: Chunking Strategies

> **The Question:** "What are chunking strategies, and how do you choose the right chunk size?"

---

## The Technical Breakdown

### Why Chunking Matters

Chunking is the most underrated decision in RAG. Get it wrong, and your retrieval fails — either returning incomplete context (chunks too small) or diluting relevant information with noise (chunks too large).

```
Too small (100 tokens):  "The refund policy allows..." — Missing the actual policy details
Too large (2000 tokens): Contains refund policy + shipping policy + warranty — Diluted, less relevant
Just right (500 tokens): Complete refund policy section — Focused and self-contained
```

### The Four Main Chunking Strategies

#### 1. Fixed-Size Chunking
Split by character or token count with overlap:

```python
def fixed_chunk(text, size=512, overlap=50):
    chunks = []
    for i in range(0, len(text), size - overlap):
        chunks.append(text[i:i + size])
    return chunks
```

| Pros | Cons |
|------|------|
| Simple, fast, predictable | Cuts mid-sentence, mid-paragraph |
| Easy to reason about token budgets | Ignores document structure |
| Uniform chunk sizes | Important context may span two chunks |

#### 2. Recursive Character Splitting
Tries progressively smaller separators until chunks fit the size limit:

```python
separators = ["\n\n", "\n", ". ", ", ", " ", ""]

# Try splitting by double newline (paragraphs) first
# If chunks are still too large, split by single newline
# Then by sentence, then by comma, then by word
```

This preserves natural document structure — paragraphs stay together when possible.

#### 3. Semantic Chunking
Uses embedding similarity to find natural breakpoints:

```python
# 1. Split into sentences
# 2. Embed each sentence
# 3. Compare adjacent sentence embeddings
# 4. Split where similarity drops below threshold

sentences = split_into_sentences(text)
embeddings = embed(sentences)

for i in range(1, len(embeddings)):
    similarity = cosine_similarity(embeddings[i-1], embeddings[i])
    if similarity < threshold:
        # Topic shift detected — split here
        create_chunk_boundary(i)
```

#### 4. Document-Structure-Aware Chunking
Uses the document's native structure (headers, sections, pages):

```python
# Markdown: Split by ## headers
# HTML: Split by <section>, <article>, <h2>
# PDF: Split by page boundaries or detected sections
# Code: Split by function/class definitions
```

### Choosing the Right Chunk Size

| Factor | Smaller Chunks (100-300 tokens) | Larger Chunks (500-1500 tokens) |
|--------|-------------------------------|-------------------------------|
| **Precision** | Higher — focused retrieval | Lower — more noise per chunk |
| **Recall** | Lower — may miss context | Higher — more complete context |
| **Context window cost** | Cheap — fit more chunks | Expensive — fewer chunks fit |
| **Best for** | Factoid QA, specific lookups | Summarization, complex reasoning |
| **Embedding quality** | May lose context | Embedding models handle well |

### The Overlap Factor

```
Chunk 1: "...the enterprise plan costs $499/month. It includes"
Chunk 2: "costs $499/month. It includes unlimited API calls and..."

Overlap = 50 tokens
```

Overlap (typically 10-20% of chunk size) ensures that information at chunk boundaries isn't lost. Without overlap, a question about "enterprise plan API limits" might miss the answer that spans two chunks.

### The Golden Rules

1. **Chunk size should match your retrieval unit** — what's the smallest self-contained piece of information a user would need?
2. **Test empirically** — no universal best chunk size exists. Test 256, 512, 1024 on your actual queries.
3. **Preserve metadata** — every chunk should carry source document, section, page number.
4. **Consider your embedding model's context window** — if your model maxes at 512 tokens, chunks of 1024 tokens get truncated.

---

## Accuracy Check
| Claim | Verified? |
|-------|-----------|
| Chunk size significantly impacts retrieval quality | ✅ LlamaIndex chunking experiments, Pinecone benchmarks |
| Recursive splitting preserves document structure better than fixed-size | ✅ LangChain documentation, empirical testing |
| Semantic chunking detects topic boundaries via embedding similarity | ✅ Greg Kamradt's "semantic chunking" approach, LlamaIndex SemanticSplitter |
| Overlap prevents information loss at chunk boundaries | ✅ Standard practice across RAG implementations |

## Scenario Examples
### A: An API documentation system uses document-structure-aware chunking — each endpoint's docs (description + parameters + example + response) become one chunk. This works because each endpoint is a natural retrieval unit. Fixed-size chunking would have split endpoint descriptions mid-parameter-table, making retrieved chunks useless.
### B: A legal contract review system uses semantic chunking because contract clauses vary wildly in length (some are 50 words, others are 500). Fixed-size chunking cuts clauses in half. Semantic chunking detects clause boundaries by measuring embedding similarity drops at section transitions. Result: retrieval precision improves from 62% to 84%.

## Follow-Up Questions
### Q1: "How do you know your chunk size is wrong?"
**Answer:** Three symptoms: (1) **Low retrieval recall** — relevant info exists but isn't being retrieved (chunks too large, embedding averages out the signal). (2) **Retrieved chunks lack context** — the LLM generates "I need more context" or gives partial answers (chunks too small). (3) **High redundancy** — multiple chunks contain the same info (excessive overlap). Measure with retrieval evaluation: calculate precision@k and recall@k across a test set of queries.

### Q2: "Should you chunk code differently than prose?"
**Answer:** Absolutely. Code has explicit structure — functions, classes, methods. The best approach: chunk by function or class definition, preserving the entire unit. A function split across two chunks is useless. For large functions, split by logical blocks (setup, processing, return). Include docstrings and type signatures with each chunk for context. Tools like `tree-sitter` parse code ASTs for structure-aware splitting.

### Q3: "What about chunking tables and structured data?"
**Answer:** Tables are the hardest chunking problem. Options: (1) Keep the entire table as one chunk (if it fits). (2) Convert to natural language ("Row 3: Product X costs $49 with 100 users included"). (3) Store table metadata separately and use SQL/Pandas for retrieval. (4) Use multi-modal embeddings that understand tabular structure. Never split a table row across chunks — the data becomes meaningless.

---

<p align="center"><a href="../README.md">← Back to all questions</a></p>
