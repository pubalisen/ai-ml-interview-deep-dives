# Part 8: Structured Output Prompting

> **The Question:** "How do you structure prompts for consistent structured output (JSON, XML)?"

---

## The Technical Breakdown

### Techniques (Ranked by Reliability)

**1. Structured Output API (best)**
```python
response = client.chat.completions.create(
    model="gpt-4o",
    response_format={"type": "json_schema", "json_schema": schema}
)
```

**2. Constrained decoding** — Outlines, XGrammar enforce grammar at token level. 100% valid.

**3. Explicit schema in prompt**
```
Respond with a JSON object matching this exact schema:
{"name": string, "age": number, "skills": string[]}

Do not include any text outside the JSON block.
```

**4. XML tags for structure** (works well with Claude)
```
<output>
  <answer>The capital is Paris</answer>
  <confidence>0.95</confidence>
  <sources>Wikipedia, CIA World Factbook</sources>
</output>
```

**5. Markdown code blocks** — Wrap output in ```json``` fences for parsing.

### Key Principles
- Show the **exact** output format in the prompt (not a description of it)
- Use few-shot examples of correctly formatted output
- Place format instructions at the **end** of the prompt (recency bias)
- Validate and retry on parse failure

---

## Scenario Examples
### A: An API that returns product recommendations uses Pydantic + constrained decoding. Every response is valid JSON matching the schema — zero parsing errors across 1M+ requests.
### B: A data extraction pipeline uses XML tags for multi-field extraction from documents. XML is easier for the model to produce correctly than JSON for nested structures (no comma/bracket matching issues).

## Follow-Up Questions
### Q1: "JSON vs XML — which is better for LLM output?"
**Answer:** JSON is standard for APIs. XML is more robust for LLM generation (fewer syntax errors — no trailing comma issues). For production APIs, use JSON with structured output APIs. For intermediate processing where you parse the output yourself, XML tags are surprisingly reliable.

### Q2: "What about YAML?"
**Answer:** YAML's whitespace-sensitivity makes it error-prone for LLM output. Models frequently generate invalid YAML (wrong indentation). Avoid YAML for LLM structured output. Stick to JSON (with schema enforcement) or XML.

### Q3: "How do you handle partial/streaming structured output?"
**Answer:** Stream tokens and parse incrementally. Libraries like `partial-json-parser` handle incomplete JSON during streaming. Alternatively, generate the full response before parsing. For real-time UX, show a loading state until the complete JSON is available.

---

<p align="center"><a href="../../README.md">← Back to all questions</a></p>
