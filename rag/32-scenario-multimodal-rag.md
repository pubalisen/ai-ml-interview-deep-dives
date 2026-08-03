# Scenario 7: Multimodal RAG

> **The Scenario:** "You need to extend your text RAG system to handle images, tables, and diagrams. How?"

---

## Solutions

1. **Image captioning** — Use a vision model (GPT-4o, Gemini, LLaVA) to generate text descriptions of images, then index the captions alongside text chunks.
2. **Table serialization** — Convert tables to natural language or structured markdown, then embed as regular text chunks.
3. **Multimodal embeddings** — Use models that embed both text and images into the same vector space (CLIP, OpenAI multimodal embeddings), enabling cross-modal retrieval.
4. **OCR + Layout analysis** — For scanned documents, use OCR (Tesseract, DocTR, Azure Document Intelligence) to extract text while preserving layout structure.
5. **Separate modality pipelines** — Route image-related queries to an image index and text queries to a text index. Combine results before generation.
6. **Vision-language generation** — Pass retrieved images directly to a multimodal LLM (GPT-4o, Gemini) along with text context for generation.

```python
# Image captioning pipeline
for page in pdf_pages:
    for image in page.images:
        caption = vision_model.describe(image)
        chunk = f"[Image on page {page.num}]: {caption}"
        embed_and_store(chunk, metadata={"type": "image", "page": page.num})

# Table extraction
for table in page.tables:
    markdown = table.to_markdown()  # Preserve structure
    natural_lang = table.to_natural_language()  # Readable sentences
    embed_and_store(markdown, metadata={"type": "table"})
```

## Scenario Examples
### A: An engineering knowledge base contains architectural diagrams, flowcharts, and system design docs. Engineers ask "How does the payment service communicate with the order service?" The diagram showing the microservice architecture is the best answer, but text-only RAG can't retrieve it. Fix: GPT-4o generates a detailed caption for each diagram ("This diagram shows a microservice architecture with Payment Service communicating via gRPC to Order Service, which publishes events to a Kafka topic consumed by Inventory Service"). The caption is embedded and retrieves correctly.
### B: A financial analysis platform indexes annual reports containing revenue tables, pie charts, and earnings graphs. "What was Tesla's gross margin trend from 2020-2024?" requires a table. Fix: tables are extracted via Document Intelligence API and serialized to markdown. Charts are captioned by GPT-4o ("Bar chart showing Tesla's gross margin: 21.0% (2020), 25.3% (2021), 25.6% (2022), 18.2% (2023), 17.9% (2024)"). Both the table data and chart captions are embedded and retrieved for comprehensive answers.

## Follow-Up Questions
### Q1: "When should you use multimodal embeddings vs captioning?"
**Answer:** Captioning is simpler and works with existing text-only infrastructure. Multimodal embeddings (CLIP) enable direct image-to-text retrieval without captioning but require a separate embedding model and may miss details that captions capture. Use captioning for: diagrams, charts, screenshots where text descriptions are meaningful. Use multimodal embeddings for: photo search, visual similarity (e-commerce product images), when you need image-to-image retrieval.

### Q2: "How do you handle tables that are too complex for text serialization?"
**Answer:** Three strategies: (1) **Cell-level indexing** — each table cell becomes a chunk with its row/column header as metadata. (2) **SQL approach** — load the table into a database and use Text-to-SQL for queries. (3) **Hybrid** — store both the markdown table AND a natural language summary. The summary retrieves for high-level questions; the full table is passed to the LLM for specific cell lookups. For very large tables (>100 rows), SQL is the only practical approach.

### Q3: "What about video content in RAG?"
**Answer:** Videos are the hardest modality. Approaches: (1) **Transcription** — extract audio, transcribe with Whisper, index the transcript. Works for lectures, presentations. (2) **Frame sampling** — extract keyframes (every N seconds or on scene change), caption each frame, index captions with timestamps. (3) **Chapter segmentation** — split video into logical chapters (via transcript analysis), create chunk per chapter with transcript + frame captions. Always include timestamps so users can jump to the relevant moment.

---

<p align="center"><a href="../README.md">← Back to all questions</a></p>
