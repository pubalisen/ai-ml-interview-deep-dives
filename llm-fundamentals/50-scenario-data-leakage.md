# Scenario 5: Preventing Training Data Memorization Leaks

> **The Scenario:** "Your LLM memorized proprietary training data and leaks it in responses. How do you prevent this?"

---

## Solutions

1. **Data deduplication** — Remove exact and near-duplicate content from training data. Memorization correlates strongly with data repetition.
2. **Differential privacy** — Add calibrated noise during training (DP-SGD) to mathematically bound memorization. Trade-off: quality degradation at strong privacy budgets.
3. **Output filtering** — Post-generation, check if output matches training data verbatim using membership inference tests or n-gram overlap detection.
4. **Canary detection** — Insert known "canary" strings during training. If the model can reproduce canaries, memorization is occurring.
5. **Temperature > 0** — Avoid temperature=0 (greedy decoding) which is most likely to reproduce memorized text.

## Accuracy Check
| Method | Effectiveness | Impact |
|--------|-------------|--------|
| Deduplication | High | Prevents root cause |
| Differential privacy | Provable guarantees | 5-15% quality loss |
| Output filtering | High for exact matches | Runtime overhead |
| Temperature > 0 | Moderate | Changes output distribution |

## Scenario Examples
### A: A model trained on customer emails reproduces full emails verbatim. Deduplication finds the same email appears 47 times in training data. Removing duplicates and retraining eliminates the leak.
### B: A healthcare LLM must comply with HIPAA. DP-SGD training with ε=8 provides a mathematical guarantee that no individual patient record can be extracted, with a ~10% quality trade-off.

## Follow-Up Questions
### Q1: "How do you detect if memorization is happening?"
**Answer:** (1) Prompt completion attacks — give the model the start of a training document and check if it completes it verbatim. (2) Membership inference — test if the model assigns higher probability to known training examples vs. unseen examples. (3) Extractable memorization tests — prompt the model with prefixes and measure exact-match rates.

### Q2: "Does model size affect memorization?"
**Answer:** Yes — larger models memorize more. Carlini et al. (2022) showed that memorization scales log-linearly with model size. A 1.5B model memorizes ~2× more than a 350M model. This makes memorization prevention more critical for frontier-scale models.

### Q3: "What about copyright and legal liability?"
**Answer:** If the model reproduces copyrighted training data, the deployer may face legal liability. The NYT vs. OpenAI case highlights this. Mitigations: output filtering against known copyrighted content, licensing training data, and implementing content moderation pipelines that detect potential reproduction.

---

<p align="center">
  <a href="../README.md">← Back to all questions</a>
</p>
