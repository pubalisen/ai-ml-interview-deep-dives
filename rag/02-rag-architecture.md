# Part 2: Architecture of a Basic RAG System

> **The Question:** "Explain the architecture of a basic RAG system."

---

## The Technical Breakdown

### The Two-Phase Architecture

Every RAG system has two distinct phases:

```
PHASE 1: INDEXING (Offline — runs once, updated periodically)
══════════════════════════════════════════════════════════════

  Raw Documents
       │
       ▼
  ┌──────────┐     ┌──────────┐     ┌───────────────┐     ┌──────────────┐
  │  Loader   │ →  │  Chunker  │ →  │  Embedding     │ →  │  Vector DB    │
  │ (PDF, web,│    │ (split    │    │  Model         │    │  (Pinecone,   │
  │  DB, API) │    │  into     │    │  (text→vec)    │    │   Weaviate,   │
  └──────────┘    │  chunks)  │    └───────────────┘    │   Qdrant)     │
                   └──────────┘                          └──────────────┘

PHASE 2: RETRIEVAL + GENERATION (Online — every user query)
══════════════════════════════════════════════════════════════

  User Query
       │
       ▼
  ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
  │  Embed Query  │ →  │  Vector      │ →  │  Top-K        │
  │  (same model) │    │  Similarity  │    │  Chunks       │
  └──────────────┘    │  Search      │    └──────┬───────┘
                       └──────────────┘           │
                                                  ▼
                                        ┌──────────────────┐
                                        │  Prompt Assembly  │
                                        │  System + Context │
                                        │  + User Query     │
                                        └────────┬─────────┘
                                                 │
                                                 ▼
                                        ┌──────────────────┐
                                        │  LLM Generation   │
                                        │  (GPT-4, Claude,  │
                                        │   Llama, etc.)    │
                                        └──────────────────┘
                                                 │
                                                 ▼
                                            Response
```

### Component Breakdown

#### 1. Document Loader
Ingests raw data from various sources:
- **Files:** PDF, DOCX, HTML, Markdown, CSV
- **APIs:** Confluence, Notion, Slack, Google Drive
- **Databases:** SQL dumps, exported records
- **Web:** Crawled pages, sitemaps

#### 2. Chunker (Text Splitter)
Splits documents into retrieval-sized pieces (typically 256-1024 tokens):
```
Full Document (10,000 tokens)
    ↓
Chunk 1 (512 tokens) — "Section: Product Overview..."
Chunk 2 (512 tokens) — "Section: Pricing Plans..."
Chunk 3 (512 tokens) — "Section: API Reference..."
    ...
```

#### 3. Embedding Model
Converts each chunk into a dense vector (768-3072 dimensions):
```
"Our enterprise plan costs $499/month" → [0.023, -0.841, 0.127, ..., 0.445]
```
Popular models: OpenAI `text-embedding-3-small`, Cohere `embed-v3`, open-source `bge-large`, `E5-mistral`.

#### 4. Vector Database
Stores embeddings with metadata and enables fast approximate nearest neighbor (ANN) search:
```json
{
  "id": "chunk_042",
  "vector": [0.023, -0.841, ...],
  "metadata": {
    "source": "pricing.pdf",
    "page": 3,
    "last_updated": "2024-11-15"
  },
  "text": "Our enterprise plan costs $499/month..."
}
```

#### 5. Retriever
At query time, embeds the user question with the same embedding model and finds top-k similar chunks via cosine similarity or dot product.

#### 6. Prompt Assembly
Constructs the final prompt:
```
System: You are a helpful assistant. Answer ONLY based on the
provided context. If the context doesn't contain the answer,
say "I don't have that information."

Context:
[Chunk 1 text]
[Chunk 2 text]
[Chunk 3 text]

User: What does the enterprise plan cost?
```

#### 7. LLM Generator
Produces the final answer grounded in the retrieved context.

---

## Accuracy Check
| Claim | Verified? |
|-------|-----------|
| RAG has separate indexing and retrieval phases | ✅ Lewis et al. (2020), standard RAG architecture |
| Embedding models convert text to dense vectors for similarity search | ✅ Reimers & Gurevych (2019), Sentence-BERT |
| Vector databases use ANN for fast similarity search | ✅ FAISS (Meta), ScaNN (Google), HNSW algorithm |

## Scenario Examples
### A: An HR chatbot indexes the employee handbook (200 pages). Loader extracts text from the PDF, chunker splits it into 400 chunks of ~500 tokens with 50-token overlap, embedding model converts each to a 1536-dim vector, and Pinecone stores them. When an employee asks "How many vacation days do I get?", the query is embedded, top-3 chunks about PTO policy are retrieved, and GPT-4 answers: "Full-time employees receive 20 vacation days per year" — citing page 47 of the handbook.
### B: A developer documentation system indexes 5,000 API reference pages. Each endpoint's docs become a chunk. When a developer asks "How do I authenticate with OAuth2?", the retriever returns the authentication guide chunks, and the LLM generates a step-by-step guide with code examples pulled directly from the docs.

## Follow-Up Questions
### Q1: "Why use the same embedding model for indexing and querying?"
**Answer:** The embedding model defines the vector space geometry. If you embed documents with Model A and queries with Model B, they live in different vector spaces — cosine similarity becomes meaningless. Both documents and queries must be embedded with the same model so that semantically similar text produces nearby vectors. If you switch embedding models, you must re-index everything.

### Q2: "What happens if you skip the chunking step and embed full documents?"
**Answer:** Two problems: (1) Most embedding models have a max input length (512-8192 tokens) — longer documents get truncated, losing information. (2) Even with long-context embedding models, full-document embeddings are too coarse — a 50-page document about many topics produces a single "average" vector that matches poorly with specific queries. Chunking ensures each vector represents a focused, retrievable unit of information.

### Q3: "How do you handle the prompt getting too long with too many retrieved chunks?"
**Answer:** Set a budget: if your LLM has a 128K context window, allocate ~4K for system prompt + instructions, ~2K for the user query + conversation history, and the rest for retrieved chunks. In practice, 3-10 chunks usually suffice. More chunks aren't always better — irrelevant chunks dilute the signal (the "lost in the middle" problem). Use re-ranking to ensure only the most relevant chunks make it into the prompt.

---

<p align="center"><a href="../README.md">← Back to all questions</a></p>
