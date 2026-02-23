Standard `GROUP BY` ruins this because it collapses the rows. You lose the individual transaction details. This is where **Window Functions** step in.

Let's set up our final playground.

**Table: `Transactions**`

| txn_id | account_id | amount | status |
| --- | --- | --- | --- |
| 101 | A | 50 | Cleared |
| 102 | A | 150 | Cleared |
| 103 | A | 150 | Flagged |
| 104 | B | 5000 | Flagged |
| 105 | B | 10 | Cleared |

---

### **Concept 6: The "Window" vs the "Group"**

* **`GROUP BY`** takes 100 rows, groups them into 5 buckets, and returns exactly **5 rows**. The original details are gone.
* **`OVER (PARTITION BY ...)`** takes 100 rows, calculates the math for the 5 buckets in the background, but still returns all **100 rows**. It simply tacks the answer onto the end of each row.

In distributed systems (like Spark or Flink), `PARTITION BY` physically routes data with the same key to the same processing node so it can calculate the window sequentially.

---

### **Level 4 Challenge: Questions 16-20**

These are the absolute top-tier interview questions. Think them through carefully.

**Q16. The Ranking Dilemma (`RANK` vs `DENSE_RANK` vs `ROW_NUMBER`)**
We want to rank the transactions for Account A by `amount` in descending order. Account A has amounts: 150, 150, 50.

```sql
SELECT amount, 
       ROW_NUMBER() OVER(ORDER BY amount DESC) as rn,
       RANK() OVER(ORDER BY amount DESC) as rnk,
       DENSE_RANK() OVER(ORDER BY amount DESC) as drnk
FROM Transactions WHERE account_id = 'A';

```

* **For the two transactions worth 150, what exact numbers are output for `rn`, `rnk`, and `drnk`?**

**Q17. The Window Filter Trap**
We want to find only the highest transaction for each account.

```sql
SELECT txn_id, account_id, amount,
       MAX(amount) OVER(PARTITION BY account_id) as max_amt
FROM Transactions
WHERE amount = MAX(amount) OVER(PARTITION BY account_id);

```

* **This query throws an ERROR. Why? And how do you fix it?**

**Q18. The Running Total (Cumulative Sum)**
To track account velocity (crucial for fraud detection), we need a running total of the amounts.

```sql
SELECT txn_id, amount,
       SUM(amount) OVER(PARTITION BY account_id ORDER BY txn_id) as running_total
FROM Transactions;

```

* **What is the `running_total` for txn_id 103 (Account A's third transaction)?**
* **What happens if we remove the `ORDER BY txn_id` from the `OVER()` clause?**

**Q19. The Correlated Subquery Assassin**
You need to find all transactions that are larger than the average transaction size *for that specific account*.
**Query A (Correlated Subquery):**

```sql
SELECT * FROM Transactions t1
WHERE amount > (SELECT AVG(amount) FROM Transactions t2 WHERE t1.account_id = t2.account_id);

```

**Query B (Window Function + CTE):**

```sql
WITH CTE AS (
  SELECT *, AVG(amount) OVER(PARTITION BY account_id) as avg_amt FROM Transactions
)
SELECT * FROM CTE WHERE amount > avg_amt;

```

* **Both give the exact same correct result. In a table with 10 million rows, why will Query A likely crash your database while Query B finishes in seconds?**

**Q20. Bringing it all home: `EXISTS` vs `IN**`
We want to find all accounts that have at least one 'Flagged' transaction.

```sql
-- Option 1
SELECT DISTINCT account_id FROM Transactions WHERE status = 'Flagged';

-- Option 2
SELECT a.account_id FROM Accounts a 
WHERE EXISTS (SELECT 1 FROM Transactions t WHERE t.account_id = a.account_id AND t.status = 'Flagged');

```

* **If Account B has 10,000 flagged transactions, which option is better and why?**

*(Take your time. This is the final test before graduation. Scroll down for the answers.)*

---

### **Level 4 Answers & Explanations**

**A16. The Ranking Dilemma**

* **`ROW_NUMBER`**: 1, 2, 3. (Always increments, no matter what. Breaks ties randomly based on disk read order).
* **`RANK`**: 1, 1, 3. (Gives ties the same rank, but *skips* the next number).
* **`DENSE_RANK`**: 1, 1, 2. (Gives ties the same rank, and *does not skip* numbers).
* **Grandmaster Tip:** Always use `ROW_NUMBER` for deduplication, and `DENSE_RANK` when dealing with leaderboards or finding the "Nth highest salary".

**A17. The Window Filter Trap**

* **Why it errors:** Order of Operations again (Module 2). `WHERE` runs before the `SELECT` clause. Window functions are calculated during the `SELECT` phase. You cannot filter on a calculation that hasn't happened yet.
* **The Fix:** Wrap it in a CTE (Common Table Expression) or an inline subquery.
```sql
WITH RankedTxns AS (
   SELECT *, ROW_NUMBER() OVER(PARTITION BY account_id ORDER BY amount DESC) as rn
   FROM Transactions
)
SELECT * FROM RankedTxns WHERE rn = 1;

```



**A18. The Running Total**

* **Running Total for 103:** 50 + 150 + 150 = **350**.
* **What if `ORDER BY` is removed?** It stops being a running total. It just becomes the grand total for the partition. Every row for Account A would just say `350`. The `ORDER BY` inside an `OVER` clause fundamentally changes `SUM()` from a "total sum" to an "accumulating sum".

**A19. The Correlated Subquery Assassin**

* **Why Query A fails:** A Correlated Subquery (indicated by `t1.account_id = t2.account_id` inside the subquery) forces the engine to run the subquery **Row-By-Row**. If you have 10 million rows, it runs that inner `AVG()` query 10 million individual times (RBAR - Row By Agonizing Row).
* **Why Query B wins:** Window functions process sets of data in memory. It scans the table once, hashes the partitions, calculates the averages in bulk, and attaches them. Always replace Correlated Subqueries with Window Functions or Joins for scale.

**A20. `EXISTS` vs `IN` (The Short-Circuit)**

* **Option 2 (`EXISTS`) is far superior.** * **Why?** `EXISTS` is a boolean check. The moment the SQL engine finds the *very first* 'Flagged' transaction for Account B, it returns TRUE and immediately stops scanning Account B's history. Option 1 will read all 10,000 flagged rows, hold them in memory, and then perform a costly `DISTINCT` sort operation to deduplicate them down to 1 row.

---