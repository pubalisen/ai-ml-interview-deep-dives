# Scenario 4: Controlling Response Verbosity

> **The Scenario:** "Your LLM generates responses that are too verbose. How do you control response length?"

---

## Solutions

1. **max_tokens parameter** — Hard limit on output length. Simple but may truncate mid-sentence.
2. **System prompt instructions** — "Respond in 2-3 sentences maximum." Works ~80% of the time.
3. **Few-shot examples** — Show the desired length through examples in the prompt.
4. **Fine-tuning on concise data** — Train on datasets with short, direct answers.
5. **Post-processing** — Generate freely, then use a second LLM call to summarize into target length.
6. **Length penalty in decoding** — Penalize the EOS token probability to encourage shorter sequences (or boost it).

## Accuracy Check
| Method | Reliability | Quality Impact |
|--------|------------|---------------|
| max_tokens | 100% (hard cutoff) | May truncate mid-thought |
| System prompt | ~80% | None |
| Few-shot | ~85% | None |
| Fine-tuning | ~95% | Best quality at target length |

## Scenario Examples
### A: An API returns product descriptions. max_tokens=100 ensures responses fit the UI widget, plus a post-processing step adds "..." if truncated mid-sentence.
### B: A meeting summarizer is fine-tuned on (transcript, 3-sentence-summary) pairs, consistently producing concise summaries without explicit length instructions.

## Follow-Up Questions
### Q1: "Why do LLMs tend to be verbose?"
**Answer:** RLHF training often rewards longer, more detailed answers (annotators prefer comprehensive responses). This creates a bias toward verbosity. Models like Claude explicitly counteract this with training on concise response patterns.

### Q2: "How do you handle cases where brevity sacrifices accuracy?"
**Answer:** Use conditional instructions: "Be concise, but prioritize accuracy. If a complete answer requires more detail, provide it." Or use a two-pass approach: generate a full response, then produce a TL;DR.

### Q3: "Can you dynamically adjust verbosity based on query complexity?"
**Answer:** Yes — classify query complexity first, then set different max_tokens/instructions. Simple factual queries get 1-2 sentences; complex analysis gets longer responses. A lightweight classifier or the LLM itself can assess complexity.

---

<p align="center">
  <a href="../../README.md">← Back to all questions</a>
</p>
