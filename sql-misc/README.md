# 🔀 sql-misc

This directory contains notes that sit **outside the core SQL mechanics** — broader engineering decisions about where SQL fits in a data system and where it starts to break down.

---

## 📄 Files & What They Cover

| File | Key Topics |
|---|---|
| [`when-does-sql-nosql-struggle.md`](when-does-sql-nosql-struggle.md) | When relational SQL becomes unsuitable at scale, when NoSQL fails too, workloads that break both, and a decision framework for choosing the right tool |

### `when-does-sql-nosql-struggle.md`

Answers the question: *"I have 500M+ records per day. What kinds of operations will SQL or NoSQL struggle with?"*

Covers:

- **When SQL becomes unsuitable** — high unbatched write throughput, massive multi-table joins, repeated full-scan aggregations, cross-region consistency
- **When NoSQL becomes unsuitable** — complex multi-table joins, strong transactional guarantees, ad-hoc analytical queries
- **Workloads that break both** — full daily reprocessing, real-time stream enrichment, high-cardinality indexing
- **A 5-question decision framework** for choosing between SQL, NoSQL, a data warehouse, Spark, or Flink
- **A mental model table** mapping workload type to best-fit technology

---

## 🗺️ How It Fits Into the Learning Journey

After building SQL mechanics (`sql-internals`) and production query patterns (`sql_patterns`), this directory provides **system-level context**: knowing when to reach for a different tool entirely is as important as knowing how to write efficient SQL.

This is particularly useful for:
- System design interview preparation
- Deciding whether a new data pipeline belongs in a relational database, a warehouse, or a streaming engine
- Understanding why analytics engineering often sits at the boundary between SQL and distributed compute

---

## 🤝 Contributing

See the [root README](../README.md) for contribution guidelines.

## 📄 License

Content by [Sepuri Sai Krishna](https://github.com/SEPURI-SAI-KRISHNA).
