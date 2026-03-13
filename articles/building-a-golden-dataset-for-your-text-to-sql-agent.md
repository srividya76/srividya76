# Building a Golden Dataset for Your Text-to-SQL Agent

**Subtitle:** Crafting realistic, high-leverage test cases that keep a text-to-SQL agent honest  
**Author:** Srividya Vaidyanathan  
**Original published version:** https://srividya76.substack.com/p/building-a-golden-dataset-for-your  
**Published on Substack:** January 22, 2026

---

## Why this article exists

A rubric tells you how to judge a text-to-SQL agent. A golden dataset tells you what to judge it on.

If the previous step in evaluation design is defining pillars such as effectiveness, efficiency, robustness, and safety, the next step is building the set of test cases that will repeatedly expose whether the agent is actually meeting those standards.

This article turns the idea of a “golden dataset” into something practical and usable for a text-to-SQL system.

## What a golden dataset is in practice

For a text-to-SQL agent, a golden dataset is not just a random collection of prompts. It is a curated set of representative, important, and failure-prone questions where you already know what good behavior should look like.

A strong test case usually includes:

1. A natural-language user question  
2. A schema snapshot or schema version  
3. A canonical SQL query, where one exists  
4. An expected result for a known data snapshot, or a clear review rule  
5. Tags that identify what the case is testing

The key idea is repeatability. Whenever the agent changes — whether because of code, prompts, model version, tools, or safety logic — you run the same dataset again and compare outcomes.

## What good coverage looks like

A useful golden dataset does not aim for volume first. It aims for coverage across the kinds of questions and failures that matter.

For text-to-SQL, good coverage usually spans:

### 1. Query type coverage

You want cases across different query shapes, such as:

- Simple filters and aggregations
- Joins across related tables
- Time-based queries
- Top-K and ranking queries
- More complex aggregations

### 2. Difficulty coverage

Include a mix of:

- Straightforward questions
- Moderately complex questions
- Tricky edge cases

### 3. Robustness coverage

You want cases that test behavior under imperfect conditions, such as:

- Ambiguous questions
- Incomplete information
- Schema changes
- Data-quality problems like nulls and outliers

### 4. Safety coverage

Some cases should deliberately test whether the agent stays within policy and access limits, including:

- Permission boundaries
- Sensitive or PII-related requests
- Destructive or risky query requests
- Prompt-injection-style instructions

A dataset with 30 excellent cases is more useful than a dataset with 300 shallow ones.

## Six practical buckets of test cases

A clean way to start is to organize your dataset into six buckets.

## Bucket 1 — Simple filters and aggregations

These are the bread-and-butter cases. Your agent should get these right consistently.

Examples include:

- “What was the total revenue in 2024?”
- “How many active customers did we have in Europe in March 2024?”

These test:

- basic schema understanding
- mapping language to filters
- grouping and aggregation
- explanation clarity

## Bucket 2 — Joins and relationships

This is where many agents begin to struggle.

Examples include:

- revenue by country using `orders` joined to `customers`
- order counts by sales rep
- average order value by product category across multiple tables

These test:

- correct join logic
- correct grouping after joins
- avoiding double counting
- correct use of keys

## Bucket 3 — Time-based queries

Time interpretation is a common source of subtle failure.

Examples include:

- “Show total revenue by month for 2023.”
- “What was the revenue in the last 30 days?”
- “Compare total revenue for Q1 2024 vs Q1 2023.”

These test:

- date boundaries
- relative versus fixed time windows
- period comparison logic
- clarity about assumptions

## Bucket 4 — Ambiguity and edge cases

A trustworthy agent should not bluff when the question is vague.

Examples include:

- “Show me performance by product.”
- “How did we do last quarter?”
- requests involving a nonexistent category or plan

These test:

- whether the agent asks a clarifying question
- whether it states assumptions explicitly
- whether it avoids inventing unsupported answers
- transparent handling of uncertainty

## Bucket 5 — Failure and error-handling scenarios

You also need to test how the agent behaves when something breaks.

Examples include:

- a schema change where a column name has changed
- a query that risks timing out
- null-heavy or messy data

These test:

- graceful failure handling
- transparent error explanation
- adaptation versus guessing
- avoidance of silent data corruption

## Bucket 6 — Safety, permissions, and policy

These are hard-blocker cases.

Examples include:

- requests for restricted salaries or named employee data
- cross-tenant or out-of-scope data requests
- destructive requests such as deleting records
- prompt-injection-style requests to ignore policy

These test:

- permission enforcement
- tenant isolation
- sensitive-data handling
- policy compliance
- refusal behavior for unsafe actions

## Turning ideas into fully specified test cases

Each question idea should eventually become a structured artifact in your repo.

A practical minimal structure can look like this:

```yaml
id: T2SQL_001
bucket: simple_aggregation
description: "Total revenue in 2024"
user_question: "What was our total revenue in 2024?"
schema_version: "v1.3"
canonical_sql: |
  SELECT SUM(amount) AS total_revenue
  FROM orders
  WHERE order_date >= '2024-01-01'
    AND order_date < '2025-01-01';
expected_result:
  total_revenue: 1234567.89
rubric_tags:
  - effectiveness.intent_match
  - effectiveness.numeric_correctness
  - efficiency.latency
  - robustness.basic
  - safety.none
hard_blockers:
  - safety
