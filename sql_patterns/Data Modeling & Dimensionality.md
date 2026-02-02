
## Data Modeling & Dimensionality

### 7. The "Date Scaffold" (Filling Reporting Gaps)

**Problem Statement:** You need to report daily signups. If a day has zero signups, it simply doesn't exist in the database. A standard `GROUP BY` will skip that day, breaking charts that expect continuous time axes. You need to force zero-value rows for missing dates.

**Optimized SQL (Standard SQL using Recursive CTE):**

```sql
WITH RECURSIVE date_spine AS (
    -- 1. Generate a continuous list of dates (The "Spine")
    SELECT DATE('2023-01-01') as calendar_date
    UNION ALL
    SELECT calendar_date + INTERVAL '1 day'
    FROM date_spine
    WHERE calendar_date < CURRENT_DATE
),
daily_signups AS (
    -- 2. Your actual data, which might have gaps
    SELECT 
        DATE(created_at) as signup_date,
        COUNT(*) as total_signups
    FROM users
    GROUP BY 1
)
SELECT 
    ds.calendar_date,
    COALESCE(s.total_signups, 0) as total_signups -- Replace NULL with 0
FROM date_spine ds
LEFT JOIN daily_signups s ON ds.calendar_date = s.signup_date
ORDER BY 1 DESC;

```

**Explanation:**

* **The Spine:** We create a "master" table of dates (`date_spine`). In production, this often comes from a permanent `dim_date` table.
* **The Join:** We use a `LEFT JOIN` starting from the Spine. This ensures every date is preserved.
* **The Null Handler:** `COALESCE` is critical. If no match is found in the data, the join returns `NULL`, which we convert to `0` for the report.

**Performance Considerations:**

* **Specific Engines:**
* **Postgres:** Use `GENERATE_SERIES`.
* **Snowflake:** Use `Generator(rowcount => ...)` or a dedicated Calendar table (recommended).
* **BigQuery:** Use `UNNEST(GENERATE_DATE_ARRAY(...))`.


* Avoid generating spines for overly wide ranges (e.g., 100 years) inside a query.

---

### 8. Range Joins (Non-Equi Joins for Attribution)

**Problem Statement:** You have a `transactions` table and a `marketing_campaigns` table. Campaigns have a `start_date` and `end_date`. You need to attribute each transaction to the campaign that was active *at the moment* the transaction occurred.

**Optimized SQL:**

```sql
SELECT 
    t.transaction_id,
    t.amount,
    t.transaction_date,
    c.campaign_name,
    c.campaign_type
FROM transactions t
LEFT JOIN campaigns c 
    ON t.transaction_date >= c.start_date 
    AND t.transaction_date <= c.end_date
    -- Optional: Tie-breaker if ranges overlap
    AND c.status = 'active';

```

**Explanation:**

* **Non-Equi Join:** Instead of joining on `ID = ID`, we join using inequality operators (`>=`, `<=`, or `BETWEEN`).
* **Fan-out Risk:** If campaigns overlap (e.g., a "Summer Sale" and a "Flash Sale" are active on the same day), this join will duplicate the transaction rows (one for each matching campaign). You must ensure your dimension table (`campaigns`) has mutually exclusive ranges, or aggregate the results after joining.

**Performance Considerations:**

* **The "Explosion" Problem:** Non-equi joins are computationally expensive because the database cannot simply hash the keys. It effectively has to scan ranges.
* **Optimization:** Ensure the `start_date` and `end_date` columns are indexed/clustered. In Spark/Databricks, this often triggers a "Broadcast Nested Loop Join," which can be slow; bucket your data by month or year if possible to narrow the search space.

---

