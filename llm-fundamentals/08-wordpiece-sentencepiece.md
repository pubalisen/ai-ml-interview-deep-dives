# Part 9: WordPiece and SentencePiece

> **The Question:** "Explain WordPiece and SentencePiece."

---

## What They're Actually Testing

The interviewer wants you to compare tokenization algorithms beyond BPE. They're checking if you know the **mechanical differences** between WordPiece, SentencePiece, and BPE — and can explain when and why each is used.

---

## The Technical Breakdown

### WordPiece

WordPiece was developed at Google for speech recognition and later adopted for **BERT**. It's similar to BPE but uses a different merge criterion.

**How it differs from BPE:**

| Aspect | BPE | WordPiece |
|--------|-----|-----------|
| **Merge criterion** | Most **frequent** pair | Pair that maximizes **likelihood** of training data |
| **Selection formula** | count(ab) | score(ab) = count(ab) / (count(a) × count(b)) |
| **Result** | Favors raw frequency | Favors pairs whose co-occurrence is surprising relative to their individual frequencies |
| **Subword prefix** | No special marking | `##` prefix for continuation tokens |

**WordPiece tokenization example:**

```
Input: "unbelievable"
Tokens: ["un", "##believ", "##able"]

Input: "playing"
Tokens: ["playing"]  ← common enough to be a single token

Input: "tokenization"
Tokens: ["token", "##ization"]
```

The `##` prefix marks tokens that are **continuations** (not word-starting). This lets the model distinguish "un" at the start of a word from "un" as a continuation.

**Why the likelihood criterion matters:** BPE might merge ("t", "h") early because "th" is frequent. WordPiece considers: is "th" appearing together more often than you'd expect from the individual frequencies of "t" and "h"? This produces merges that are more linguistically meaningful.

### SentencePiece

SentencePiece is a **language-independent** tokenization framework developed by Google. It's not a single algorithm — it supports both BPE and Unigram methods. Its key innovation is treating the input as a **raw byte stream** with no pre-tokenization.

**How it differs from BPE/WordPiece:**

| Aspect | BPE / WordPiece | SentencePiece |
|--------|----------------|---------------|
| **Pre-tokenization** | Requires word segmentation first (split on spaces/punctuation) | Treats input as raw character stream — no word boundaries assumed |
| **Whitespace** | Handled separately | Whitespace is treated as a regular character (replaced with `▁`) |
| **Language support** | Biased toward space-delimited languages | Works equally well for Chinese, Japanese, Thai (no spaces) |
| **Algorithms** | BPE (for BPE), WordPiece (for WordPiece) | BPE mode or Unigram mode |
| **Reversibility** | May lose whitespace info | Perfectly reversible — `▁` encodes spaces explicitly |

**SentencePiece tokenization example:**

```
Input: "I love Tokyo"
Tokens: ["▁I", "▁love", "▁To", "ky", "o"]

Input: "東京は美しい" (Japanese, no spaces)
Tokens: ["▁東京", "は", "美", "しい"]
```

The `▁` (U+2581) marks the beginning of a new word. This means SentencePiece can reconstruct the original text perfectly, including whitespace placement.

### The Unigram Algorithm (SentencePiece's Other Mode)

Unlike BPE (which builds bottom-up by merging), the Unigram algorithm works **top-down**:

1. Start with a **large** candidate vocabulary (all possible substrings up to a length limit)
2. Assign each token a probability based on its frequency in the corpus
3. For each input, find the tokenization that **maximizes the overall probability** (using Viterbi algorithm)
4. Remove tokens that contribute least to the overall corpus probability
5. Repeat until vocabulary reaches target size

**Key advantage:** Unigram can output **multiple valid tokenizations** for the same input with different probabilities, enabling **subword regularization** — a training technique where the tokenization is randomly sampled during training to improve model robustness.

### Which Models Use Which?

| Model | Tokenizer |
|-------|-----------|
| BERT, DistilBERT | WordPiece |
| GPT-2, GPT-3, GPT-4 | Byte-level BPE |
| LLaMA 1/2/3, Mistral | SentencePiece (BPE mode) |
| T5, mT5 | SentencePiece (Unigram mode) |
| Gemma, Gemini | SentencePiece |

