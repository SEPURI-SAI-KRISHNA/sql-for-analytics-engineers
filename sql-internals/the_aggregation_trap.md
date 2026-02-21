**The Aggregation Trap**.

This is where 90% of candidates fail because they assume SQL math works like Excel math. It does not.

We will use the **same table** so you can see how the logic changes when we switch from row-by-row checks to "Group" checks.

**Table: `Employees` (Refresher)**

| id | name | salary | manager_id |
| --- | --- | --- | --- |
| 1 | Alice | 1000 | NULL |
| 2 | Bob | 0 | 1 |
| 3 | Charlie | 2000 | 1 |
| 4 | David | NULL | 1 |
| 5 | Eve | 3000 | 2 |

---

### **Concept 3: The "Ignore" Rule**

When you use Aggregate functions (`SUM`, `AVG`, `COUNT`, `MIN`, `MAX`), SQL **ignores NULLs completely**. It pretends they don't exist.

* **Exception:** `COUNT(*)` counts **rows**. It doesn't care what's inside the rows.
* **The Danger:** `AVG` is `SUM / COUNT`. If `COUNT` ignores NULLs, your denominator is smaller than you think!

---

### **Level 2 Challenge: Questions 6-10**

Try to solve these mentally.

**Q6. The Counting Paradox**
You want to count how many employees represent a cost to the company (have a salary recorded).
Query:

```sql
SELECT COUNT(*) as total_rows, COUNT(salary) as salary_count
FROM Employees;

```

* **What are the two values returned?**

**Q7. The Average Trap (Crucial Interview Question)**
You want the average salary of the department.
Query: `SELECT AVG(salary) FROM Employees;`

* **Calculate the exact number.** (Hint: Don't just guess. Do the math: Sum / Count).

**Q8. The "Sum of Nothing"**
Imagine we filter for a non-existent employee (ID = 99).
Query: `SELECT SUM(salary) FROM Employees WHERE id = 99;`

* **What is the result?** (Is it `0`? Is it `NULL`? Is it an error?)

**Q9. The Grouping Mystery**
We want to group employees by their manager to see how many people report to each.
Query:

```sql
SELECT manager_id, COUNT(*) 
FROM Employees 
GROUP BY manager_id;

```

* **How many groups (rows) are returned?**
* **Does the `NULL` manager (Alice) get her own group?**

**Q10. The `HAVING` Filter**
We want to find managers who manage **more than 1 person**.
Query:

```sql
SELECT manager_id
FROM Employees
WHERE COUNT(*) > 1
GROUP BY manager_id;

```

* **This query throws an ERROR. Why?**

*(Take a moment. Scroll down for answers.)*

---

### **Level 2 Answers & Explanations**

**A6. Result:** `total_rows = 5`, `salary_count = 4`

* **Why?**
* `COUNT(*)` counts every row: 1, 2, 3, 4, 5.
* `COUNT(salary)` checks the salary column. It sees: `1000`, `0`, `2000`, `NULL`, `3000`.
* It ignores the `NULL`. It counts 4 values.


* **Grandmaster Insight:** If you want to count "Employees with no salary" (David), you use `COUNT(*) - COUNT(salary)`.

**A7. Result:** **1500** (Not 1200!)

* **The Math:**
* Sum = 1000 + 0 + 2000 + 3000 = 6000.
* Count of Salaries = 4 (David is ignored!).
* Average = 6000 / 4 = **1500**.


* **The Logic:** SQL calculates the average salary *of people who have a salary*. It does not treat NULL as 0.
* **Correction:** If you wanted to treat David as 0 and divide by 5, you must write:
`AVG(COALESCE(salary, 0))` -> 6000 / 5 = 1200.

**A8. Result:** **NULL**

* **Why?** If `SUM` finds no matching rows, it returns `NULL`, not `0`.
* **The Trap:** If you use this result in a math operation later (e.g., `Result + 10`), it will turn the whole thing to NULL.
* **Fix:** Always wrap sums in Coalesce if you expect empty sets: `COALESCE(SUM(salary), 0)`.

**A9. Result:** **3 Groups**

1. **NULL**: 1 count (Alice reports to no one)
2. **1**: 3 count (Bob, Charlie, David report to 1)
3. **2**: 1 count (Eve reports to 2)

* **Why?** Unlike `WHERE`, `GROUP BY` treats NULLs as a valid bucket. It puts all "managerless" people into one single group.

**A10. Result:** **ERROR**

* **Why?** Order of Operations (Module 2).
* `WHERE` runs **before** `GROUP BY`.
* At the `WHERE` stage, the groups haven't been formed yet, so `COUNT(*)` doesn't exist.


* **Fix:** You must use `HAVING`.
```sql
SELECT manager_id
FROM Employees
GROUP BY manager_id
HAVING COUNT(*) > 1;

```



---

