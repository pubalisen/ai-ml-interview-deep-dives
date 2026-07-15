# Part 8: BPE (Byte Pair Encoding)

> **The Question:** "Explain BPE (Byte Pair Encoding)."

---

## What They're Actually Testing

The interviewer wants you to walk through the BPE **algorithm step by step** — not just say "it merges frequent pairs." They want to see if you understand the training process, the merge rules, and why this specific approach produces good subword vocabularies for LLMs.

---

## The Technical Breakdown

### What Is BPE?

BPE is a **compression algorithm** adapted for NLP tokenization. It builds a vocabulary by starting with individual characters (or bytes) and iteratively merging the most frequent adjacent pair until the vocabulary reaches a target size.

### The Algorithm (Step by Step)

**Training Phase (happens once, on the training corpus):**

```
Corpus: "low low low lower lower newest newest widest"

Step 0: Initialize vocabulary with individual characters
Vocab: {l, o, w, e, r, n, s, t, i, d, _}
Split: l·o·w  l·o·w  l·o·w  l·o·w·e·r  l·o·w·e·r  n·e·w·e·s·t  n·e·w·e·s·t  w·i·d·e·s·t

Step 1: Count all adjacent pairs
        (l, o) = 5, (o, w) = 5, (w, e) = 4, (e, r) = 2, (e, s) = 3, (s, t) = 3, ...
        Most frequent: (l, o) with count 5
        → Merge: 'l' + 'o' → 'lo'
Vocab: {l, o, w, e, r, n, s, t, i, d, _, lo}
Split: lo·w  lo·w  lo·w  lo·w·e·r  lo·w·e·r  n·e·w·e·s·t  n·e·w·e·s·t  w·i·d·e·s·t

Step 2: Recount pairs
        (lo, w) = 5, (w, e) = 4, (e, s) = 3, (s, t) = 3, ...
        Most frequent: (lo, w) with count 5
        → Merge: 'lo' + 'w' → 'low'
Vocab: {l, o, w, e, r, n, s, t, i, d, _, lo, low}
Split: low  low  low  low·e·r  low·e·r  n·e·w·e·s·t  n·e·w·e·s·t  w·i·d·e·s·t

Step 3: (e, s) = 3, (s, t) = 3, (low, e) = 2, ...
        → Merge: 'e' + 's' → 'es'

Step 4: (es, t) = 3
        → Merge: 'es' + 't' → 'est'

Step 5: (low, e) = 2
        → Merge: 'low' + 'e' → 'lowe'

... and so on until vocabulary reaches target size (e.g., 50,000)
```

**Encoding Phase (happens at inference time):**

Given the learned merge rules (in order), apply them greedily to any new text:

```
Input: "lowest"
Start: l·o·w·e·s·t
Apply rule 1 (l+o→lo): lo·w·e·s·t
Apply rule 2 (lo+w→low): low·e·s·t
Apply rule 3 (e+s→es): low·es·t
Apply rule 4 (es+t→est): low·est
Result: ["low", "est"] → [token_id_1, token_id_2]
```

### BPE vs. Byte-Level BPE

| Variant | Base units | OOV handling | Used by |
|---------|-----------|--------------|---------|
| **Original BPE** | Unicode characters | Can have OOV for rare chars | Early NMT models |
| **Byte-level BPE** | Raw bytes (256 base tokens) | Zero OOV — any byte sequence is representable | GPT-2, GPT-3, GPT-4 |

Byte-level BPE starts with 256 byte values instead of Unicode characters, ensuring that **any input** (including emojis, code, binary data) can be tokenized — no OOV tokens ever.

### Key Properties of BPE Vocabularies

1. **Frequency-driven**: Common words ("the", "and") become single tokens
2. **Composable**: Rare words decompose into known sub-pieces
3. **Deterministic**: The same text always produces the same tokens (given the same merge rules)
4. **Lossless**: Concatenating tokens reconstructs the original text exactly

---

## Accuracy Check

