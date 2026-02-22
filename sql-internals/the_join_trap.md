
Forget the standard "Venn Diagram" explanations you see online. We are going to look at what the engine actually does: **Row matching and NULL injection.**

Here is our new playground. We have added a `Departments` table and simplified `Employees`.

**Table A: `Employees**`

| emp_id | name | dept_id |
| --- | --- | --- |
| 1 | Alice | 10 |
| 2 | Bob | 10 |
| 3 | Charlie | 20 |
| 4 | David | NULL |

**Table B: `Departments**`

| dept_id | dept_name |
| --- | --- |
| 10 | Engineering |
| 20 | Sales |
| 30 | Marketing |

---

### **Concept 4: The INNER JOIN "Drop"**

An `INNER JOIN` demands a perfect match. If the engine looks at David (`dept_id = NULL`), it searches the Departments table for a `NULL` ID. Because `NULL = NULL` is "Unknown" (remember Level 1!), it finds no match. **David is dropped.** Similarly, it looks at Marketing (`dept_id = 30`), searches the Employees table, finds no 30, and **Marketing is dropped.**

### **Concept 5: The LEFT JOIN "NULL Injection"**

A `LEFT JOIN` guarantees that **every row from the Left table survives**.
If the engine can't find a match in the Right table, it doesn't drop the row. Instead, it creates a "ghost row" of `NULL`s for the Right table and attaches it.

---

### **Level 3 Challenge: Questions 11-15**

Try to predict the exact output.

**Q11. The Basic Inner Join**

```sql
SELECT e.name, d.dept_name 
FROM Employees e 
INNER JOIN Departments d ON e.dept_id = d.dept_id;

```

* **How many rows are returned? Who is missing?**

**Q12. The Left Join Guarantee**

```sql
SELECT e.name, d.dept_name 
FROM Employees e 
LEFT JOIN Departments d ON e.dept_id = d.dept_id;

```

* **What is the `dept_name` next to David?**

**Q13. The Cross Join (Cartesian Product)**
If we use a `CROSS JOIN` (which matches every row in Table A to every row in Table B), we don't use an `ON` clause.

```sql
SELECT e.name, d.dept_name 
FROM Employees e CROSS JOIN Departments d;

```

* **Exactly how many rows total will this query output?**

**Q14. The Row Multiplication Effect**
Let's reverse the Left Join.

```sql
SELECT d.dept_name, e.name 
FROM Departments d 
LEFT JOIN Employees e ON d.dept_id = e.dept_id;

```

* **How many times does "Engineering" appear in the output?** **Q15. The Grandmaster Trap (`ON` vs `WHERE`)**
This is one of the hardest interview questions. Look at these two queries closely.

**Query A:**

```sql
SELECT e.name, d.dept_name 
FROM Employees e 
LEFT JOIN Departments d ON e.dept_id = d.dept_id AND d.dept_name = 'Sales';

```

**Query B:**

```sql
SELECT e.name, d.dept_name 
FROM Employees e 
LEFT JOIN Departments d ON e.dept_id = d.dept_id 
WHERE d.dept_name = 'Sales';

```

* **Which query returns ALL 4 employees?**
* **Which query accidentally turns into an INNER JOIN and only returns Charlie?**

*(Take your time with Q15. It is the core of SQL mastery. Scroll down when ready.)*

---

### **Level 3 Answers & Explanations**

**A11. Result: 3 Rows.** (Alice, Bob, Charlie).

* **Why?** David is missing because `NULL` doesn't match anything. Marketing is missing because `30` doesn't exist in the Employees table. Inner joins are strictly for matches.

**A12. Result: `NULL`.**

* **Why?** The engine guarantees David survives because he is on the Left. It looks for his department, finds nothing, and "injects" a `NULL` into the `dept_name` column for his row.

**A13. Result: 12 Rows.**

* **Why?** 4 Employees multiplied by 3 Departments (4 x 3 = 12).
* **Concept:** Alice gets paired with Engineering, Sales, and Marketing. Bob gets paired with all three, etc. Every join in SQL actually starts as a Cross Join internally, and the `ON` clause acts as a filter to cut it down!

**A14. Result: 2 Times.**

* **Why?** Departments is on the Left. The engine takes "Engineering" and scans Employees. It finds Alice. That's Row 1. It keeps scanning and finds Bob. That's Row 2.
* **The Danger:** If you join a table of 100 products to a table of 1,000 sales, your resulting table is 1,000 rows. If you aren't careful, `SUM()` operations after a join will wildly inflate your numbers because of this multiplication!

**A15. The Grandmaster Trap**

* **Query A returns ALL 4 Employees.**
* **Query B returns ONLY Charlie (Acts as an Inner Join).**

**The Deep Dive (Why this happens):**

* In **Query A**, the condition `d.dept_name = 'Sales'` is in the `ON` clause. The engine applies this *during* the join. It says: "Only attach Department data if it is Sales." So Charlie gets "Sales". Alice, Bob, and David get attached to `NULL`. Because it's a `LEFT JOIN`, Alice, Bob, and David still survive!
* In **Query B**, the join happens first. The engine builds the standard Left Join (Alice-Engineering, Bob-Engineering, Charlie-Sales, David-NULL). *Then*, the `WHERE` clause runs. The `WHERE` clause demands `dept_name = 'Sales'`. It looks at Alice (`Engineering` -> False), David (`NULL` -> Unknown). It throws them all away except Charlie.
* **The Golden Rule:** Putting a Right-table filter in the `WHERE` clause of a `LEFT JOIN` destroys the Left Join and turns it into an Inner Join.

---
