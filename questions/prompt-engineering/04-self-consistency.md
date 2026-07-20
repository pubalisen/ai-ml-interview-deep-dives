# Part 4: Self-Consistency Prompting

> **The Question:** "Explain self-consistency prompting and how it improves reasoning."

---

## The Technical Breakdown

### How It Works

1. Generate **N reasoning chains** for the same problem using temperature > 0
2. Extract the final answer from each chain
3. Take the **majority vote** across all answers

```
Problem: "If a shirt costs $25 and is 20% off, what's the final price?"

Chain 1: "20% of 25 = 5, so 25 - 5 = $20" → $20
Chain 2: "Discount = 0.2 × 25 = 5. Price = 25 - 5 = $20" → $20
Chain 3: "20% off means 80% remaining. 0.8 × 25 = $20" → $20
Chain 4: "20% of 25 = 4. 25 - 4 = $21" → $21 (arithmetic error)
Chain 5: "25 × 0.8 = $20" → $20

Majority vote: $20 (4/5 chains agree) ✅
```

### Why It Works

Different reasoning paths may make **different errors** in intermediate steps, but the correct answer is more likely to be reached by multiple independent paths. Errors are randomly distributed; correctness converges.

### Configuration

| Parameter | Typical Value | Trade-off |
|-----------|--------------|-----------|
| N (number of chains) | 5-20 | More N = higher accuracy but more cost |
| Temperature | 0.5-0.8 | Higher = more diverse chains |
| Voting | Majority | Weighted by log-prob also works |

---

## Accuracy Check
| Claim | Verified? |
|-------|-----------|
| Self-consistency improves CoT accuracy by 5-15% | ✅ Wang et al. (2023) |
| Majority voting filters out random errors | ✅ Statistical property of independent trials |
| Effective N is typically 5-20 | ✅ Diminishing returns beyond ~20 |

## Scenario Examples
### A: A math tutoring system generates 10 solutions per problem. 8/10 agree on "$20" — high confidence. On a harder problem, chains split 4-3-3 across three answers — low confidence, flagged for human review.
### B: A medical symptom checker generates 5 diagnostic chains. If 4/5 agree, auto-respond. If results are split, escalate to a physician. Self-consistency provides a **natural confidence signal**.

## Follow-Up Questions
### Q1: "How is this different from beam search?"
**Answer:** Beam search explores the top-k most probable **token sequences** — it's greedy at the sequence level. Self-consistency generates **complete, independent** reasoning chains with sampling randomness, then votes on answers. Beam search finds the most likely output; self-consistency finds the most **robust** answer.

### Q2: "When is self-consistency NOT worth the cost?"
**Answer:** When (1) the task is simple enough that CoT alone achieves >95%, (2) latency is critical (N chains = N× latency without parallelism), or (3) there's no clear "right answer" to vote on (creative writing, open-ended chat).

### Q3: "Can you use self-consistency without CoT?"
**Answer:** Yes, but it's less effective. Without reasoning chains, the answer diversity is lower (all chains take similar shortcuts). CoT creates diverse reasoning paths that make the voting meaningful. Self-consistency + CoT is the intended combination.

---

<p align="center"><a href="../../README.md">← Back to all questions</a></p>
