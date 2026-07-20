# Scenario 2: Prompt Sensitivity in Classification

> **The Scenario:** "Your LLM classification system is too sensitive to prompt wording changes. How do you reduce prompt sensitivity?"

---

## Solutions

1. **Ensemble multiple prompt variants** — Run 3-5 differently worded prompts, aggregate results (majority vote)
2. **Prompt optimization (DSPy/OPRO)** — Algorithmically find robust phrasings that perform well across input variations
3. **Fine-tuning** — A fine-tuned model is inherently less prompt-sensitive because the behavior is in the weights, not the prompt
4. **Structured output enforcement** — Constrain output to valid labels only, reducing the impact of prompt wording on format
5. **Calibration** — Measure per-class bias for each prompt variant and apply correction factors

## Scenario Examples
### A: Changing "Classify the sentiment" to "What is the sentiment" drops accuracy from 88% to 76%. Fix: use DSPy to optimize and find: "Read the following review. Output only one word: Positive, Negative, or Neutral" — achieves 91% and is robust to minor wording changes.
### B: A multi-class classifier with 15 labels is extremely prompt-sensitive. The team fine-tunes a 7B model on 2000 labeled examples. Post fine-tuning, the model achieves 93% accuracy with a simple "Classify:" prefix — virtually zero prompt sensitivity.

## Follow-Up Questions
### Q1: "Why are some prompts more sensitive than others?"
**Answer:** Ambiguous instructions activate different "modes" in the model depending on exact wording. "Classify" vs "Label" vs "Categorize" activates slightly different pre-training patterns. Specificity reduces sensitivity — "Output exactly one of: [list]" is more robust than "classify the following."

### Q2: "How do you test for prompt sensitivity?"
**Answer:** Create 5-10 paraphrases of your prompt (same meaning, different wording). Run all on the eval dataset. If accuracy variance is >5%, the prompt is too sensitive. The most robust prompt is the one with the least variance across paraphrases.

### Q3: "When is fine-tuning worth it over prompt engineering?"
**Answer:** When: (1) prompt sensitivity is costing you accuracy and you can't stabilize it, (2) you have 500+ labeled examples, (3) the task is narrow and well-defined (classification, extraction), (4) you need consistent production behavior. Fine-tuning eliminates prompt sensitivity entirely.

---

<p align="center"><a href="../../README.md">← Back to all questions</a></p>
