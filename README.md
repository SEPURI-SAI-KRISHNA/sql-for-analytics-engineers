# 🗃️ SQL for Analytics Engineers

> Advanced SQL patterns for analytics, data engineering, and interviews — with the reasoning behind each one.

![SQL](https://img.shields.io/badge/SQL-4479A1?style=flat-square&logo=postgresql&logoColor=white)
![Analytics](https://img.shields.io/badge/Analytics%20Engineering-FF7A59?style=flat-square)

A practical SQL reference built for **analytics engineers, data engineers, and anyone preparing for technical interviews**. Every topic comes with the query, the intuition, and the production context — not just syntax, but *why* it matters at scale.

---

## 🎯 Who This Is For

- **Analytics engineers** who want to move beyond basic `SELECT` and `GROUP BY`
- **Data engineers** looking for production-ready patterns with performance reasoning
- **Interview candidates** preparing for SQL-heavy data roles
- **Students** bridging the gap between academic SQL and real-world data systems

---

## 📂 Directory Guide

| Directory | What's Inside |
|---|---|
| [`sql-internals/`](sql-internals/) | How SQL engines actually work — NULLs, joins, window functions, query plans, distributed systems |
| [`sql_patterns/`](sql_patterns/) | Reusable production query patterns — funnels, retention, cohorts, SCD, sessionization |
| [`sql-misc/`](sql-misc/) | Broader engineering decisions — when SQL or NoSQL breaks down at scale |

---

## 🗺️ Recommended Learning Order

Start with `sql-internals` to build a solid mental model of how SQL executes, then move to `sql_patterns` to see those concepts applied to real analytics problems. Use `sql-misc` to understand where SQL fits (and doesn't fit) in larger system design.

### 1 · `sql-internals/` — SQL Foundations → Production Mindset

Work through the files in this order, following the concept numbering:

1. [`three_valued_logic_and_coalesce.md`](sql-internals/three_valued_logic_and_coalesce.md) — NULL semantics, three-valued logic, COALESCE
2. [`the_aggregation_trap.md`](sql-internals/the_aggregation_trap.md) — How aggregates handle NULLs, GROUP BY, HAVING
3. [`the_join_trap.md`](sql-internals/the_join_trap.md) — INNER vs LEFT JOIN, ON vs WHERE, Cartesian products
4. [`the_multiverse_rows.md`](sql-internals/the_multiverse_rows.md) — Window functions vs GROUP BY, RANK, running totals
5. [`the_declarative_engine_and_execution_plan.md`](sql-internals/the_declarative_engine_and_execution_plan.md) — EXPLAIN plans, SARGability, data skew, CTEs
6. [`row_vs_column_killer.md`](sql-internals/row_vs_column_killer.md) — Columnar storage, partition pruning, broadcast joins, idempotency
7. [`advanced_window_framing.md`](sql-internals/advanced_window_framing.md) — ROWS BETWEEN, UNNEST/EXPLODE, SCD Type 2, salting, CTAS

### 2 · `sql_patterns/` — Production Query Patterns

- [`sql_patterns.md`](sql_patterns/sql_patterns.md) — Gaps & islands, sessionization, SCD Type 2, funnel analysis, retention and cohort heatmaps
- [`Data Modeling & Dimensionality.md`](sql_patterns/Data%20Modeling%20%26%20Dimensionality.md) — Date spines, range joins, late-arriving dimensions

### 3 · `sql-misc/` — System Design Context

- [`when-does-sql-nosql-struggle.md`](sql-misc/when-does-sql-nosql-struggle.md) — When to reach for Spark, Flink, or a data warehouse instead of a relational database

---

## 📖 How to Use This Repo

**For study:** Work through `sql-internals` sequentially. Each file ends with a challenge section — attempt the questions before reading the answers.

**For reference:** Jump directly to a pattern in `sql_patterns` when you need a production-ready query template for funnels, retention, or dimensional modeling.

**For interviews:** The `sql-internals` challenge questions mirror common senior-level interview problems. Pay special attention to NULL behavior, the ON vs WHERE distinction in LEFT JOINs, and the correlated subquery vs window function trade-off.

---

## 🎯 Topics Covered

- **NULL semantics & three-valued logic**
- **Window functions** — ranking, running totals, lead/lag, ROWS BETWEEN frames
- **Join mechanics** — INNER, LEFT, CROSS, the ON vs WHERE trap
- **Query optimization** — SARGability, execution plans, partition pruning, broadcast joins
- **Distributed SQL** — data skew, salting, columnar storage, CTEs at scale
- **Funnel & retention analysis** — multi-step conversion, N-day and cohort retention
- **Slowly changing dimensions (SCD Type 2)**
- **Data modeling** — date spines, range joins, late-arriving dimensions
- **Idempotency** — MERGE / UPSERT patterns for reliable pipelines
- **SQL vs NoSQL trade-offs** at 500M+ records/day

---

## 🤝 Contributing

Spotted an error or have a pattern worth adding? Open an issue or pull request. Keep contributions documentation-only and accurate to existing repo content.

---

## 📄 License

This repository is for educational reference. Content by [Sepuri Sai Krishna](https://github.com/SEPURI-SAI-KRISHNA).

<sub>📂 Explore more at **[sepuri-sai-krishna.pages.dev](https://sepuri-sai-krishna.pages.dev)**</sub>
