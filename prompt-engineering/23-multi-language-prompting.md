# Part 23: Multi-Language Prompting

> **The Question:** "How do you handle multi-language prompting effectively?"

---

## The Technical Breakdown

### The Challenges

| Challenge | Description |
|-----------|-------------|
| **Uneven training data** | Models trained predominantly on English; other languages get less representation |
| **Cross-lingual transfer** | Prompt in English, expect output in Japanese — doesn't always work |
| **Cultural context** | Idioms, humor, formality levels differ across languages |
| **Tokenization efficiency** | Non-Latin scripts use more tokens per word (Chinese: ~2 tokens/character vs. English: ~1 token/4 characters) |
| **Evaluation** | Quality metrics are language-specific |

### Strategies

**1. Prompt in the target language**
```
# Better than English prompt for Japanese output
日本語で回答してください。次の文章を要約してください：
```

**2. English prompt + target language instruction**
```
Summarize the following text. Respond in French.
Text: [French text]
```

**3. Translate-then-process**
```
Step 1: Translate input to English
Step 2: Process in English (strongest capability)
Step 3: Translate output to target language
```

**4. Cross-lingual few-shot**
```
Provide examples in the target language to prime the model's language mode.
```

### Quality Hierarchy (typical)

```
English > Major European > CJK > Low-resource languages
```

---

## Scenario Examples
### A: A support bot for a global SaaS handles queries in 20 languages. Strategy: detect language → if English, process directly. If other, translate to English → process → translate back. This leverages the model's strongest (English) capabilities while supporting all languages.
### B: A Japanese legal assistant prompts entirely in Japanese with Japanese few-shot examples. This outperforms English prompting + translation because legal terminology requires precise language-specific phrasing.

## Follow-Up Questions
### Q1: "Does the translate-then-process approach lose nuance?"
**Answer:** Yes — translation introduces errors, especially for domain-specific terms, idioms, and cultural context. For high-stakes applications (legal, medical), prompting natively in the target language is safer. For general-purpose tasks, translate-then-process is often good enough.

### Q2: "How do you evaluate multilingual quality?"
**Answer:** Use native speakers for evaluation, not back-translation. Automated metrics (BLEU, BERTScore) have language-specific biases. For production, A/B test with native-speaking users and track satisfaction by language.

### Q3: "What about code-switching (mixing languages)?"
**Answer:** LLMs handle code-switching reasonably well for major language pairs (English-Spanish, English-Chinese). For the prompt, be explicit: "The user may mix English and Hindi. Always respond in the language the user is currently using." Without guidance, models default to English when encountering mixed input.

---

<p align="center"><a href="../README.md">← Back to all questions</a></p>
