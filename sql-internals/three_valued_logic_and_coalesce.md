
### **The "Playground" Data**

Imagine a simple table called `Employees`.

* **Note:** `NULL` means the field is empty (e.g., The CEO has no manager).
* **Note:** `Bob` has a salary of `0` (maybe an unpaid intern), while `David` has `NULL` (salary not yet decided).

Table: Employees

| id | name | salary | manager_id |
| --- | --- | --- | --- |
| 1 | Alice | 1000 | NULL |
| 2 | Bob | 0 | 1 |
| 3 | Charlie | 2000 | 1 |
| 4 | David | NULL | 1 |
| 5 | Eve | 3000 | 2 |

---

### **Concept 1: The "Three-Valued Logic" of NULL**

In SQL, `NULL` does **not** equal Zero. It does **not** equal an empty string `""`.
**`NULL` means "Unknown".**

Think of `NULL` as a "Mystery Box."

* **Question:** Is `Bob`'s salary (0) equal to `David`'s salary (Mystery Box)?
* **Logic:** `0 = NULL`
* **Result:** **UNKNOWN**. (Maybe the box has 0, maybe 1000. We don't know).


* **Question:** Is `Alice`'s salary (1000) **NOT** equal to `David`'s salary?
* **Logic:** `1000 != NULL`
* **Result:** **UNKNOWN**. (We can't say for sure it's different because we don't know what's in the box).



**The Golden Rule:**
The `WHERE` clause **only keeps rows that are TRUE**.
It throws away **FALSE** *and* **UNKNOWN**.

Therefore, `WHERE salary != 0` will **hide** David (NULL salary).

---

### **Concept 2: COALESCE (The "Backup Plan")**

`COALESCE` is just a fancy word for "First non-null value."
It takes a list of values and returns the first one that isn't a Mystery Box.

* `COALESCE(NULL, 5)` -> **5**
* `COALESCE(NULL, NULL, 10)` -> **10**
* `COALESCE(0, 5)` -> **0** (Because 0 is a real value, not a NULL).

**Why use it?**
To fix the logic issue above. If you want to treat `NULL` as `0`, you write:
`COALESCE(salary, 0)`.

---

### **Level 1 Challenge**

Use the **Employees** table above.

**Q1. The Anti-Zero Filter**
You want to find everyone whose salary is **not** zero.
Query: `SELECT name FROM Employees WHERE salary != 0;`

* **Which names are returned?**

**Q2. The "Equal to NULL" Trap**
You want to find the CEO (who has no manager).
Query: `SELECT name FROM Employees WHERE manager_id = NULL;`

* **Which names are returned?**

**Q3. The Sorting Mystery**
Query: `SELECT name FROM Employees ORDER BY salary ASC;`

* **Where does David (NULL salary) appear? Top or Bottom?** (Assume Standard SQL).

**Q4. The Math Trap**
You want to give everyone a $100 bonus.
Query: `SELECT name, salary + 100 as new_salary FROM Employees;`

* **What is David's `new_salary` value?**

**Q5. The `NOT IN` Danger Zone (The Interview Question)**
You want to find employees who do **not** report to Alice (ID 1).
Query: `SELECT name FROM Employees WHERE manager_id NOT IN (1);`

* **Which names are returned?**


---

### **Level 1 Answers & Explanations**

**A1. Result:** **Alice, Charlie, Eve** (3 rows).

* **Why?**
* Bob: `0 != 0` is **False**. (Dropped).
* David: `NULL != 0` is **Unknown**. (Dropped).


* **Correction:** If you wanted to include David, you should have used: `WHERE COALESCE(salary, 0) != 0`.

**A2. Result:** **Zero Rows** (Empty).

* **Why?** `NULL = NULL` is **Unknown**. You cannot use math operators on NULL.
* **Correction:** You must use the special operator: `WHERE manager_id IS NULL`.

**A3. Result:** **It depends on the Database**, but usually **Bottom**.

* **Why?** Most databases (Postgres, Oracle) treat NULL as "Infinity" or the largest possible value by default. SQL Server treats them as the lowest possible value (Top).
* **Pro Tip:** You can force this using `ORDER BY salary ASC NULLS FIRST` or `NULLS LAST`.

**A4. Result:** **NULL**.

* **Why?** `Unknown + 100` is still `Unknown`. Math with a NULL always results in NULL.
* **Correction:** `COALESCE(salary, 0) + 100`.

**A5. Result:** **Eve**.

* **Why?**
* Alice (Manager NULL): `NULL != 1` -> **Unknown**. (Dropped).
* Bob (Manager 1): `1 != 1` -> **False**. (Dropped).
* David (Manager 1): `1 != 1` -> **False**. (Dropped).
* Eve (Manager 2): `2 != 1` -> **True**. (Kept).


* **Note:** This is tricky! The CEO (Alice) is excluded because her manager is NULL, and the database doesn't know if her mystery manager is "1" or not, so it plays it safe and hides her.

---
