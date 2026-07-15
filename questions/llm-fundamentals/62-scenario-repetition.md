# Scenario 17: Text Repetition in Long Outputs

> **The Scenario:** "Your text generation repeats phrases in long outputs. How do you fix repetition?"

---

## Solutions
1. **Repetition penalty** — Reduce the logit of tokens that have already appeared. Most APIs expose this as a parameter (1.0 = no penalty, 1.2 = moderate).
2. **Frequency penalty** — Penalize proportional to how many times a token has appeared (linear scaling).
3. **Presence penalty** — Penalize any token that has appeared at all (binary — appeared or not).
4. **Top-p sampling** — Use nucleus sampling instead of greedy decoding. The randomness prevents getting stuck in repetitive loops.
5. **N-gram blocking** — Prevent any n-gram (e.g., 4-gram) from appearing more than once. Used in beam search.
6. **Post-processing** — Detect and remove repetitive paragraphs/sentences from the output.

## Scenario Examples
### A: A story generator writes "the dark forest" 15 times in one story. Setting frequency_penalty=0.5 gradually discourages reuse, leading to varied descriptions: "the shadowed woods," "the dense canopy."
### B: A code generator repeats the same comment block every 20 lines. 4-gram blocking prevents any comment from being exactly repeated, forcing variation.

## Follow-Up Questions
### Q1: "Why do LLMs repeat?"
**Answer:** Autoregressive models predict the next token based on context. If the same context pattern recurs, the model predicts the same continuation — a feedback loop. Greedy decoding (T=0) is most susceptible because it always picks the highest-probability token. Sampling breaks this loop.

### Q2: "How do you set the repetition penalty?"
**Answer:** Start at 1.1-1.2 for general text. Increase to 1.3-1.5 for long-form generation. Too high (>2.0) makes text incoherent — the model avoids common words it has already used. Test with your specific content type.

### Q3: "Is repetition always bad?"
**Answer:** No — legal documents, code, and structured text have legitimate repetition. For these domains, reduce or disable repetition penalties. The key is distinguishing functional repetition from degenerate repetition.

---

<p align="center"><a href="../../README.md">← Back to all questions</a></p>
