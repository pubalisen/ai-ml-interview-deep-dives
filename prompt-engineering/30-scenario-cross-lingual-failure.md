# Scenario 7: Cross-Lingual Transfer Failure

> **The Scenario:** "Your zero-shot cross-lingual transfer from English fails on other languages. How do you fix it?"

---

## Solutions

1. **Target-language few-shot** — Add 3-5 examples in the target language. Even a few native examples dramatically improve cross-lingual performance.
2. **Translate training data** — Machine-translate your English few-shot examples/training data to target languages. Quality isn't perfect but better than zero-shot.
3. **Multilingual model** — Switch to a model with stronger multilingual training (mT5, XLM-RoBERTa, or multilingual LLMs like Aya, Qwen).
4. **Language-specific fine-tuning** — Fine-tune on target-language data, even a small amount (100-500 examples).
5. **Pivot language** — If direct transfer fails, try an intermediate language closer to the target (e.g., Portuguese via Spanish).
6. **Bilingual prompting** — Include the prompt in both English and the target language.

## Scenario Examples
### A: An English NER model gets 90% F1 on English but 55% on Thai. Adding 5 Thai examples improves to 72%. Fine-tuning on 200 Thai-annotated examples reaches 85%.
### B: A sentiment classifier trained on English reviews works well on French (85%) but poorly on Japanese (60%). The gap is due to Japanese-specific negation patterns that don't exist in English. Adding Japanese-specific examples covering double-negation patterns improves to 79%.

## Follow-Up Questions
### Q1: "Why does cross-lingual transfer work at all?"
**Answer:** Multilingual models learn shared representations across languages during pre-training. "Dog" and "犬" (Japanese for dog) map to similar vectors. This shared space enables transferring task knowledge from one language to another. The quality depends on how much overlap exists in the model's training data.

### Q2: "What languages are hardest for cross-lingual transfer?"
**Answer:** Languages with: (1) different scripts (Thai, Arabic, Korean), (2) very different syntax (Japanese SOV vs. English SVO), (3) low representation in training data (Swahili, Tagalog). The more structurally and typologically different from English, the harder the transfer.

### Q3: "How much target-language data do you need?"
**Answer:** Often surprisingly little. 50-100 labeled examples in the target language can close 50-70% of the cross-lingual gap. 500 examples typically reach 80-90% of monolingual performance. This is far less than training from scratch in the target language.

---

<p align="center"><a href="../README.md">← Back to all questions</a></p>
