# Scenario 7: Domain Term Tokenization Issues

> **The Scenario:** "Your tokenizer splits important domain terms into meaningless subword pieces. How do you fix it?"

---

## Solutions
1. **Add domain tokens to the vocabulary** — Extend the tokenizer with domain-specific tokens (e.g., "COVID-19" as a single token instead of ["CO", "VID", "-", "19"]). Requires resizing the embedding matrix and continued pre-training.
2. **Use a character-level or byte-level tokenizer** — BPE tokenizers handle unseen words better than WordPiece. SentencePiece is more robust for multilingual/technical domains.
3. **Prompt engineering** — Define domain terms in the system prompt: "When I say 'CRISPR-Cas9', I mean the gene editing technology."
4. **Fine-tune on domain text** — While the tokenizer doesn't change, the model learns to associate subword sequences with domain meanings through exposure.
5. **Train a custom tokenizer** — For extreme domain specialization, train a BPE tokenizer on domain corpus, then continue pre-training the model with the new tokenizer.

## Scenario Examples
### A: A biotech model splits "BRCA1" into ["BR", "CA", "1"]. Adding "BRCA1" and "BRCA2" as single tokens and continuing pre-training for 1% of original compute solves the issue.
### B: A legal model handles Latin terms poorly ("habeas corpus" → ["hab", "eas", "cor", "pus"]). Fine-tuning on legal text teaches the model that this subword sequence means one concept, even without tokenizer changes.

## Follow-Up Questions
### Q1: "What's the risk of adding too many custom tokens?"
**Answer:** Each new token needs its own embedding vector (random initialization) and sufficient training examples. Adding 1000 tokens without enough training data for each leaves their embeddings poorly trained — the model outputs garbage when using those tokens.

### Q2: "How does tokenizer quality affect downstream performance?"
**Answer:** Poor tokenization wastes context window (more tokens per concept), reduces the model's ability to learn concept boundaries, and increases inference cost. A term split into 5 subwords costs 5× the attention computation of a single-token term.

### Q3: "Can you change the tokenizer without retraining?"
**Answer:** You can extend the tokenizer by adding new tokens, but you MUST do continued pre-training to learn the new token embeddings. Simply adding tokens without training assigns random embeddings that produce nonsensical outputs.

---

<p align="center"><a href="../README.md">← Back to all questions</a></p>
