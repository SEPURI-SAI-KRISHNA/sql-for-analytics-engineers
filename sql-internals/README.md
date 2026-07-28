# 🔧 sql-internals

This directory covers **how SQL engines actually work** — not just what the syntax does, but the physical and logical mechanics that determine correctness and performance. Each file builds on the previous one, guiding you from foundational NULL semantics through distributed-systems-level query patterns.

The material is structured as a progressive series of concepts and challenges. Working through them in order gives you the mental model that separates an analytics engineer who writes correct SQL from one who writes production-grade SQL.

---

## 📄 Files & What They Cover

| File | Concepts | Key Topics |
|---|---|---|
| [`three_valued_logic_and_coalesce.md`](three_valued_logic_and_coalesce.md) | 1–2 | NULL as "Unknown", three-valued logic, COALESCE, IS NULL |
| [`the_aggregation_trap.md`](the_aggregation_trap.md) | 3 | How aggregates ignore NULLs, AVG vs SUM/COUNT, GROUP BY NULLs, HAVING |
| [`the_join_trap.md`](the_join_trap.md) | 4–5 | INNER JOIN "drop", LEFT JOIN NULL injection, ON vs WHERE filter placement |
| [`the_multiverse_rows.md`](the_multiverse_rows.md) | 6 | Window functions vs GROUP BY, ROW_NUMBER / RANK / DENSE_RANK, running totals, correlated subqueries |
| [`the_declarative_engine_and_execution_plan.md`](the_declarative_engine_and_execution_plan.md) | 7–10 | EXPLAIN plans, SARGability, data skew and the shuffle, CTEs for pipeline readability |
| [`row_vs_column_killer.md`](row_vs_column_killer.md) | 11–15 | Row vs columnar storage, partition pruning, broadcast joins, idempotent MERGE/UPSERT, HyperLogLog |
| [`advanced_window_framing.md`](advanced_window_framing.md) | 16–20 | ROWS BETWEEN frames, UNNEST/EXPLODE, SCD Type 2, salting for data skew, Views vs CTAS |

---

## 🗺️ How It Fits Into the Learning Journey

This is the **foundation layer** of the repository. Concepts here explain the mechanics that every pattern in [`sql_patterns/`](../sql_patterns/) relies on. If a production query uses a window function frame, a MERGE statement, or a CTE pipeline, the reasoning for *why* it's written that way is explained here.

**Recommended approach:**
- Read each file in order — concept numbers are sequential across files.
- Attempt the challenge questions at the end of each file before reading the answers.
- Pay particular attention to the "traps" (the aggregation trap, the join trap, the correlated subquery trap) — these are the most common sources of silent data errors in analytics work.

---

## 🤝 Contributing

See the [root README](../README.md) for contribution guidelines.

## 📄 License

Content by [Sepuri Sai Krishna](https://github.com/SEPURI-SAI-KRISHNA).
