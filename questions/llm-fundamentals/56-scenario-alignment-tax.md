# Scenario 11: The Alignment Tax

> **The Scenario:** "After RLHF alignment, your LLM became safer but lost capability on hard tasks. How do you manage the alignment tax?"

---

## Solutions
1. **Targeted alignment** — Only apply RLHF to safety-critical behaviors (refusals, harmful content). Keep general capability training separate.
2. **DPO with capability preservation data** — Mix preference data (safety) with capability data (reasoning, coding) during DPO training.
3. **Constitutional AI** — Use AI-generated critiques instead of human RLHF, which can be more precisely targeted and less likely to over-refuse.
4. **Capability-specific evaluation** — Monitor capability benchmarks during alignment training. Stop if math/code scores drop more than X%.
5. **Post-alignment continued training** — After RLHF, do a brief continued pre-training phase on capability-rich data to recover lost skills.

## Scenario Examples
### A: After RLHF, the model refuses to write code that handles file deletion (safety over-generalization). Adding nuanced examples ("file deletion in a file manager app is okay") to the preference data fixes the over-refusal.
### B: A team tracks HumanEval (code) and GSM8K (math) during RLHF. When code scores drop 5%, they add 20% coding data to the next RLHF batch, recovering the capability.

## Follow-Up Questions
### Q1: "Is alignment tax inevitable?"
**Answer:** Some tax is inevitable — the model's capacity is finite, and dedicating some to safety behaviors means less for other capabilities. But with careful training (mixed objectives, targeted alignment), the tax can be minimized to 1-3% on benchmarks.

### Q2: "How do you detect alignment over-refusal?"
**Answer:** Track refusal rates on benign queries. If >5% of legitimate requests are refused, the model is over-aligned. Use red-team datasets of borderline-but-safe queries to calibrate the refusal boundary.

### Q3: "What is Constitutional AI?"
**Answer:** Anthropic's approach where AI generates its own critiques based on a "constitution" (set of principles). The model critiques its own outputs, then revises them. This reduces reliance on human annotators and can be more precisely targeted than RLHF.

---

<p align="center"><a href="../../README.md">← Back to all questions</a></p>
