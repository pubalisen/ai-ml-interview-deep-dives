# Part 26: Encoder-Only vs. Decoder-Only vs. Encoder-Decoder

> **The Question:** "What is the difference between encoder-only, decoder-only, and encoder-decoder Transformer architectures?"

---

## What They're Actually Testing

The interviewer wants you to explain the three architectural variants, their attention patterns, training objectives, and which tasks each excels at.

---

## The Technical Breakdown

| Architecture | Attention | Training Objective | Strengths | Examples |
|-------------|-----------|-------------------|-----------|----------|
| **Encoder-only** | Bidirectional (every token sees every token) | Masked Language Modeling (MLM) — predict randomly masked tokens | Classification, NER, embeddings, understanding tasks | BERT, RoBERTa, DeBERTa |
| **Decoder-only** | Causal (each token sees only previous tokens) | Next-token prediction (CLM) — predict the next token | Text generation, chat, code, reasoning | GPT-4, Claude, LLaMA, Gemini |
| **Encoder-Decoder** | Encoder: bidirectional; Decoder: causal + cross-attention | Seq2seq — encoder reads input, decoder generates output | Translation, summarization, structured transformation | T5, BART, mT5, Whisper |

### Why Decoder-Only Won

Almost all modern LLMs are decoder-only. Key reasons:
1. **Simpler architecture** — one set of parameters, no encoder/decoder split
2. **Unified input/output** — prompt and response are a single sequence
3. **Scalable** — scaling laws showed decoder-only models improve most predictably
4. **Versatile** — can do generation AND understanding via prompting
5. **Efficient training** — every token in the sequence contributes to the loss (CLM), versus only 15% masked tokens in MLM

---

## Accuracy Check

| Claim | Verified? |
|-------|-----------|
| BERT is encoder-only with bidirectional attention | ✅ Devlin et al. (2019) |
| GPT, LLaMA, Claude are decoder-only with causal attention | ✅ Architecture documentation |
| T5 is encoder-decoder | ✅ Raffel et al. (2020) |
| Decoder-only dominates modern LLM development | ✅ Industry-wide trend since GPT-3 |

---

## Scenario Examples

### Scenario 1: Choosing an Architecture for Semantic Search

A company needs sentence embeddings for semantic search. Encoder-only models (BERT variants) produce better embeddings because bidirectional attention creates richer contextual representations — each token "sees" the full sentence. Decoder-only models produce poorer embeddings because causal masking means early tokens have limited context. The company uses a fine-tuned BERT-based embedding model (like BGE, E5) for search.

### Scenario 2: Building a Translation System

For translating French to English, an encoder-decoder model is natural — the encoder fully comprehends the French input (bidirectional attention), and the decoder generates English autoregressively. But with modern decoder-only models, you can achieve similar quality by simply prompting: "Translate to English: [French text]." The decoder-only approach is simpler to deploy but may be less efficient for high-volume translation.

---

## Follow-Up Questions

### Q1: "Can a decoder-only model do classification?"

**Answer:** Yes — by framing classification as generation. Prompt: "Classify this email as spam or not spam: [email]\nClassification:" The model generates "spam" or "not spam." This works well with large models but is less compute-efficient than BERT-style classifiers (which produce a single logit per class). For high-volume classification, a fine-tuned encoder-only model is faster and cheaper.

### Q2: "Why is BERT still used if decoder-only models are better?"

**Answer:** BERT-style models remain superior for specific tasks: (1) **embeddings** — bidirectional attention produces better representations, (2) **classification at scale** — a 110M BERT is 100× cheaper than GPT-4 for classification, (3) **token-level tasks** — NER, POS tagging, where you need per-token predictions. The "decoder-only is better" applies to **general-purpose generation and reasoning**, not to every NLP task.

### Q3: "What is a prefix LM?"

**Answer:** A prefix LM is a hybrid: the prefix (input) uses **bidirectional attention** (like an encoder), and the continuation uses **causal attention** (like a decoder). It's a single model with mixed masking. PaLM-2 and UL2 explored this. The advantage: full contextualization of the input (bidirectional) while still generating autoregressively. The disadvantage: slightly more complex masking logic and less adoption than pure decoder-only.

---

<p align="center">
  <a href="../README.md">← Back to all questions</a>
</p>
