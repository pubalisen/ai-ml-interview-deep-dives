# Part 20: Edge Cases and Adversarial Inputs

> **The Question:** "How do you handle edge cases and adversarial inputs in prompt design?"

---

## The Technical Breakdown

### Types of Edge Cases

| Category | Example | Impact |
|----------|---------|--------|
| **Empty/minimal input** | "" or "hi" | Model generates gibberish or hallucinated context |
| **Very long input** | 100K token document | Context overflow, truncation issues |
| **Ambiguous input** | "Bank" (financial? river?) | Wrong interpretation |
| **Adversarial input** | Prompt injection attempts | Security breach |
| **Out-of-domain** | Math question to a cooking bot | Hallucinated answer in wrong domain |
| **Multi-language** | Mixed languages in one query | Inconsistent responses |
| **Special characters** | SQL, HTML, code in inputs | Template injection, formatting breaks |

### Defense Strategies

1. **Input validation** — Check length, language, character set before prompting
2. **Graceful degradation** — Fallback responses for unhandled cases
3. **Explicit boundary setting** — "If the input is outside your expertise, say so"
4. **Input sanitization** — Escape special characters, strip HTML/scripts
5. **Guardrail models** — Use a classifier to detect off-topic/malicious inputs before the main model
6. **Adversarial testing** — Systematically test with known attack patterns

---

## Scenario Examples
### A: A form extraction model receives a blank PDF. Without edge case handling, it hallucinates field values. Fix: check document content length before processing; return `{"status": "empty_document"}` for blank inputs.
### B: A chatbot receives inputs in 15 different languages. Edge case prompt: "If the user writes in a language other than English, respond in their language. If you cannot reliably respond in that language, say: 'I'm most accurate in English. Would you like to continue in English?'"

## Follow-Up Questions
### Q1: "How do you build a comprehensive test suite for edge cases?"
**Answer:** Start with a taxonomy: empty, minimal, maximal, malformed, adversarial, out-of-domain, multi-language, special characters. Generate 5-10 examples per category. Add real production failures as they're discovered. This "edge case corpus" grows over time and catches regressions.

### Q2: "How do you handle cascading failures in prompt chains?"
**Answer:** Each step should validate its input (is it the expected format?) and output (does it match the expected schema?). If validation fails, return a structured error rather than propagating garbage data. Circuit breakers prevent infinite retry loops.

### Q3: "What is fuzzing for LLM prompts?"
**Answer:** Automated generation of random, malformed, or adversarial inputs to find prompt failures. Tools like Garak and ART (Adversarial Robustness Toolkit) generate diverse attack inputs and measure model robustness. This is the LLM equivalent of software fuzz testing.

---

<p align="center"><a href="../README.md">← Back to all questions</a></p>
