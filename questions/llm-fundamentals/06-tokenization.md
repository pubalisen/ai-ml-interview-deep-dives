# Part 7: Tokenization in LLMs

> **The Question:** "What is tokenization in LLMs?"

---

## What They're Actually Testing

The interviewer wants to know if you understand that LLMs don't see text — they see **integer sequences**. They want you to explain the mapping from raw characters to token IDs, why subword tokenization is used instead of word-level or character-level, and how tokenizer design impacts model behavior.

---

## The Technical Breakdown

### What Is Tokenization?

Tokenization is the process of converting raw text into a sequence of **tokens** (subword units) that the model can process. Each token is mapped to an integer ID that indexes into the model's **vocabulary**.

```
Input:  "Hello, world! Let's tokenize this."
Tokens: ["Hello", ",", " world", "!", " Let", "'s", " token", "ize", " this", "."]
IDs:    [9906, 11, 1917, 0, 6914, 594, 4037, 553, 419, 13]
```

### Why Not Word-Level or Character-Level?

| Approach | Problem |
|----------|---------|
| **Word-level** | Vocabulary explodes (millions of words). Can't handle misspellings, new words, code, or non-English text. Out-of-vocabulary (OOV) words are unsolvable. |
| **Character-level** | Sequences become extremely long (a 500-word document → 2500+ characters). The model must learn spelling from scratch. Context windows fill up fast. |
| **Subword (BPE)** | ✅ Balances vocabulary size (~32K–100K) with sequence length. Common words are single tokens; rare words decompose into known sub-pieces. No OOV problem. |

### How BPE (Byte Pair Encoding) Works

BPE builds a vocabulary by iteratively merging the most frequent adjacent character pairs:

```
Step 0: Start with individual characters: ['l', 'o', 'w', 'e', 'r', ...]
Step 1: Most frequent pair: ('l', 'o') → merge into 'lo'
Step 2: Most frequent pair: ('lo', 'w') → merge into 'low'
Step 3: Most frequent pair: ('e', 'r') → merge into 'er'
Step 4: Most frequent pair: ('low', 'er') → merge into 'lower'
... repeat until vocabulary reaches target size
```

The result is a vocabulary where:
- Common words like "the", "and" → 1 token each
- Regular words like "running" → 1-2 tokens
- Rare/technical words like "pneumonoultramicroscopicsilicovolcanoconiosis" → many tokens
- Code like `def __init__` → 3-4 tokens

### Token ≠ Word: Why This Matters

```python
# GPT-4 (cl100k_base tokenizer) examples:
"Hello"     → [9906]                    # 1 token
"indivisibility" → [485, 344, 3714, 488] # 4 tokens  
" a"        → [264]                      # 1 token (space included!)
"日本語"     → [11017, 102, 36707]        # 3 tokens
"\n"        → [198]                      # 1 token
```

Key implications:
1. **Cost** — You're billed per token, not per word. A 100-word prompt might be 130-150 tokens.
2. **Context window** — The context limit is in tokens, not words. 128K tokens ≈ ~96K words.
3. **Model behavior** — The model reasons at the **token level**. If "strawberry" is tokenized as ["straw", "berry"], the model doesn't inherently "see" it as one word.

---

## Accuracy Check

| Claim | Verified? |
|-------|-----------|
| Modern LLMs use subword tokenization (BPE or variants) | ✅ GPT uses BPE; LLaMA uses SentencePiece BPE |
| BPE iteratively merges most frequent character pairs | ✅ Sennrich et al., "Neural Machine Translation of Rare Words with Subword Units" (2016) |
| Common words become single tokens; rare words decompose | ✅ Observable via tiktoken, sentencepiece libraries |
| Token count ≠ word count | ✅ OpenAI tokenizer tool confirms |
| Leading whitespace is often included in the token | ✅ GPT tokenizers encode preceding space as part of the token |

---

## Scenario Examples

### Scenario 1: Why GPT Can't Count Letters in "Strawberry"

A user asks, "How many r's are in strawberry?" and the model answers incorrectly (says 2 instead of 3). The issue is tokenization: "strawberry" might be tokenized as ["str", "aw", "berry"] — the model never sees the individual characters. It's performing "character counting" on **subword chunks**, not on actual characters. This is a fundamental limitation of BPE tokenization — the model's basic unit of processing is the token, not the character.

### Scenario 2: Multilingual Cost Explosion

A company deploys their LLM chatbot in Japan and discovers that Japanese messages cost 3-4× more in API fees than equivalent English messages. The reason: the tokenizer was trained primarily on English text, so English words are efficiently encoded (1 word ≈ 1-1.3 tokens), while Japanese characters are split into more tokens (1 character ≈ 1-3 tokens). A 50-character Japanese sentence might consume 80-120 tokens. Solutions: use a model with a multilingual tokenizer (like LLaMA 3, which expanded its vocabulary to 128K tokens with better multilingual coverage), or use a model specifically trained for Japanese.

---

## Follow-Up Questions

### Q1: "What happens if you change the tokenizer for an existing model?"

**Answer:** You **cannot** swap tokenizers on a trained model. The model's embedding layer has a fixed vocabulary size — each row in the embedding matrix corresponds to a specific token ID. If you change the tokenizer, the mapping between text and token IDs changes, which means the embeddings are now associated with wrong tokens. The model would produce garbage output. Changing the tokenizer requires **retraining from scratch** (or at minimum, re-initializing the embedding layer and doing extensive continued pre-training). This is why tokenizer design is one of the earliest and most consequential decisions in LLM development.

### Q2: "How does vocabulary size affect model performance?"

**Answer:** Vocabulary size is a trade-off:
- **Larger vocabulary** (100K-256K tokens): Fewer tokens per sentence (more efficient context usage), better coverage of rare words and non-English languages, but larger embedding matrix (more parameters, more memory).
- **Smaller vocabulary** (32K tokens): More tokens per sentence (less efficient), may split common words unnecessarily, but smaller embedding matrix and faster softmax computation.

LLaMA 1 used 32K vocabulary; LLaMA 3 expanded to 128K specifically to improve multilingual performance and code handling. The Chinchilla-optimal vocabulary size depends on the training data composition.

### Q3: "Why does the tokenizer include whitespace in tokens?"

**Answer:** Including the preceding whitespace (e.g., " the" rather than "the") allows the tokenizer to distinguish between a word at the start of a sentence and a word in the middle. Without this, "The" and " the" would be the same token, losing information about sentence boundaries. It also means the model can reconstruct the original text exactly by concatenating tokens — no ambiguity about where spaces go. This is a design choice in GPT-style BPE tokenizers (using the `Ġ` prefix internally). SentencePiece uses a similar approach with the `▁` prefix character.

---

## Further Reading

- [Neural Machine Translation of Rare Words with Subword Units — BPE (Sennrich et al., 2016)](https://arxiv.org/abs/1508.07909)
- [SentencePiece: A simple and language independent subword tokenizer (Kudo & Richardson, 2018)](https://arxiv.org/abs/1808.06226)
- [OpenAI Tokenizer Tool](https://platform.openai.com/tokenizer)

---

<p align="center">
  <a href="../../README.md">← Back to all questions</a>
</p>
