# Part 14: Cross Attention

> **The Question:** "What is Cross Attention in Transformers?"

---

## What They're Actually Testing

The interviewer wants you to contrast cross-attention with self-attention mechanically and explain where it's used (encoder-decoder models, multimodal models). They want to see if you know that decoder-only LLMs don't use cross-attention.

---

## The Technical Breakdown

### What Is Cross Attention?

Cross-attention is an attention mechanism where **Q comes from one sequence** and **K, V come from a different sequence**. It allows one part of the model to attend to representations from another part.

```
Self-Attention:   Q, K, V ← same sequence
Cross-Attention:  Q ← decoder,  K, V ← encoder output
```

### Where It's Used

```
Encoder-Decoder (T5, BART, original Transformer for translation):

Encoder processes: "Le chat est sur le tapis"
                   → produces encoder hidden states [H₁, H₂, ..., H₆]

Decoder generates: "The cat is on the mat"
                   → at each step, Q comes from decoder
                   → K, V come from encoder hidden states

Cross-attention lets the decoder "look at" the source sentence
while generating the translation.
```

### The Computation

```python
# Decoder state (current generation step)
Q = decoder_hidden @ W_Q    # Query from decoder

# Encoder output (fixed after encoding)
K = encoder_output @ W_K    # Keys from encoder
V = encoder_output @ W_V    # Values from encoder

# Same attention formula
scores = Q @ K.T / sqrt(d_k)
attn_weights = softmax(scores)
output = attn_weights @ V    # Decoder now has encoder context
```

### Self-Attention vs. Cross-Attention

| Property | Self-Attention | Cross-Attention |
|----------|---------------|-----------------|
| Q source | Same sequence | Decoder (target) |
| K, V source | Same sequence | Encoder (source) |
| Use case | Contextualizing tokens within a sequence | Connecting two different sequences |
| Models | All Transformers | Encoder-decoder only (T5, BART, Whisper) |
| In decoder-only LLMs? | ✅ (with causal mask) | ❌ Not used |

### Why Decoder-Only LLMs Don't Need It

GPT, LLaMA, and Claude are **decoder-only** — there's no encoder. The input and output are concatenated into a single sequence:

```
[system prompt + user message + assistant response]
```

Self-attention with a causal mask handles everything. The model attends to the prompt tokens (using self-attention) the same way cross-attention would attend to encoder outputs. This simplification is one reason decoder-only architectures dominate — fewer components, simpler training.

---

## Accuracy Check

| Claim | Verified? |
|-------|-----------|
| Cross-attention uses Q from decoder, K/V from encoder | ✅ Vaswani et al. (2017), Section 3.2.3 |
| Encoder-decoder models (T5, BART) use cross-attention | ✅ T5 (Raffel et al., 2020), BART (Lewis et al., 2020) |
| Decoder-only LLMs don't use cross-attention | ✅ GPT, LLaMA architecture specifications |
| Cross-attention connects two different sequences | ✅ Used in translation, summarization, multimodal models |

---

## Scenario Examples

### Scenario 1: Machine Translation

In translating "Je suis étudiant" → "I am a student," the encoder processes the French sentence and produces hidden states. When the decoder generates "student," its cross-attention Q vector asks "What's the content word I should translate?" and attends most strongly to "étudiant" in the encoder output. Without cross-attention, the decoder would have no access to the source sentence's representations.

### Scenario 2: Vision-Language Models

In multimodal models like Flamingo, cross-attention connects text and image modalities. The image is processed by a vision encoder into a sequence of patch embeddings. The language model's decoder uses cross-attention where Q comes from text tokens and K, V come from image patch embeddings. This lets the text generation "look at" specific parts of the image — when generating "a red car," the model's cross-attention focuses on the image patches containing the car.

---

## Follow-Up Questions

### Q1: "If decoder-only models don't use cross-attention, how do they handle the input prompt?"

**Answer:** The input prompt is simply concatenated with the generated output into one sequence. Self-attention with a causal mask lets generated tokens attend to all prompt tokens (they're earlier in the sequence). The prompt tokens serve the same functional role as encoder outputs — they provide context that generation conditions on. The key difference is that in encoder-decoder models, the encoder and decoder have separate parameter sets, while in decoder-only models, the same parameters process both input and output.

### Q2: "Where does cross-attention appear in the decoder block of an encoder-decoder model?"

**Answer:** A decoder block in an encoder-decoder model has **three sub-layers** (not two like decoder-only):
1. **Causal self-attention** — decoder tokens attend to previous decoder tokens
2. **Cross-attention** — decoder tokens attend to encoder output
3. **Feed-forward network** — per-position transformation

The order matters: self-attention first (contextualize generated tokens), then cross-attention (incorporate source information), then FFN (non-linear transformation).

### Q3: "Is cross-attention used in any modern architectures beyond encoder-decoder?"

**Answer:** Yes — cross-attention is experiencing a revival in **multimodal models**. Flamingo uses gated cross-attention to connect vision and language. DALL-E and Stable Diffusion use cross-attention to condition image generation on text prompts. Whisper (speech-to-text) uses cross-attention between audio encoder and text decoder. The pattern "one modality queries another modality's representations" is fundamentally cross-attention, and it's the standard approach for connecting different modalities or different information sources.

---

<p align="center">
  <a href="../README.md">← Back to all questions</a>
</p>
