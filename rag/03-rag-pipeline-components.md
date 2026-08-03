# Part 3: Key Components of a RAG Pipeline

> **The Question:** "What are the key components of a RAG pipeline?"

---

## The Technical Breakdown

### The Seven Core Components

A production RAG pipeline has seven critical components, each with specific engineering decisions:

```
┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
│1. Data    │→│2. Chunking│→│3. Embedding│→│4. Vector  │
│  Ingestion│  │  Strategy │  │  Model    │  │  Store   │
└──────────┘  └──────────┘  └──────────┘  └──────────┘
                                               │
User Query ──→ ┌──────────┐  ┌──────────┐     │
               │5. Retriever│←┘          │     │
               └─────┬──────┘           │     │
                     ▼                   │     │
              ┌──────────┐  ┌──────────┐│
              │6. Prompt  │→│7. LLM     ││
              │  Builder  │  │  Generator││
              └──────────┘  └──────────┘│
```

### 1. Data Ingestion Layer

Responsible for extracting clean text from source documents:

| Source Type | Tool | Challenge |
|-------------|------|-----------|
| PDF | `PyMuPDF`, `Unstructured`, `DocTR` | Tables, images, multi-column layouts |
| HTML | `BeautifulSoup`, `Trafilatura` | Boilerplate removal, nav/footer noise |
| DOCX | `python-docx` | Embedded images, tracked changes |
| Database | SQL queries | Schema understanding, joins |
| API | Custom connectors | Rate limits, pagination, auth |

**Production concern:** Garbage in, garbage out. If your PDF parser misreads a table, the LLM generates wrong answers with high confidence.

### 2. Chunking Strategy

How you split documents determines retrieval quality:

```python
# Fixed-size chunking (simplest)
chunks = split_text(doc, chunk_size=512, overlap=50)

# Recursive character splitting (LangChain default)
splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=200,
    separators=["\n\n", "\n", ". ", " "]
)

# Semantic chunking (best quality, highest cost)
chunks = semantic_split(doc, embedding_model, threshold=0.8)
```

### 3. Embedding Model

Converts text chunks into dense vectors. Key selection criteria:

| Criteria | Consideration |
|----------|--------------|
| **Dimension** | 384 (MiniLM) → 3072 (OpenAI large). Higher = better quality, more storage |
| **Max tokens** | 512 (most models) → 8192 (newer models). Must exceed chunk size |
| **Domain fit** | General-purpose vs domain-specific (code, legal, medical) |
| **Speed** | Local inference (BGE) vs API calls (OpenAI). Latency vs cost |
| **Multilingual** | If your docs are multi-language, use a multilingual model |

### 4. Vector Store

Stores and indexes embeddings for fast retrieval:

| Store | Type | Best For |
|-------|------|----------|
| **Pinecone** | Managed cloud | Production, zero ops |
| **Weaviate** | Open-source | Hybrid search, multi-modal |
| **Qdrant** | Open-source | High performance, filtering |
| **pgvector** | Postgres extension | Already using Postgres |
| **Chroma** | In-memory | Prototyping, small datasets |
| **FAISS** | Library | Research, custom pipelines |

### 5. Retriever

Fetches the most relevant chunks for a given query:

- **Dense retrieval** — Embed the query, find nearest vectors (semantic similarity)
- **Sparse retrieval** — BM25/TF-IDF keyword matching
- **Hybrid** — Combine dense + sparse with Reciprocal Rank Fusion (RRF)
- **Re-ranked** — Retrieve top-50 with fast search, re-rank to top-5 with a cross-encoder

### 6. Prompt Builder

Assembles the final prompt from system instructions, retrieved context, and user query:

```
[System Instructions]
- Role definition
- Output format constraints
- Citation instructions

[Retrieved Context]
- Chunk 1 (source: pricing.pdf, page 3)
- Chunk 2 (source: faq.md, section: billing)
- Chunk 3 (source: terms.pdf, page 12)

[User Query]
- The actual question

[Output Instructions]
- "If the context doesn't contain the answer, say so"
- "Cite your sources"
```

### 7. LLM Generator

Generates the final answer. Key decisions:
- **Model choice:** GPT-4o for quality, GPT-4o-mini for cost, Claude for long context, Llama for privacy
- **Temperature:** 0.0-0.3 for factual QA (low creativity)
- **Max tokens:** Set based on expected answer length
- **Streaming:** Stream tokens for perceived latency reduction

---

## Accuracy Check
| Claim | Verified? |
|-------|-----------|
| RAG pipelines have distinct ingestion, retrieval, and generation stages | ✅ LlamaIndex, LangChain architecture docs |
| Hybrid retrieval (dense + sparse) outperforms either alone | ✅ Luan et al. (2021), hybrid search benchmarks |
| Cross-encoder re-ranking improves retrieval precision | ✅ Nogueira & Cho (2019), MS MARCO results |

## Scenario Examples
### A: Building an internal knowledge assistant for a 500-person company. Data ingestion connects to Confluence (5,000 pages), Google Drive (10,000 docs), and Slack (archived channels). Each source has a different connector. Chunks are 500 tokens with 100-token overlap. OpenAI `text-embedding-3-small` embeds into Pinecone. Hybrid retrieval (vector + BM25) fetches top-5 chunks. GPT-4o-mini generates answers at $0.15/1M input tokens. Total cost: ~$200/month for 10K queries/day.
### B: A medical literature search tool ingests 2 million PubMed abstracts. Data ingestion uses the PubMed API. Because medical terminology matters, they use a domain-specific embedding model (`PubMedBERT`). The vector store is Qdrant (self-hosted for HIPAA compliance). The retriever uses hybrid search because exact drug names need keyword matching while symptom descriptions need semantic search.

## Follow-Up Questions
### Q1: "Which component has the biggest impact on RAG quality?"
**Answer:** The retriever — by a significant margin. If you retrieve the wrong chunks, even the best LLM will generate wrong answers (or hallucinate trying to answer without relevant context). In practice: 80% of RAG failures trace back to retrieval issues (wrong chunks, missing chunks, irrelevant chunks). Invest the most engineering effort in chunking strategy, embedding model selection, and retrieval logic.

### Q2: "How do the components differ between a prototype and production RAG system?"
**Answer:** Prototype: Chroma (in-memory), fixed-size chunking, single embedding model, simple top-k retrieval, no monitoring. Production adds: managed vector DB with backups, semantic chunking, hybrid retrieval + re-ranking, metadata filtering, query caching, observability (log every retrieval + generation), fallback models, rate limiting, access control, and automated evaluation pipelines.

### Q3: "Can you build RAG without a vector database?"
**Answer:** Yes, but it's less scalable. Alternatives: (1) BM25/Elasticsearch for keyword search only — works well for exact-match queries but misses semantic similarity. (2) Colbert-style late interaction — stores token-level embeddings and does fine-grained matching. (3) Full-text search with LLM re-ranking — retrieve with SQL LIKE/regex, then use an LLM to rank relevance. For production semantic search at scale, vector databases are the standard.

---

<p align="center"><a href="../README.md">← Back to all questions</a></p>
