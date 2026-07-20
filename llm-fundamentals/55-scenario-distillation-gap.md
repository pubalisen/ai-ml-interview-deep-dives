# Scenario 10: Distilled Model Reasoning Gap

> **The Scenario:** "Your distilled student model fails on the complex reasoning that the teacher model handled. How do you close the gap?"

---

## Solutions
1. **Chain-of-thought distillation** — Don't just distill the final answer. Distill the teacher's reasoning steps too. The student learns *how* to reason, not just *what* to answer.
2. **Progressive distillation** — Start with a medium-sized intermediate model (30B), distill to that first, then distill again to the target size (7B).
3. **Task-specific fine-tuning** — After distillation, fine-tune on hard examples where the student fails. Focus the student's capacity on the most challenging cases.
4. **Mixture of experts at student scale** — Use a small MoE model that has more total capacity than a dense model of the same inference cost.
5. **Ensemble of small models** — Deploy multiple distilled models and aggregate their outputs (majority voting for classification, best-of-N for generation).

## Scenario Examples
### A: A 7B student fails multi-step math. Distilling GPT-4's step-by-step solutions (not just final answers) improves the student's math accuracy from 45% to 72%.
### B: A team distills into a 3B model for mobile deployment. It handles 90% of queries well but fails on complex ones. They implement a routing system: simple queries → 3B on-device, complex queries → large model via API.

## Follow-Up Questions
### Q1: "What reasoning capabilities are hardest to distill?"
**Answer:** Multi-step mathematical reasoning, abstract analogy, and counterfactual reasoning are hardest because they require compositional generalization that small models struggle with due to limited capacity.

### Q2: "How do you identify which capabilities were lost?"
**Answer:** Run the student and teacher on a benchmark suite, comparing per-task performance. The tasks with the largest gaps indicate which capabilities were lost. Targeted fine-tuning on those tasks can partially recover them.

### Q3: "Is there a minimum model size for reasoning?"
**Answer:** Empirically, chain-of-thought reasoning emerges around 7B-13B parameters. Below 3B, multi-step reasoning is unreliable even with CoT distillation. For tasks requiring reasoning, 7B is a practical minimum.

---

<p align="center"><a href="../README.md">← Back to all questions</a></p>
