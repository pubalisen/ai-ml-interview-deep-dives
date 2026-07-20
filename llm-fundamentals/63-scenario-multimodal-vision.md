# Scenario 18: Can Transformers Understand Images?

> **The Scenario:** "Transformers work on text, so can they also understand images?"

---

## The Answer: Yes — Vision Transformers and Multimodal Models

### How Images Become Tokens

Transformers process **sequences of tokens**. Images are converted to sequences by splitting them into **patches**:

```
Image (224×224 pixels) → Split into 16×16 patches → 196 patches
Each patch (16×16×3 = 768 values) → Linear projection → 768-dimensional embedding
Result: 196 "visual tokens" — same shape as text tokens
```

The Transformer processes these visual tokens with the same self-attention mechanism used for text.

### Key Architectures

| Model | Approach | Use Case |
|-------|----------|----------|
| **ViT** (Vision Transformer) | Image patches as tokens, encoder-only | Image classification |
| **CLIP** | Separate image + text encoders, contrastive learning | Image-text matching, zero-shot classification |
| **GPT-4V / Gemini** | Image patches interleaved with text tokens | Multimodal chat, visual Q&A |
| **Stable Diffusion** | Text encoder + diffusion decoder | Image generation from text |

### Multimodal LLMs

Modern multimodal models handle both text and images in a single model:

```
Input: [text tokens] + [image patch embeddings] + [text tokens]
       "What is in"  + [🖼️ image patches]       + "this photo?"

The model attends across both text and image tokens using standard attention.
```

## Scenario Examples
### A: GPT-4o receives an image of a graph and text "Summarize the trend." The vision encoder converts the graph into patch embeddings. The language model attends to both the text query and image patches, generating: "Revenue increased 23% YoY with a seasonal dip in Q3."

### B: A manufacturing QA system receives photos of parts. A fine-tuned ViT classifies defects. The same architecture that processes text can identify microscopic cracks in metal parts.

## Follow-Up Questions
### Q1: "How do multimodal models align image and text representations?"
**Answer:** Contrastive learning (CLIP) trains image and text encoders to produce similar embeddings for matching pairs. BLIP-2 uses a Q-Former that learns to extract text-relevant features from image representations. LLaVA projects image features into the text embedding space using a linear projection layer.

### Q2: "What are the limitations of vision in LLMs?"
**Answer:** Spatial reasoning (precise locations, counting objects), OCR accuracy on small text, understanding complex charts/diagrams, and hallucinating visual details that aren't in the image. Vision capabilities are improving rapidly but still lag behind text understanding.

### Q3: "Can Transformers handle video?"
**Answer:** Yes — by extending the patch approach to 3D: (spatial patches × temporal frames). Each video frame is split into patches, and patches across frames form a spatiotemporal sequence. Models like VideoMAE and Gemini 1.5 Pro can process video sequences, though computational cost scales with frame count.

---

<p align="center"><a href="../README.md">← Back to all questions</a></p>
