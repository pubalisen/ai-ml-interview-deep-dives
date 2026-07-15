# Contributing to AI & ML Interview Deep Dives

Thanks for wanting to contribute! Here's how to add a new question or improve an existing one.

## Adding a New Question

### 1. Create a directory

```bash
mkdir -p questions/XX-your-topic-slug
```

Use the next available number and a kebab-case slug that describes the topic.

### 2. Write the breakdown

Create a `README.md` inside your directory using this structure:

```markdown
# Part XX: Your Topic Title

> **The Question:** "The actual interview question as you'd hear it."

---

## What They're Actually Testing
Explain the hidden evaluation criteria.

## The Technical Breakdown
The detailed, mechanical answer. Use code blocks, tables, and diagrams.

## Why This Matters in Production
Real-world implications and failure modes.

## The Curveball Follow-Up
A harder follow-up question and how to answer it.

## Key Takeaways
Numbered list of the most important points.

## Further Reading
Links to papers, docs, or resources.
```

### 3. Update the main README

Add your question to the table in the root `README.md`:

```markdown
| XX | **Your Topic** — Brief description | Category | [Read →](questions/XX-your-topic-slug/README.md) |
```

### 4. Open a PR

- Use a clear title: `Add Question XX: Your Topic`
- Include a brief description of what the question covers
- Make sure all links work

## Style Guide

- **Be mechanical, not theoretical.** Explain *how* things work, not just *what* they are.
- **Use code blocks** for anything that can be represented as code or pseudocode.
- **Use tables** for comparing options, parameters, or tradeoffs.
- **Cite papers** when referencing specific techniques or findings.
- **No fluff.** Every sentence should add information.

## Questions?

Open an issue or reach out on [LinkedIn](https://www.linkedin.com/in/pubalisen/).
