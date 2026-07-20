# Scenario 1: Inconsistent Few-Shot Results

> **The Scenario:** "Your few-shot prompting gives inconsistent results across similar inputs. How do you stabilize it?"

---

## Solutions

1. **Lower temperature** — Set temperature=0 for deterministic output on identical inputs
2. **Dynamic few-shot selection** — Retrieve the most similar examples to each input using embedding similarity, instead of using the same fixed examples for all inputs
3. **Increase example count** — Add more diverse examples covering edge cases (5 → 8-10)
4. **Normalize example format** — Ensure all examples follow exactly the same structure and length
5. **Self-consistency** — Generate N outputs, take majority vote for critical tasks
6. **Order sensitivity** — Test different example orderings; some orderings are more stable than others

## Scenario Examples
### A: A classification prompt gives different labels for "The product is okay" depending on surrounding examples. Fix: dynamic selection retrieves examples most similar to "okay/mediocre" inputs, consistently classifying as "Neutral."
### B: A data extraction prompt handles company names inconsistently ("Google LLC" vs "Google" vs "Alphabet"). Fix: add explicit normalization examples and a post-processing step that standardizes entity formats.

## Follow-Up Questions
### Q1: "How do you measure inconsistency?"
**Answer:** Run the same inputs 5-10 times with temperature=0.7. Calculate the agreement rate. Below 80% agreement means the prompt is unstable and needs redesign.

### Q2: "Does the number of examples matter for consistency?"
**Answer:** Yes — up to a point. More examples reduce variance by providing more reference patterns. But beyond 8-10 examples, consistency gains diminish while cost increases. The quality and diversity of examples matters more than quantity.

### Q3: "What role does the model play?"
**Answer:** Different models have different sensitivity to prompt variations. GPT-4 is generally more robust than GPT-3.5. Test your prompt on multiple models — if it's fragile on one model, it may break when models are updated.

---

<p align="center"><a href="../README.md">← Back to all questions</a></p>
