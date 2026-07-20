# Part 13: Prompt Templates

> **The Question:** "What is a prompt template, and how do you design one for production use?"

---

## The Technical Breakdown

### What Is a Prompt Template?

A prompt template is a **reusable, parameterized prompt structure** with placeholders that get filled at runtime:

```python
template = """You are a {role} specializing in {domain}.

Given this context:
{context}

Answer the following question in {format} format:
{question}

Requirements:
- Maximum {max_words} words
- Cite sources when possible
- If unsure, say "I don't know"
"""

# At runtime:
prompt = template.format(
    role="financial analyst",
    domain="cryptocurrency markets",
    context=rag_results,
    format="JSON",
    question=user_query,
    max_words=200
)
```

### Production Design Principles

| Principle | Practice |
|-----------|---------|
| **Separation of concerns** | Template structure vs. runtime data |
| **Version control** | Store templates in Git, not hardcoded |
| **A/B testing** | Deploy template variants, measure performance |
| **Validation** | Verify all placeholders are filled before sending |
| **Injection safety** | Sanitize user inputs before inserting into template |
| **Observability** | Log template version + parameters for debugging |

### Template Architecture

```
templates/
├── customer_support/
│   ├── v1_general.yaml
│   ├── v2_with_cot.yaml
│   └── v3_concise.yaml
├── code_review/
│   └── v1_review.yaml
└── summarization/
    ├── v1_extractive.yaml
    └── v2_abstractive.yaml
```

---

## Scenario Examples
### A: An enterprise deploys 15 different prompt templates for different workflows. Each template has version history, A/B test results, and associated eval scores. When template v3 causes a regression, they roll back to v2 in seconds.
### B: A multi-tenant SaaS uses the same base template with per-customer parameters: `{company_name}`, `{tone}`, `{products}`. One template serves 500 customers with personalized behavior.

## Follow-Up Questions
### Q1: "What format should templates be stored in?"
**Answer:** YAML or JSON for structured templates with metadata (version, description, parameters). Jinja2 templates for complex logic (conditionals, loops). Some teams use dedicated prompt management platforms (PromptLayer, Humanloop, LangSmith) for versioning, evaluation, and deployment.

### Q2: "How do you handle template evolution?"
**Answer:** Semantic versioning: v1.0 (initial), v1.1 (minor wording fix), v2.0 (structural change). Deploy new versions behind feature flags. Run parallel evaluation against the eval dataset. Only promote to production if the new version maintains or improves metrics.

### Q3: "Should templates include few-shot examples?"
**Answer:** Yes, but manage them separately. Store a pool of examples and dynamically select the most relevant ones at runtime (dynamic few-shot). This keeps templates reusable and adapts examples to each query's characteristics.

---

<p align="center"><a href="../README.md">← Back to all questions</a></p>
