
### **Concept 16: Advanced Window Framing (`ROWS BETWEEN`)**

In B.Tech, if you need a monthly total, you `GROUP BY month`. But what if you are building a system that requires a **rolling 7-day average** or a **moving sum** for every single transaction as it happens? (This is exactly how you build features for things like live fraud detection models).

You use a Window Frame.

**The Syntax:**

```sql
SELECT 
    user_id,
    transaction_date,
    amount,
    SUM(amount) OVER (
        PARTITION BY user_id 
        ORDER BY transaction_date 
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) as rolling_7_txn_sum
FROM transactions;

```

**The Physics:** The engine sorts the data by date for each user. For every row it evaluates, it physically looks at the 6 rows immediately above it, adds them to the current row, outputs the sum, and then moves down one step.
*Note: Distributed streaming engines (like Flink) use this exact SQL logic to hold "state" in memory for live data streams.*

---

### **Concept 17: Flattening the JSON Array (`UNNEST` / `EXPLODE`)**

In traditional databases, data is perfectly relational (1st Normal Form). In modern data engineering, data arrives from Kafka or APIs as massive JSON payloads containing nested arrays.

Imagine an `orders` table with a column called `items` that contains an array: `['laptop', 'mouse', 'keyboard']`.

**The Goal:** You need a separate row for every item so you can join it to an inventory table.

**The Production SQL (Unnesting):**

```sql
-- Syntax varies by engine (Postgres uses UNNEST, Spark/DuckDB use EXPLODE)
SELECT 
    order_id, 
    customer_id, 
    item
FROM orders
CROSS JOIN UNNEST(items) AS t(item);

```

**The Physics:** This is called a **Table-Generating Function**. The engine takes a single row, reads the array, and physically generates new rows on the fly, copying the `order_id` and `customer_id` for each element in the array.

---

### **Concept 18: Slowly Changing Dimensions (SCD Type 2)**

This is a fundamental data warehousing concept.
In B.Tech, if a customer updates their address from 'New York' to 'California', you write an `UPDATE` statement.

In a production Data Warehouse, **you never run an `UPDATE` to overwrite history.** If you overwrite their address, your historical sales reports for New York will suddenly drop, and California will spike, ruining the company's financial records.

**The Grandmaster Solution (SCD Type 2):**
Instead of updating, you insert a *new* row and use SQL dates to track the timeline.

**Table: `Customer_Dim**`

| id | name | state | valid_from | valid_to | is_current |
| --- | --- | --- | --- | --- | --- |
| 1 | Alice | NY | 2023-01-01 | 2026-02-23 | False |
| 1 | Alice | CA | 2026-02-24 | 9999-12-31 | True |

**The Production Query:**
When you join sales to customers, you must join on the date the sale happened so the engine links to the correct historical state!

```sql
SELECT sum(s.amount), c.state
FROM sales s
JOIN Customer_Dim c 
  ON s.customer_id = c.id 
  AND s.sale_date >= c.valid_from 
  AND s.sale_date < c.valid_to
GROUP BY c.state;

```

---

### **Concept 19: The "Salting" Workaround for Data Skew**

In Level 5, we talked about **Data Skew**—when one CPU core gets crushed by a massive partition of data (like a celebrity account with millions of followers, or a massive `NULL` group).

What if you *can't* filter it out? What if you *must* group by that skewed key?

**The Grandmaster Hack (Salting):**
You artificially break the massive group into smaller, random chunks, distribute them to all CPU cores to aggregate partially, and then sum the partials together.

**Step 1: Add Salt (Randomness)**

```sql
WITH Salted_Data AS (
    SELECT 
        user_id, 
        amount,
        -- Generate a random number between 0 and 9
        CAST(RAND() * 10 AS INT) as salt 
    FROM massive_transactions
),
-- Step 2: Partial Aggregation (Distributed across 10 cores)
Partial_Agg AS (
    SELECT 
        user_id, 
        salt, 
        SUM(amount) as partial_sum
    FROM Salted_Data
    GROUP BY user_id, salt
)
-- Step 3: Final Aggregation (Done quickly on one core)
SELECT 
    user_id, 
    SUM(partial_sum) as final_sum
FROM Partial_Agg
GROUP BY user_id;

```

This is how Data Engineers keep massive distributed Spark or Flink clusters from crashing.

---

### **Concept 20: Pipeline Orchestration (Views vs. CTAS)**

How do we actually save the results of these massive queries so dashboards can read them?

* **Standard View (`CREATE VIEW`):** It is just a saved text file of your SQL query. It stores **no data**. When the CEO opens the dashboard, the engine runs your entire 500-line SQL query from scratch. (Great for logic sharing, terrible for heavy compute).
* **CTAS (`CREATE TABLE AS SELECT`):** This is the bread and butter of data engineering pipelines.
```sql
CREATE TABLE daily_sales_summary AS 
SELECT ... (massive complex query) ...

```


This physically executes the query and writes the final results to the hard drive as a brand-new table. The dashboard queries this new table, which returns in milliseconds. You schedule this CTAS to run every night at 2 AM.

---