| Claim | Verified? |
|-------|-----------|
| BPE iteratively merges the most frequent adjacent pair | ✅ Sennrich et al. (2016) |
| Byte-level BPE starts with 256 byte values | ✅ Radford et al., GPT-2 (2019) |
| Byte-level BPE guarantees zero OOV | ✅ Any byte sequence is representable |
| GPT-2/3/4 use byte-level BPE | ✅ OpenAI documentation, tiktoken source code |
| BPE tokenization is deterministic | ✅ Same merge rules → same output |

---

## Scenario Examples

### Scenario 1: Building a Domain-Specific Tokenizer

A biotech company fine-tunes LLaMA on genomic sequences (strings of A, C, G, T). With the default tokenizer, a sequence like "ATCGATCG" is tokenized into 8 individual character tokens because these letter combinations weren't frequent in the general pre-training corpus. This wastes 8× the context window capacity. By training a **custom BPE tokenizer** on their genomic corpus, common motifs like "ATCG" or "GATC" become single tokens. This allows the model to process 8× longer sequences within the same context window, dramatically improving performance on downstream tasks.

### Scenario 2: Debugging Inconsistent Outputs

A developer notices that their LLM answers "What is the capital of Japan?" correctly but fails on "What is the capital of japan?" (lowercase). Investigation reveals that the tokenizer encodes "Japan" as one token (ID 7526) but "japan" as two tokens ["j", "apan"] (IDs 73, 21111). Because these map to completely different embedding vectors, the model processes them as fundamentally different inputs. The fix: normalize input text (capitalize proper nouns) before sending to the model, or fine-tune with case variations.

---

## Follow-Up Questions

### Q1: "How does BPE handle completely unseen words?"

**Answer:** This is BPE's core strength — it **never** encounters a truly unseen word. Since the vocabulary always includes individual characters (or bytes in byte-level BPE), any new word will simply be decomposed into its constituent characters/subwords. For example, if the model has never seen "covfefe", it will be tokenized as ["cov", "fe", "fe"] or similar subword splits. The model's understanding of this word comes from combining its knowledge of the sub-pieces. It won't produce perfect results, but it won't crash or output an `<UNK>` token — which was a fatal problem with word-level tokenizers.

### Q2: "What determines the optimal vocabulary size?"

**Answer:** Vocabulary size is a trade-off between:
- **Compression efficiency**: Larger vocab → fewer tokens per text → more efficient context window usage
- **Embedding parameters**: Larger vocab → larger embedding matrix (vocab_size × d_model) → more memory
- **Softmax cost**: Larger vocab → more expensive final layer computation (logits over entire vocabulary)
- **Training data coverage**: Need enough training data for each token to learn good embeddings

In practice: GPT-2 used 50,257 tokens, GPT-4 uses ~100K (cl100k_base), LLaMA 3 uses 128K. The trend is toward larger vocabularies as models scale up, because the relative overhead of a larger embedding matrix decreases while the benefits of better compression increase.

### Q3: "What is the difference between BPE and WordPiece?"

**Answer:** Both are subword tokenization algorithms, but they differ in how they select merges:
- **BPE** selects the most **frequent** adjacent pair in the corpus
- **WordPiece** selects the pair that maximizes the **likelihood** of the training data (a probabilistic criterion)

In practice, BPE greedily counts raw frequency, while WordPiece considers how much a merge improves the language model's probability assignment. WordPiece tends to produce slightly different vocabularies that favor linguistically meaningful units. BPE is used by GPT models; WordPiece is used by BERT. For modern LLMs, the distinction is less important than the vocabulary size and training data composition.

---

## Further Reading

- [Neural Machine Translation of Rare Words with Subword Units (Sennrich et al., 2016)](https://arxiv.org/abs/1508.07909)
- [Language Models are Unsupervised Multitask Learners — GPT-2 (Radford et al., 2019)](https://cdn.openai.com/better-language-models/language_models_are_unsupervised_multitask_learners.pdf)
- [tiktoken — OpenAI's BPE tokenizer library](https://github.com/openai/tiktoken)

---

<p align="center">
  <a href="../../README.md">← Back to all questions</a>
</p>
