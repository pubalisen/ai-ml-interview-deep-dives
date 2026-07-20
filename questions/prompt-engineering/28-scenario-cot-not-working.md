# Scenario 5: CoT Not Improving Accuracy

> **The Scenario:** "Your chain-of-thought prompting is not improving LLM accuracy on reasoning tasks. What do you fix?"

---

## Diagnosis & Solutions

1. **Model too small** — CoT is an emergent capability (>10B parameters). If using a 3B model, CoT generates plausible-but-wrong reasoning. Fix: upgrade to a larger model.
2. **Wrong task type** — CoT doesn't help with simple factual recall or classification. Remove CoT for non-reasoning tasks.
3. **Poor CoT examples** — Few-shot CoT examples with incorrect reasoning teach the model to reason incorrectly. Fix: verify every step in your examples.
4. **Not enough reasoning steps** — "Think step by step" may produce too-brief chains. Fix: specify "Show all intermediate calculations" or provide detailed reasoning examples.
5. **Self-consistency needed** — A single CoT chain may make errors. Generate 5-10 chains and majority vote.
6. **Reasoning is correct, answer extraction is wrong** — The model reasons correctly but formats the final answer incorrectly. Fix: add explicit "Therefore, the final answer is:" extraction.

## Scenario Examples
### A: CoT on a 3B model generates: "Let's think step by step. The answer is 42." — no actual reasoning, just the magic phrase followed by a guess. Upgrading to 13B produces genuine multi-step reasoning.
### B: CoT on a math problem: the model shows 5 correct steps but then writes the wrong final number in the "Answer:" field. Fix: add "Double-check your final answer against your work" to the prompt.

## Follow-Up Questions
### Q1: "How do you verify CoT reasoning quality?"
**Answer:** Use process reward models or manual inspection to check each reasoning step. Correct final answers can come from wrong reasoning (lucky errors). Only verifying final answers misses reasoning quality issues.

### Q2: "What about structured CoT?"
**Answer:** Instead of free-form "think step by step," structure the reasoning: "Step 1: Identify the variables. Step 2: Write the equation. Step 3: Solve. Step 4: Verify." This structured approach produces more reliable reasoning chains.

### Q3: "When should you give up on CoT and try something else?"
**Answer:** If CoT doesn't improve accuracy after: (1) upgrading model size, (2) providing high-quality examples, (3) structuring the reasoning format — the task may require domain-specific tools (calculators, code execution) rather than pure LLM reasoning. For math: use code interpreter. For logic: use a formal solver.

---

<p align="center"><a href="../../README.md">← Back to all questions</a></p>
