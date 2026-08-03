# Part 16: Structured Data in RAG

> **The Question:** "How do you handle structured data (tables, SQL databases) in a RAG pipeline?"

---

## The Technical Breakdown

### The Problem

Standard RAG works with unstructured text. But real-world data includes tables, CSV files, SQL databases, and spreadsheets. Embedding a row from a database table produces a meaningless vector because rows lack natural language context.

```
Row: | John Smith | 2024-01-15 | $4,999 | Premium | Active |

Embedded as text: "John Smith 2024-01-15 4999 Premium Active"
→ What does this even mean? No column headers, no context.
```

### Strategies for Structured Data

#### 1. Text-to-SQL (NL2SQL)
Convert the natural language question into a SQL query:

```
User: "How many premium users signed up in January 2024?"
           ↓ LLM converts to SQL
SQL: SELECT COUNT(*) FROM users 
     WHERE tier = 'Premium' 
     AND created_at BETWEEN '2024-01-01' AND '2024-01-31'
           ↓ Execute query
Result: 847
           ↓ LLM generates natural language
Answer: "847 premium users signed up in January 2024."
```

#### 2. Table Serialization
Convert tables to natural language descriptions:

```
Original table row:
| Product | Price | Category | Stock |
|---------|-------|----------|-------|
| Widget A | $29.99 | Electronics | 150 |

Serialized: "Product Widget A is in the Electronics category, 
             priced at $29.99, with 150 units in stock."
```

#### 3. Hybrid: SQL + Vector Search
Route queries to the appropriate system:

```
Query Router:
  "What's our total revenue?" → SQL (aggregation query)
  "Why did revenue drop in Q3?" → Vector search (qualitative analysis)
  "Compare Q3 revenue to the CEO's Q3 statement" → Both (SQL + docs)
```

#### 4. Table-Aware Embeddings
Embed tables with column headers as context:

```python
# Instead of: "John Smith 2024-01-15 4999 Premium Active"
# Embed:
chunk = """
Table: Customer Orders
Column: customer_name = John Smith
Column: order_date = 2024-01-15
Column: order_value = $4,999
Column: tier = Premium
Column: status = Active
"""
```

### Text-to-SQL Architecture

```
User Question
     │
     ▼
┌──────────────┐
│ Schema        │  ← Table names, column names, types,
│ Discovery     │    sample values, relationships
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ SQL           │  ← LLM generates SQL using schema context
│ Generation    │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ SQL           │  ← Syntax check, injection prevention,
│ Validation    │    query complexity limits
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Execute       │  ← Run against database (read-only!)
│ Query         │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Answer        │  ← LLM converts SQL results to natural language
│ Generation    │
└──────────────┘
```

### Key Challenges

| Challenge | Solution |
|-----------|----------|
| SQL injection risk | Parameterized queries, read-only DB user, query allow-list |
| Complex schemas | Provide only relevant tables/columns in context, not entire schema |
| Ambiguous column names | Add column descriptions in schema metadata |
| Large result sets | LIMIT clauses, pagination, summarization |
| Join complexity | Provide example queries for common joins |
| Wrong SQL generation | Retry with error message, self-correction loop |

---

## Accuracy Check
| Claim | Verified? |
|-------|-----------|
| Text-to-SQL converts natural language to executable SQL queries | ✅ Rajkumar et al. (2022), industry standard pattern |
| Table serialization improves embedding quality for tabular data | ✅ Empirically validated, LlamaIndex table handling docs |
| Hybrid routing (SQL + vector) handles both quantitative and qualitative queries | ✅ Production pattern used by Databricks, Snowflake AI features |

## Scenario Examples
### A: A business intelligence chatbot connects to a Postgres database with 50 tables. Users ask "What were our top 5 products by revenue last quarter?" The system: (1) selects relevant tables (orders, products, order_items), (2) generates SQL with proper JOINs and GROUP BY, (3) validates the query is read-only, (4) executes and gets results, (5) generates: "Your top 5 products by Q3 revenue were: 1. Enterprise Plan ($2.3M), 2. Pro Plan ($1.8M)..." This is impossible with vector search alone.
### B: An analytics platform combines SQL and RAG. "Why did churn increase in Q3?" → SQL query shows churn went from 4.2% to 6.8%. Vector search retrieves internal reports mentioning "pricing change in July" and "competitor launch in August." The LLM synthesizes both: "Churn increased 62% in Q3, likely driven by the July price increase and a competitive product launched by Competitor X in August."

## Follow-Up Questions
### Q1: "How do you prevent SQL injection in Text-to-SQL systems?"
**Answer:** Multiple layers: (1) **Read-only database user** — the query account can only SELECT, never INSERT/UPDATE/DELETE. (2) **Query validation** — regex check that the generated SQL contains no DDL/DML statements. (3) **Parameterized execution** — use prepared statements, never string interpolation. (4) **Row/time limits** — LIMIT 1000 and query timeout of 10 seconds. (5) **Allow-list tables** — only expose specific tables to the system.

### Q2: "How accurate is Text-to-SQL with current LLMs?"
**Answer:** On standard benchmarks (Spider, Bird-SQL): GPT-4 achieves ~85% execution accuracy on simple queries and ~60-70% on complex multi-table JOINs. In production, accuracy depends on schema complexity and query ambiguity. Best practices that boost accuracy: (1) include 3-5 example queries in the prompt, (2) provide column descriptions and sample values, (3) use a self-correction loop (execute → if error → retry with error message).

### Q3: "When should you use Text-to-SQL vs embedding the data?"
**Answer:** Text-to-SQL for: aggregations (SUM, COUNT, AVG), filters (WHERE), sorting (ORDER BY), joins, any quantitative question. Embedding for: qualitative analysis, fuzzy matching ("products similar to X"), free-text fields (descriptions, comments, notes). Rule of thumb: if the answer requires a calculation, use SQL. If the answer requires understanding meaning, use embeddings.

---

<p align="center"><a href="../README.md">← Back to all questions</a></p>
