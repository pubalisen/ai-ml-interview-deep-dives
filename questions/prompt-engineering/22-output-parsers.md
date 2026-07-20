# Part 22: Output Parsers

> **The Question:** "What are output parsers, and why are they needed for production applications?"

---

## The Technical Breakdown

### What Are Output Parsers?

Output parsers are components that **transform raw LLM text output into structured data** that downstream systems can consume:

```python
# Raw LLM output (text):
"The sentiment is positive with a confidence of 0.87"

# After output parser:
{"sentiment": "positive", "confidence": 0.87}
```

### Why They're Needed

LLMs generate **text** — but applications need **data**. Output parsers bridge this gap:

| Without Parser | With Parser |
|---------------|-------------|
| Raw text string | Typed data structure |
| Fragile regex extraction | Schema-validated objects |
| Breaks on format changes | Handles variations |
| Manual error handling | Automatic retry on parse failure |

### Types of Output Parsers

| Type | Mechanism | Reliability |
|------|----------|-------------|
| **Regex** | Pattern matching on output | Low — fragile |
| **JSON parser** | `json.loads()` on fenced JSON | Medium — breaks on invalid JSON |
| **Pydantic parser** | Parse + validate against schema | High — type-safe |
| **XML parser** | Parse XML tags from output | Medium-High |
| **Function calling** | Model returns structured tool calls natively | Highest — API-level guarantee |
| **Structured output API** | Schema-enforced generation | Highest — token-level guarantee |

### Parser + Retry Pattern

```python
from langchain.output_parsers import PydanticOutputParser
from pydantic import BaseModel

class Sentiment(BaseModel):
    label: str    # "positive" | "negative" | "neutral"
    confidence: float
    reasoning: str

parser = PydanticOutputParser(pydantic_object=Sentiment)

# If parsing fails, retry with error message appended
for attempt in range(3):
    response = llm(prompt + parser.get_format_instructions())
    try:
        return parser.parse(response)
    except OutputParserException as e:
        prompt += f"\nYour previous response could not be parsed: {e}"
```

---

## Scenario Examples
### A: An API returns LLM responses to frontend clients. Without a parser, the frontend receives unpredictable text. With a Pydantic parser + structured output API, every response is guaranteed to match the TypeScript interface. Zero parsing errors in production.
### B: A data pipeline extracts entities from documents. The output parser validates that every entity has a `name`, `type`, and `confidence` field, rejecting malformed responses and retrying automatically.

## Follow-Up Questions
### Q1: "When should you use structured output APIs vs. output parsers?"
**Answer:** Structured output APIs (OpenAI, Anthropic) are preferred — they enforce format at the token level, making parsers unnecessary. Use output parsers when: (1) using models without structured output support, (2) you need custom validation beyond JSON schema, or (3) parsing non-JSON formats.

### Q2: "How do you handle partial failures in parsing?"
**Answer:** Implement fallback parsing: try strict JSON → try lenient JSON (fix common errors like trailing commas) → try regex extraction → return error. Libraries like `json_repair` can fix common JSON malformations.

### Q3: "What is the cost of retry-based parsing?"
**Answer:** Each retry is an additional LLM call (~2× the cost and latency). With structured output APIs, retry rate drops to <0.1%. Without them, retry rates of 5-15% are common, adding 5-15% to your LLM costs. This alone often justifies switching to structured output.

---

<p align="center"><a href="../../README.md">← Back to all questions</a></p>
