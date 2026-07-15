# Scenario 16: Summarization Hallucination

> **The Scenario:** "Your summarization system hallucinated facts not in the original article. How do you fix it?"

---

## Solutions
1. **Extractive + abstractive hybrid** — First extract key sentences, then abstractively rephrase them. This grounds the summary in actual source text.
2. **Factual consistency checking** — Use NLI models (e.g., TRUE, SummaC) to verify each summary sentence is entailed by the source.
3. **Citation-grounded generation** — Require the model to cite specific paragraphs for each claim. Verify citations exist and support the claim.
4. **Low temperature** — Use temperature=0 for factual summarization. Lower randomness = less hallucination.
5. **Faithfulness fine-tuning** — Fine-tune on datasets with faithfulness annotations, penalizing summaries that add unsupported information.

## Scenario Examples
### A: A news summarizer states "the CEO resigned" when the article only says "the CEO is considering his options." NLI checking catches this — "resigned" is not entailed by "considering options."
### B: A medical summary adds "patients should take 500mg" when the article discusses dosing in general terms. Citation checking reveals no source passage supports this specific dosage.

## Follow-Up Questions
### Q1: "What's the difference between intrinsic and extrinsic hallucination?"
**Answer:** Intrinsic: contradicts the source (says A when source says B). Extrinsic: adds information not in the source (neither supported nor contradicted). Both are problematic for summarization, but extrinsic is harder to detect because the information might be true — just not in the source.

### Q2: "How do you measure hallucination rate?"
**Answer:** Use automated metrics: SummaC, FactCC, or DAE scores. Combine with human evaluation on a sample. Typical approach: randomly sample 100 summaries, have humans annotate each sentence as supported/unsupported/contradicted.

### Q3: "Does chain-of-thought help with faithful summarization?"
**Answer:** Somewhat — prompting "First, identify the key facts in the source. Then, write a summary using only those facts" improves faithfulness. But the fundamental issue is the model's tendency to fill in plausible details, which CoT doesn't fully eliminate.

---

<p align="center"><a href="../../README.md">← Back to all questions</a></p>
