# Scenario 1: Enforcing Structured Output

> **The Scenario:** "Your LLM keeps ignoring your instructions. How do you make it follow structured output formats?"

---

## The Diagnosis

The model is generating free-form text when you need structured JSON, XML, or specific formats. This is one of the most common production issues.

---

## Solutions (Ranked by Reliability)

### 1. Constrained Decoding (Most Reliable)

Use logit processors that set invalid token probabilities to zero at each generation step:

```python
# Using Outlines (open-source) or guidance
from outlines import models, generate

model = models.transformers("meta-llama/Llama-3-8B")
generator = generate.json(model, schema=MyPydanticModel)
result = generator("Extract the name and age from: John is 30 years old")
# Guaranteed valid JSON matching schema
```

**Why it works:** The model physically cannot generate tokens that violate the schema. 100% reliability.

### 2. Structured Output APIs

OpenAI, Anthropic, and Gemini offer native structured output:
```python
response = client.chat.completions.create(
    model="gpt-4o",
    response_format={"type": "json_schema", "json_schema": schema},
    messages=[...]
)
```

### 3. Better Prompting (Least Reliable, Easiest)

```
Respond ONLY with a valid JSON object matching this schema:
{"name": string, "age": number}

Do not include any text before or after the JSON.
```

Add few-shot examples of the exact format you expect.

### 4. Output Parsing + Retry

Generate freely, then validate. If invalid, retry with the error message:
```python
for attempt in range(3):
    output = model.generate(prompt)
    try:
        result = json.loads(output)
        validate(result, schema)
        return result
    except (JSONDecodeError, ValidationError) as e:
        prompt += f"\nPrevious output was invalid: {e}. Try again."
```

---

## Accuracy Check

| Solution | Reliability | Latency Impact | Complexity |
|----------|------------|----------------|-----------|
| Constrained decoding | 100% | Minimal | Medium |
| Structured output API | ~99% | None | Low |
| Better prompting | ~80-90% | None | Low |
| Parse + retry | ~95-99% | Higher (retries) | Medium |

---

## Scenario Examples

### Scenario A: An invoice extraction pipeline needs JSON with specific fields. Using constrained decoding with a Pydantic schema guarantees every output is valid, parseable JSON — zero parsing failures across 100K documents.

### Scenario B: A chatbot API endpoint validates responses against a JSON schema. With parse + retry, 94% of first attempts are valid. The 6% failure rate triggers retries, and 99.5% succeed within 3 attempts. The remaining 0.5% are logged for human review.

---

## Follow-Up Questions

### Q1: "Does constrained decoding degrade output quality?"

**Answer:** Minimally. The model's top-choice tokens usually already conform to the schema. Constrained decoding only intervenes when the model would otherwise generate an invalid token, redirecting it to the next most likely valid token. Quality loss is typically < 1%.

### Q2: "How do you handle nested or complex schemas?"

**Answer:** Use Pydantic models for schema definition. Constrained decoding libraries (Outlines, XGrammar) support arbitrarily complex schemas including nested objects, arrays, enums, and optional fields. The grammar is compiled from the schema before generation.

### Q3: "What about XML or custom formats?"

**Answer:** Constrained decoding supports any format expressible as a context-free grammar (CFG). JSON, XML, YAML, SQL, and even custom DSLs can be expressed as CFGs and enforced during generation.

---

<p align="center">
  <a href="../README.md">← Back to all questions</a>
</p>
