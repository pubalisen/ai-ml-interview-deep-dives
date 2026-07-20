# Part 43: Continual Learning in LLMs

> **The Question:** "What is Continual Learning in LLMs?"

---

## What They're Actually Testing

They want you to explain the challenge of updating LLM knowledge without catastrophic forgetting and current approaches.

---

## The Technical Breakdown

### The Problem

LLMs are trained on a fixed dataset with a knowledge cutoff. The world changes — new events, updated facts, deprecated APIs. Retraining from scratch is prohibitively expensive ($10M+ for frontier models). **Continual learning** aims to update model knowledge incrementally.

### Key Challenges

| Challenge | Description |
|-----------|-------------|
| **Catastrophic forgetting** | Training on new data causes the model to forget old knowledge |
| **Distribution shift** | New data may have different characteristics than pre-training data |
| **Knowledge conflict** | Old and new information may contradict each other |
| **Evaluation** | Hard to measure if new knowledge was learned without losing old |

### Current Approaches

| Method | How It Works | Trade-offs |
|--------|-------------|-----------|
| **Continued pre-training** | Resume pre-training on new data | Risk of forgetting; needs careful data mixing |
| **RAG** | Retrieve current information at inference time | No weight updates; latency overhead |
| **Knowledge editing** | Surgically modify specific FFN weights | Scales poorly to many edits |
| **LoRA adapters** | Train small adapter modules on new data | Can be composed; limited capacity |
| **Replay** | Mix new data with samples from old data | Requires storing/accessing old data |

---

## Accuracy Check

| Claim | Verified? |
|-------|-----------|
| Catastrophic forgetting is the core challenge | ✅ McCloskey & Cohen (1989); widely documented in LLMs |
| RAG is a practical alternative to weight updates | ✅ Industry standard for knowledge freshness |
| Knowledge editing targets FFN weights | ✅ ROME (Meng et al., 2022), MEMIT |

---

## Scenario Examples

### Scenario 1: A legal AI system needs to incorporate new legislation passed after the model's training cutoff. Rather than retraining, they use RAG — storing new legal texts in a vector database and retrieving relevant laws at query time. The model's weights stay frozen; knowledge freshness comes from the retrieval system.

### Scenario 2: A company fine-tunes their LLM on 2024 product documentation. After training, they discover the model has forgotten how to perform general reasoning tasks it could do before. They implement **data replay** — mixing 20% of general instruction data with the product-specific data during fine-tuning — which preserves general capabilities.

---

## Follow-Up Questions

### Q1: "Is RAG a form of continual learning?"

**Answer:** Strictly, no — RAG doesn't update model weights. It's an inference-time augmentation. But practically, it achieves the same goal (up-to-date knowledge) and is the most reliable approach. Many practitioners consider it the best "continual learning" solution precisely because it avoids catastrophic forgetting entirely.

### Q2: "How does data mixing prevent forgetting?"

**Answer:** By including old data alongside new data during continued training, the model's gradients balance between learning new patterns and reinforcing old ones. Typical mixing ratios: 80% general data + 20% new domain data. This prevents the model from completely overwriting old knowledge.

### Q3: "What are the limits of knowledge editing?"

**Answer:** Knowledge editing (ROME, MEMIT) works for individual fact updates (change the president's name) but degrades with scale — editing 1000+ facts simultaneously causes cascading errors. It also doesn't handle relational updates well (changing a country's capital should update many related facts).

---

<p align="center">
  <a href="../README.md">← Back to all questions</a>
</p>
