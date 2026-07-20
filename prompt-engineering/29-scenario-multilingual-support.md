# Scenario 6: Adding Multilingual Support

> **The Scenario:** "Your AI system works in English but fails for other languages. How do you add multilingual support?"

---

## Solutions

1. **Language detection + routing** — Detect input language → route to language-specific prompts with examples in that language
2. **Translate-process-translate** — Translate input to English → process with English-optimized prompt → translate output to target language
3. **Multilingual few-shot examples** — Include examples in each target language in the prompt
4. **Multilingual fine-tuning** — Fine-tune on labeled data in all target languages
5. **Multilingual embedding models** — For RAG, use models like multilingual-e5 or Cohere multilingual that embed all languages in the same space

## Practical Architecture

```
User Input → Language Detection (fasttext/CLD3)
  → English: Process with English prompt
  → Major language (FR/DE/ES/JA/ZH): Process with target-language prompt
  → Low-resource: Translate to English → Process → Translate back
```

## Scenario Examples
### A: A global customer support bot needs French, German, Japanese. For each language: (1) translate the system prompt, (2) provide 3 native-language few-shot examples, (3) test with native speakers. Each language version is independently evaluated.
### B: A RAG system serves Korean users but the knowledge base is English. Solution: use multilingual embeddings for retrieval (Korean query finds relevant English documents) + generate the response in Korean using the English context.

## Follow-Up Questions
### Q1: "Which approach gives the best quality?"
**Answer:** Native-language prompting > translate-process-translate > English prompting with language instruction. Native prompting preserves nuance but requires per-language prompt engineering. Translate-process-translate leverages English strength but loses nuance in translation.

### Q2: "How do you evaluate multilingual quality?"
**Answer:** You need native speakers for each language. Automated metrics (BLEU, BERTScore) are language-specific and can be misleading. For production, track user satisfaction by language and investigate any language-specific drops.

### Q3: "What about right-to-left languages (Arabic, Hebrew)?"
**Answer:** LLMs handle RTL text well at the content level (tokenization and generation work fine). Challenges are in UI rendering, mixed LTR/RTL text, and tokenization efficiency (Arabic uses more tokens per word). Ensure your prompt templates handle mixed directionality.

---

<p align="center"><a href="../README.md">← Back to all questions</a></p>
