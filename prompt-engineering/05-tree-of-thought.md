# Part 5: Tree-of-Thought Prompting

> **The Question:** "What is tree-of-thought prompting?"

---

## The Technical Breakdown

### What Is Tree-of-Thought (ToT)?

Tree-of-thought extends chain-of-thought by exploring **multiple reasoning branches** at each step, evaluating them, and selecting the most promising paths — like a search tree:

```
                      Problem
                    /    |    \
               Path A  Path B  Path C     ← Step 1: Generate candidates
                 ↓       ↓       ↓
              Eval: 8  Eval: 3  Eval: 7   ← Step 2: Evaluate/score each
                 ↓               ↓
            Path A1  A2      Path C1  C2   ← Step 3: Expand best paths
               ↓       ↓       ↓
            Eval: 9  Eval: 5  Eval: 6     ← Step 4: Evaluate again
               ↓
            Final Answer                   ← Best path wins
```

### Key Components

1. **Thought generation** — At each step, generate K candidate "thoughts" (partial solutions)
2. **State evaluation** — Use the LLM to score how promising each thought is
3. **Search algorithm** — BFS (breadth-first) or DFS (depth-first) to explore the tree
4. **Backtracking** — Abandon dead-end paths and try alternatives

### ToT vs. CoT vs. Self-Consistency

| Method | Exploration | Backtracking | Cost |
|--------|------------|-------------|------|
| **CoT** | Single linear chain | ❌ | 1× |
| **Self-consistency** | N independent chains | ❌ | N× |
| **ToT** | Branching tree with evaluation | ✅ | 10-50× |

---

## Accuracy Check
| Claim | Verified? |
|-------|-----------|
| ToT uses tree search over reasoning steps | ✅ Yao et al. (2023), "Tree of Thoughts" |
| ToT outperforms CoT on planning/puzzle tasks | ✅ Demonstrated on Game of 24, crosswords, creative writing |
| ToT is significantly more expensive | ✅ Multiple LLM calls per step for generation + evaluation |

## Scenario Examples
### A: **Game of 24:** Given 4 numbers, make 24 using arithmetic. CoT achieves 7.3% success. ToT achieves 74% — because it can try different operation orderings and backtrack from dead ends.
### B: **Code architecture design:** ToT generates 3 architectural approaches, evaluates each for scalability/complexity, explores the best 2 in detail, and selects the optimal design. The backtracking capability is crucial when an approach hits an unforeseen constraint.

## Follow-Up Questions
### Q1: "When is ToT overkill?"
**Answer:** For tasks with a single correct reasoning path (simple math, factual Q&A), CoT is sufficient. ToT excels when the problem has **multiple possible approaches** and it's unclear upfront which one leads to the solution (puzzles, planning, design).

### Q2: "How does the LLM evaluate its own thoughts?"
**Answer:** A separate LLM call prompts: "Evaluate if this partial solution is promising. Score 1-10 with reasoning." The model acts as both generator and critic. This self-evaluation is imperfect but sufficient for pruning clearly bad paths.

### Q3: "Can you implement ToT without multiple LLM calls?"
**Answer:** A simplified "ToT-in-one-prompt" version asks the model to consider multiple approaches, evaluate them, and select the best — all in a single generation. This is less rigorous but much cheaper. True ToT with separate generation and evaluation calls is more powerful but 10-50× more expensive.

---

<p align="center"><a href="../README.md">← Back to all questions</a></p>
