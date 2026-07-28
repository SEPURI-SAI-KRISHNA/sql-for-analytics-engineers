# 📐 sql_patterns

This directory contains **reusable production SQL patterns** for common analytics engineering problems. Each pattern comes with an optimized query, an explanation of the underlying mechanics, and performance notes for large-scale engines like Snowflake, BigQuery, Spark, and PostgreSQL.

These are the patterns you reach for when building reporting pipelines, product analytics, and data warehouse models — not toy examples, but queries shaped by real production constraints.

---

## 📄 Files & What They Cover

### [`sql_patterns.md`](sql_patterns.md) — Advanced Temporal Patterns

| Pattern | Problem Solved |
|---|---|
| **Gaps and Islands** | Find consecutive streaks in a date sequence (e.g., login streaks) |
| **Sessionization with Cumulative Sums** | Group event logs into sessions based on inactivity gaps |
| **SCD Type 2 with LEAD** | Generate effective/expiry date ranges from a changelog table |
| **N-Step Funnel Analysis** | Count and rate users through a defined event sequence |
| **Classic N-Day Retention** | Measure what percentage of a cohort returned on day 1, 7, etc. |
| **Cohort Heatmap (Month-over-Month)** | Build the standard VC/PM retention matrix by join month |

### [`Data Modeling & Dimensionality.md`](Data%20Modeling%20%26%20Dimensionality.md) — Dimensional Modeling Patterns

| Pattern | Problem Solved |
|---|---|
| **Date Scaffold** | Fill reporting gaps by generating a continuous date spine and left-joining actual data |
| **Range Joins** | Attribute facts (e.g., transactions) to dimension records valid during a time window |
| **Late-Arriving Dimensions** | Prevent data loss when fact records arrive before their dimension rows exist |

---

## 🗺️ How It Fits Into the Learning Journey

These patterns are the **applied layer** of the repository. They assume familiarity with the concepts in [`sql-internals/`](../sql-internals/) — window functions, NULL handling, CTEs, and join mechanics — and show how those concepts combine to solve real analytics problems.

**Recommended approach:**
- If a pattern uses a technique you haven't seen before (e.g., LEAD for SCD, cumulative SUM for sessionization), read the relevant concept file in `sql-internals` first.
- Use these queries as starting templates. The `Performance Considerations` sections in each pattern explain what to adapt for your specific engine.
- The funnel and retention patterns are directly applicable to product analytics work in tools like dbt, Looker, or Metabase.

---

## 🤝 Contributing

See the [root README](../README.md) for contribution guidelines.

## 📄 License

Content by [Sepuri Sai Krishna](https://github.com/SEPURI-SAI-KRISHNA).