---

## Accuracy Check

| Claim | Verified? |
|-------|-----------|
| WordPiece uses likelihood-based merge criterion | ✅ Schuster & Nakajima (2012); BERT paper (Devlin et al., 2019) |
| WordPiece uses `##` prefix for continuations | ✅ BERT tokenizer implementation |
| SentencePiece treats input as raw byte stream | ✅ Kudo & Richardson (2018) |
| SentencePiece uses `▁` to encode whitespace | ✅ SentencePiece documentation |
| Unigram works top-down by pruning | ✅ Kudo (2018), "Subword Regularization" |
| LLaMA uses SentencePiece BPE | ✅ LLaMA model card, Touvron et al. (2023) |

---

## Scenario Examples

### Scenario 1: Building a Multilingual Application

A team is building a multilingual chatbot serving English, Mandarin, Japanese, and Thai. With a BPE tokenizer trained primarily on English (like GPT-2's tokenizer), Thai text (which has no spaces between words) gets split into individual UTF-8 bytes, producing 5-6 tokens per character — making it 10× more expensive than English. By switching to a SentencePiece-based model (like LLaMA 3 with 128K vocabulary), the tokenizer handles all languages natively because it doesn't assume space-delimited word boundaries. Thai token efficiency improved from 6 tokens/character to 1.5 tokens/character.

### Scenario 2: Subword Regularization for Robustness

A machine translation team notices their model is brittle — small typos in source text cause large translation errors. They switch from BPE (deterministic tokenization) to SentencePiece's **Unigram mode with subword regularization**. During training, each sentence is tokenized differently each time (sampling from multiple valid tokenizations). For example, "unbelievable" might be tokenized as ["un", "believable"] one time and ["unbe", "liev", "able"] another. This forces the model to learn from multiple subword decompositions, making it more robust to variations at inference time.

---

## Follow-Up Questions

### Q1: "Why does BERT use WordPiece while GPT uses BPE?"

**Answer:** It's largely a historical/team decision rather than a fundamental technical requirement. BERT was developed at Google, which had already built WordPiece for their speech systems. GPT was developed at OpenAI, which chose BPE based on the Sennrich et al. (2016) machine translation work. In practice, the **vocabulary size, training data composition, and byte-level handling** matter more for downstream performance than the specific merge algorithm (frequency-based BPE vs. likelihood-based WordPiece). Both produce high-quality subword vocabularies.

### Q2: "What is subword regularization and why does it help?"

**Answer:** Subword regularization (Kudo, 2018) is a training technique where the tokenization of each sentence is **randomly sampled** from multiple valid tokenizations rather than using the single "best" tokenization. For example, "New York" could be tokenized as ["New", "York"] or ["N", "ew", "Y", "ork"]. By exposing the model to different segmentations, it learns to be robust to tokenization variations, misspellings, and morphological changes. This is only possible with the Unigram algorithm (which assigns probabilities to all possible tokenizations) and is a key advantage over BPE (which is deterministic).

### Q3: "How would you decide between BPE and Unigram for a new model?"

**Answer:** Choose **BPE** when:
- You're building a decoder-only generative model (GPT-style)
- You want deterministic, reproducible tokenization
- Your primary language is English or a space-delimited language

Choose **Unigram** when:
- You're building a multilingual model or serving many languages
- You want subword regularization for training robustness
- Your domain includes unsegmented languages (Chinese, Japanese, Thai)
- You're building an encoder model where classification robustness matters

In practice, **SentencePiece with BPE mode** (used by LLaMA) gives you the best of both worlds: BPE's simplicity with SentencePiece's language independence.

---

## Further Reading

- [SentencePiece: A Simple and Language Independent Subword Tokenizer (Kudo & Richardson, 2018)](https://arxiv.org/abs/1808.06226)
- [Subword Regularization: Improving Neural Network Translation Models with Multiple Subword Candidates (Kudo, 2018)](https://arxiv.org/abs/1804.10959)
- [Japanese and Korean Voice Search — WordPiece (Schuster & Nakajima, 2012)](https://static.googleusercontent.com/media/research.google.com/ja//pubs/archive/37842.pdf)

---

<p align="center">
  <a href="../README.md">← Back to all questions</a>
</p>
