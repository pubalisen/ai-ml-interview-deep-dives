# Scenario 6: Outdated Code Generation

> **The Scenario:** "Your LLM coding assistant generates outdated code using deprecated libraries. How do you fix it?"

---

## Solutions
1. **RAG with current docs** — Index the latest library documentation in a vector database. Retrieve relevant API docs at query time. The model sees current APIs.
2. **System prompt with version constraints** — "Use React 18 hooks. Do not use class components or lifecycle methods."
3. **Fine-tuning on current code** — Train on recent code examples from updated repositories.
4. **Tool use** — Let the model query live documentation APIs or package registries (npm, PyPI) at runtime.
5. **Post-validation** — Run generated code through a linter configured with current style rules and deprecation warnings.

## Scenario Examples
### A: A Python assistant recommends `tensorflow.keras` patterns from TF 1.x. RAG retrieval from TF 2.x docs ensures it generates current `tf.keras` code with the latest API signatures.
### B: A React assistant generates class components. System prompt + few-shot examples of functional components with hooks trains it to use modern patterns.

## Follow-Up Questions
### Q1: "How often should you update the RAG documentation index?"
**Answer:** For fast-moving libraries (React, LangChain): weekly. For stable libraries (NumPy, standard library): monthly. Automate with CI/CD pipelines that scrape and re-index documentation on release.

### Q2: "Can you combine RAG with fine-tuning?"
**Answer:** Yes — fine-tune for coding style and patterns, RAG for current API details. This gives the model good coding habits (from fine-tuning) with up-to-date knowledge (from retrieval).

### Q3: "What about security vulnerabilities in generated code?"
**Answer:** Use SAST (Static Application Security Testing) tools to scan generated code for known vulnerabilities. Integrate tools like Semgrep or CodeQL into the generation pipeline as a validation step.

---

<p align="center"><a href="../README.md">← Back to all questions</a></p>
