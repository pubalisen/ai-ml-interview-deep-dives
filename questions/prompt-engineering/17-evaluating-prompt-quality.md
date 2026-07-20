# Part 17: Evaluating Prompt Quality

> **The Question:** "How do you evaluate and iterate on prompt quality?"

---

## The Technical Breakdown

### The Evaluation Loop

```
Design prompt → Run on eval set → Measure metrics → Identify failures → Iterate
```

### Building an Eval Dataset

1. **Collect 50-200 representative examples** with ground truth answers
2. Include **edge cases** (ambiguous, adversarial, empty inputs)
3. Cover the **full distribution** of expected inputs
4. Version the eval dataset alongside prompts

### Metrics by Task

| Task | Metric | Tool |
|------|--------|------|
| Classification | Accuracy, F1, Precision, Recall | Scikit-learn |
| Generation | ROUGE, BERTScore, BLEU | HuggingFace evaluate |
| Summarization | Faithfulness, relevance | RAGAS, DeepEval |
| Code | pass@k, functional correctness | Custom test harness |
| Open-ended | LLM-as-judge, human preference | GPT-4 scoring |

### LLM-as-Judge

Use a strong model to score outputs:

```python
judge_prompt = """Rate this response on a scale of 1-5 for:
1. Accuracy (factual correctness)
2. Completeness (covers all aspects)
3. Conciseness (no unnecessary info)

Response to evaluate: {response}
Ground truth: {ground_truth}

Provide scores and brief justification."""
```

### Iteration Framework

1. Run eval → identify lowest-scoring examples
2. Analyze failure patterns (formatting? reasoning? knowledge?)
3. Modify prompt to address the top failure pattern
4. Re-run eval → verify improvement without regression
5. Repeat

---

## Scenario Examples
### A: A team discovers their prompt achieves 92% accuracy overall but 45% on negation queries ("Which products are NOT eligible?"). They add few-shot examples with negation, improving negation accuracy to 83% without regressing overall accuracy.
### B: An automated eval pipeline runs nightly: every prompt version is scored against 200 test cases. Slack alerts fire if accuracy drops below 85%. This catches prompt regressions within 24 hours.

## Follow-Up Questions
### Q1: "How reliable is LLM-as-judge?"
**Answer:** GPT-4 as a judge correlates ~80-85% with human judgments (Zheng et al., 2023). It's biased toward verbose, well-structured responses. Mitigations: use rubrics, calibrate with human scores, and average across multiple judge calls with different orderings (position bias).

### Q2: "What is DSPy and how does it automate prompt iteration?"
**Answer:** DSPy (Declarative Self-improving Language Programs) treats prompts as optimizable programs. You define input/output signatures and a metric; DSPy automatically searches for optimal prompt wording, few-shot examples, and chain structures. It replaces manual prompt iteration with automated optimization.

### Q3: "How do you handle subjective tasks where there's no single correct answer?"
**Answer:** Use pairwise comparison: "Which response is better, A or B?" Human annotators or LLM judges choose the better response. This avoids absolute scoring (unreliable for subjective tasks) and produces relative rankings. Elo ratings can aggregate pairwise comparisons into a leaderboard.

---

<p align="center"><a href="../../README.md">← Back to all questions</a></p>
