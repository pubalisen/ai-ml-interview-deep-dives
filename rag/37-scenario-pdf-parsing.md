# Scenario 12: PDF Parsing Challenges

> **The Scenario:** "Your RAG system needs to handle PDF documents with complex tables, multi-column layouts, and embedded images. How do you approach this?"

---

## Solutions

1. **Use advanced PDF parsers** — `PyMuPDF` for simple text, `Unstructured.io` for complex layouts, `Azure Document Intelligence` for enterprise-grade extraction with table/image/layout understanding.
2. **Table-specific extraction** — Detect tables in PDFs and extract them as structured data (markdown or CSV) rather than flattened text.
3. **Layout-aware chunking** — Respect the document's visual structure: headers, columns, sidebars, footnotes. Don't mix content from different columns into the same chunk.
4. **Image extraction + captioning** — Extract embedded images, run them through a vision model (GPT-4o) for captions, and index the captions alongside the surrounding text.
5. **Multi-column handling** — Detect column boundaries and read text in proper reading order (left column top-to-bottom, then right column top-to-bottom), not across columns.
6. **Preprocessing validation** — After extraction, spot-check a sample of documents to verify text quality. Garbage-in from bad PDF parsing is the #1 cause of RAG quality issues.

### PDF Parser Comparison

| Parser | Tables | Layout | Images | Speed | Cost |
|--------|--------|--------|--------|-------|------|
| **PyMuPDF (fitz)** | Basic | Poor | Extract only | Fast | Free |
| **pdfplumber** | Good | Basic | No | Medium | Free |
| **Unstructured.io** | Good | Good | Caption support | Medium | Free/API |
| **Azure Doc Intelligence** | Excellent | Excellent | OCR + captions | Medium | $1.50/1K pages |
| **LlamaParse** | Good | Good | Good | Medium | API |
| **Docling (IBM)** | Excellent | Excellent | Good | Medium | Free |

```python
# Multi-stage PDF processing
def process_pdf(pdf_path):
    # Stage 1: Extract with layout awareness
    doc = UnstructuredDocument.from_pdf(pdf_path, strategy="hi_res")
    
    elements = []
    for element in doc.elements:
        if element.type == "Table":
            # Convert table to markdown for structured retrieval
            markdown = element.to_markdown()
            elements.append({"type": "table", "content": markdown})
        elif element.type == "Image":
            # Caption the image with vision model
            caption = vision_model.describe(element.image)
            elements.append({"type": "image", "content": caption})
        elif element.type == "NarrativeText":
            elements.append({"type": "text", "content": element.text})
    
    # Stage 2: Chunk respecting element boundaries
    chunks = chunk_elements(elements, max_tokens=512)
    return chunks
```

## Scenario Examples
### A: A financial firm indexes 10,000 quarterly earnings PDFs. Each PDF has revenue tables, bar charts, and multi-column executive summaries. Using basic `PyMuPDF`, tables come out as jumbled text ("Revenue 2024 Q3 Q2 Q1 45.2 42.1 39.8" instead of structured columns) and multi-column text mixes paragraphs from adjacent columns. Fix: switch to Azure Document Intelligence — tables are extracted as proper markdown with headers, charts are OCR'd and captioned, and column reading order is correctly resolved. RAG answer accuracy on financial questions improves from 52% to 87%.
### B: A legal firm processes 500-page contracts with headers, footers, page numbers, and footnotes. Naive text extraction includes page headers ("CONFIDENTIAL — Page 47 of 523") in every chunk, diluting the embedding quality. Fix: Unstructured.io with `hi_res` strategy detects and strips headers, footers, and page numbers. Footnotes are linked to their reference paragraphs. The clean text produces focused embeddings and eliminates "CONFIDENTIAL — Page X" from appearing in LLM answers.

## Follow-Up Questions
### Q1: "How do you handle scanned PDFs (image-only, no selectable text)?"
**Answer:** OCR is required. Options: (1) Tesseract (free, good for clean scans), (2) Azure Document Intelligence (best for complex layouts), (3) Google Cloud Vision API (strong multilingual OCR). For scanned documents, expect lower accuracy than digital PDFs. Always post-process OCR output: spell-check, format cleanup, confidence scoring. Documents with OCR confidence < 90% should be flagged for human review.

### Q2: "What about PDFs with mixed languages or non-Latin scripts?"
**Answer:** Use a multilingual OCR engine (Azure Document Intelligence supports 300+ languages). For embedding, use a multilingual model (Cohere embed-v3, GTE-Qwen2). Key challenge: some PDFs mix languages within a single page (English headers with Japanese body text). The OCR engine must correctly detect language per region. Test with representative samples from your actual document corpus before committing to a parser.

### Q3: "How do you validate that PDF extraction is working correctly?"
**Answer:** Build a validation pipeline: (1) Process 50 representative PDFs. (2) For each, manually extract 5 key facts (a price, a date, a table cell, a paragraph). (3) Run your extraction pipeline and check if those facts are present and correct in the output. Calculate extraction accuracy. Target: >95% fact extraction accuracy. Common failures: garbled table data, incorrect reading order, missing text from sidebars, and images without captions. Fix issues per document type.

---

<p align="center"><a href="../README.md">← Back to all questions</a></p>
