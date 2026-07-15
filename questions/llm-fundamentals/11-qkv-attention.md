# Part 12: Query, Key, and Value in Attention

> **The Question:** "Explain the Query (Q), Key (K), and Value (V) in attention."

---

## What They're Actually Testing

The interviewer wants you to explain Q, K, V as a **retrieval mechanism**, not just "three matrices." They want to hear the analogy to information retrieval and the mechanical role each plays in computing attention weights.

---

## The Technical Breakdown

### The Intuition: Database Lookup

Think of attention as a **soft database query**:

| Concept | Database Analogy | Attention Role |
|---------|-----------------|----------------|
| **Query (Q)** | Your search query | "What am I looking for?" — the current token's request for information |
| **Key (K)** | Index/tags on each record | "What do I contain?" — each token's identifier that gets matched against queries |
| **Value (V)** | The actual record content | "Here's my information" — the data that gets retrieved when a key matches a query |

### The Mechanics

For each token, three vectors are computed via **learned linear projections**:

```
Q = X × W_Q    # [seq_len × d_k]
K = X × W_K    # [seq_len × d_k]
V = X × W_V    # [seq_len × d_v]
```

The attention computation:

```
Step 1: Compute scores     → S = Q × Kᵀ         # How relevant is each key to each query?
Step 2: Scale              → S = S / √d_k        # Prevent large values from saturating softmax
Step 3: (Optional) Mask    → S = S + mask         # Causal mask for decoder-only models
Step 4: Normalize          → A = softmax(S)       # Convert to probability weights (sum to 1)
Step 5: Aggregate          → Output = A × V       # Weighted sum of values
```

### Concrete Example

```
Sentence: "The cat sat on the mat"
Processing token: "sat" (position 2)

Q_sat = "I'm a verb — I need my subject"
K_the = "I'm an article"     → low match with Q_sat
K_cat = "I'm a noun/subject" → HIGH match with Q_sat  
K_sat = "I'm a verb"         → moderate self-match
K_on  = "I'm a preposition"  → low match

Attention weights for "sat": [0.05, 0.70, 0.15, 0.05, 0.03, 0.02]
                               the   cat   sat   on   the   mat

Output_sat = 0.05 × V_the + 0.70 × V_cat + 0.15 × V_sat + ...
           = mostly V_cat's information → "sat" now carries context about its subject
```

### Why Separate Q, K, V?

If you used the same vector for all three roles, every token would always attend most strongly to **itself** (highest dot product with itself). Separate projections let the model learn **asymmetric relationships**:
- A verb's **query** can search for subjects
- A noun's **key** can advertise that it's a subject
- A noun's **value** can carry the actual semantic content

---

## Accuracy Check

| Claim | Verified? |
|-------|-----------|
| Q, K, V are computed via separate learned linear projections | ✅ Vaswani et al. (2017), Section 3.2 |
| Attention scores are Q × Kᵀ / √d_k | ✅ Vaswani et al. (2017), Equation 1 |
| Softmax normalizes scores to probability distribution | ✅ Standard softmax definition |
| Separate projections enable asymmetric relationships | ✅ Demonstrated in attention probing studies (Clark et al., 2019) |

---

## Scenario Examples

### Scenario 1: Coreference Resolution

In "Sarah said she would go to the store," the model needs to understand that "she" refers to "Sarah." When processing "she," its **query** vector encodes "I'm a pronoun looking for my referent." "Sarah's" **key** vector encodes "I'm a named entity, feminine." The high dot-product score between these Q and K vectors produces a strong attention weight, so "she's" output representation is heavily weighted by Sarah's **value** vector — effectively resolving the coreference.

### Scenario 2: Long-Range Dependencies in Code

In a Python function, a `return` statement at line 50 needs to reference the function signature at line 1. The `return` token's **query** says "I need to know what this function returns." The function name's **key** says "I define the function contract." Because attention computes pairwise scores across **all** positions, this connection is made in a single layer — unlike RNNs, which would need to propagate this through 50 sequential steps.

---

## Follow-Up Questions

### Q1: "Why is the scaling factor √d_k needed?"

**Answer:** Without scaling, the dot products Q·K grow proportionally to the dimension d_k. For large d_k (e.g., 128), the dot products become very large, pushing the softmax into regions where its gradients are extremely small (near-zero). This is called **softmax saturation**. The model effectively becomes a hard argmax — attending to only one token — which kills gradient flow during training. Dividing by √d_k keeps the variance of the dot products at approximately 1, keeping softmax in a region with healthy gradients.

### Q2: "Can Q and K have different dimensions than V?"

**Answer:** Yes — d_k (Q/K dimension) and d_v (V dimension) can differ. Q and K **must** have the same dimension because they're dot-producted together. V can have any dimension — it determines the output dimension of the attention layer. In practice, most implementations set d_k = d_v = d_model / num_heads for simplicity, but some architectures (like GQA) decouple them.

### Q3: "What happens if you share weights between Q, K, and V projections?"

**Answer:** Sharing W_Q = W_K = W_V collapses the model into **symmetric attention** — each token's score with another is the same in both directions, and the output is just a weighted average of the input with itself getting the highest weight. The model loses its ability to learn directional relationships (e.g., "verb looks for subject" is different from "subject looks for verb"). Experiments show that weight sharing significantly degrades performance on all language tasks. The separate projections are what give attention its expressive power.

---

<p align="center">
  <a href="../../README.md">← Back to all questions</a>
</p>
