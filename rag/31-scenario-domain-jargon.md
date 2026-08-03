# Scenario 6: Domain-Specific Jargon

> **The Scenario:** "Your RAG system is failing on domain-specific jargon. How do you fix it?"

---

## Solutions

1. **Hybrid search** — Add BM25 keyword search alongside vector search. Domain jargon ("EBITDA," "ICD-10," "kubectl") needs exact matching that keyword search excels at.
2. **Domain-specific embedding model** — Switch from a general-purpose embedding model to one trained on your domain (PubMedBERT for medical, FinBERT for finance, CodeBERT for code).
3. **Fine-tune embeddings** — Generate (query, relevant_doc) pairs from your domain data and fine-tune a general embedding model. Even 1,000-5,000 pairs can significantly improve domain-specific retrieval.
4. **Synonym expansion** — Maintain a domain glossary and expand queries with synonyms: "MI" → "MI, myocardial infarction, heart attack."
5. **Custom tokenizer** — If your domain uses specialized terms that general tokenizers split incorrectly ("COVID-19" → "CO", "VID", "-", "19"), use a domain-adapted tokenizer.
6. **Glossary-enriched chunks** — Append glossary definitions to chunks during indexing: "EBITDA (Earnings Before Interest, Taxes, Depreciation, and Amortization)."

## Scenario Examples
### A: A medical chatbot fails when doctors ask "What's the dosage for abx in pediatric UTI?" because the embedding model doesn't understand "abx" (antibiotics) or "UTI" (urinary tract infection). The vector search returns generic infection docs instead of specific antibiotic guidelines. Fix: (1) add a medical abbreviation dictionary that expands "abx" → "antibiotics" and "UTI" → "urinary tract infection," (2) switch to PubMedBERT embeddings, (3) add BM25 for exact drug name matching. Retrieval recall improves from 45% to 82%.
### B: A DevOps knowledge base fails on "k8s crashloopbackoff pod restart" because the general embedding model doesn't understand Kubernetes shorthand. It retrieves general container docs instead of the specific error troubleshooting guide. Fix: (1) hybrid search catches "crashloopbackoff" as a keyword, (2) fine-tune embeddings on 2,000 DevOps (query, doc) pairs, (3) enrich chunks with expanded terms during indexing: "CrashLoopBackOff (a Kubernetes pod status indicating repeated container crashes)."

## Follow-Up Questions
### Q1: "How do you build a domain glossary efficiently?"
**Answer:** Three approaches: (1) **Extract from documents** — use an LLM to scan your corpus and extract all domain-specific terms with definitions. Prompt: "List all specialized terms in this document with their definitions." (2) **Crowdsource from users** — when users search for terms that return no results, log those terms and have domain experts add definitions. (3) **Use existing ontologies** — medical (MeSH, SNOMED), legal (legal thesaurus), financial (GAAP glossary). Automate synonym expansion using the glossary.

### Q2: "How much does fine-tuning embedding models help for domain-specific retrieval?"
**Answer:** Typically 15-30% improvement in retrieval recall on domain-specific queries. The key is training data quality. You need (query, relevant_document) pairs from your domain. Generate them by: (1) having domain experts write 500+ queries and label relevant docs, (2) using an LLM to generate synthetic queries from your documents ("What question would this paragraph answer?"), (3) mining search logs for click-through data. Fine-tuning is the single biggest quality lever for domain-specific RAG.

### Q3: "Can you solve this with prompt engineering instead of infrastructure changes?"
**Answer:** Partially. You can add a glossary to the system prompt: "When you see 'abx,' treat it as 'antibiotics.'" But this has limits: (1) the glossary consumes context window space, (2) it only helps generation, not retrieval (the wrong chunks are still being fetched), (3) you can't fit 10,000 domain terms in a prompt. Prompt engineering is a quick patch; hybrid search + domain embeddings are the permanent fix.

---

<p align="center"><a href="../README.md">← Back to all questions</a></p>
