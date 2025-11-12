# **Indexing & Query Optimization**

We’ll go step by step — from the basics of indexing to multi-column indexes, performance analysis, and practical optimization examples — with **input tables, query outputs, and practice exercises**.

---

## Part 1 — What is an Index?

### Concept:

An **index** in SQL is like an index in a book — instead of scanning every page to find a topic, you jump directly using the index.

Similarly, in databases, an **index** helps the database engine quickly locate rows without scanning the entire table.

---

###  Example: Table Without Index

**Input Table – Employees**

| emp_id | emp_name | department | salary |
| ------ | -------- | ---------- | ------ |
| 1      | Asha     | HR         | 40000  |
| 2      | Rohit    | IT         | 70000  |
| 3      | Meena    | IT         | 80000  |
| 4      | Arjun    | Sales      | 35000  |
| 5      | Ravi     | Sales      | 30000  |

Suppose this table has **10 million rows**.

---

**Query:**

```sql
SELECT * FROM Employees WHERE emp_name = 'Meena';
```

 Without an index, the DBMS performs a **full table scan** — checking every row sequentially.
 **Slow for large tables.**

---

**Create an index:**

```sql
CREATE INDEX idx_emp_name ON Employees(emp_name);
```

Now the database can **jump directly** to where `emp_name = 'Meena'` is stored.
 **Much faster lookup.**

---

###  Key Takeaway:

| Operation        | With Index                                      | Without Index |
| ---------------- | ----------------------------------------------- | ------------- |
| Search by column | Fast                                            | Slow          |
| Insert/Update    | Slightly slower                                 | Faster        |
| Storage usage    | Increases                                       | Less          |
| Best used for    | Columns used in WHERE, JOIN, ORDER BY, GROUP BY | -             |

---

##  Part 2 — Clustered vs Non-Clustered Index

###  Clustered Index

* Sorts and stores data **physically** in table order.
* Each table can have **only one** clustered index (usually on Primary Key).

**Example:**

```sql
CREATE CLUSTERED INDEX idx_emp_id ON Employees(emp_id);
```

Now rows are stored **physically sorted by emp_id**.

**Analogy:**
 Like sorting books alphabetically on a shelf — only one arrangement possible.

---

###  Non-Clustered Index

* Creates a **separate structure** that stores pointers to actual rows.
* You can create **multiple** non-clustered indexes.

**Example:**

```sql
CREATE NONCLUSTERED INDEX idx_department ON Employees(department);
```

This builds a separate lookup for departments.

**Analogy:**
 Like an index page at the end of a book — it lists topics and page numbers (pointers).

---

###  Example Comparison

| Type          | Data physically sorted? | How many allowed? | Speed (SELECT)          | Speed (INSERT)  |
| ------------- | ----------------------- | ----------------- | ----------------------- | --------------- |
| Clustered     |  Yes                   | 1                 | Fast                    | Slightly slower |
| Non-Clustered |  No                    | Many              | Fast (depends on usage) | Slightly slower |

---

##  Part 3 — Indexing on Multiple Columns

Sometimes queries filter by **more than one column**.
To optimize such queries, you can create a **composite (multi-column) index**.

---

### Example

**Input Table – Orders**

| order_id | customer_id | order_date | total_amount |
| -------- | ----------- | ---------- | ------------ |
| 1        | 101         | 2025-10-01 | 500          |
| 2        | 102         | 2025-10-02 | 1000         |
| 3        | 101         | 2025-10-03 | 200          |
| 4        | 103         | 2025-10-03 | 700          |
| 5        | 101         | 2025-10-05 | 1200         |

---

**Query:**

```sql
SELECT * FROM Orders
WHERE customer_id = 101 AND order_date = '2025-10-03';
```

If we index only `customer_id`, the DB must still check all orders for that customer.
But with a composite index:

```sql
CREATE INDEX idx_cust_date ON Orders(customer_id, order_date);
```

The database directly jumps to the exact row range.

---

** Rule of Thumb:**
The order of columns in a composite index matters.

* `(customer_id, order_date)` is **different** from `(order_date, customer_id)`.
* The first column is the **leading key** and must match for the index to be used efficiently.

---

## Part 4 — Drawbacks of Excessive Indexing

| Drawback                        | Description                                                            |
| ------------------------------- | ---------------------------------------------------------------------- |
| 1️⃣ Slower INSERT/UPDATE/DELETE | Each index must also be updated → slows down write operations          |
| 2️⃣ More Disk Space             | Indexes take up additional storage                                     |
| 3️⃣ Maintenance Overhead        | Too many indexes confuse the optimizer and increase rebuild costs      |
| 4️⃣ Diminishing Returns         | Not all indexes improve performance — some queries may even get slower |

** Best Practice:**
Create indexes **only on frequently used columns** in:

* WHERE clause
* JOIN conditions
* GROUP BY / ORDER BY

---

##  Part 5 — Query Optimization Examples

### Example 1: Without Optimization

```sql
SELECT * FROM Orders
WHERE YEAR(order_date) = 2025 AND total_amount > 500;
```

Problem:
Using a function (`YEAR(order_date)`) prevents index usage.

Optimized version:

```sql
SELECT * FROM Orders
WHERE order_date >= '2025-01-01' AND order_date < '2026-01-01'
  AND total_amount > 500;
```

Now indexes on `order_date` can be used.

---

### Example 2: Avoid SELECT *

Instead of:

```sql
SELECT * FROM Employees WHERE department = 'IT';
```

 Use only needed columns:

```sql
SELECT emp_id, emp_name FROM Employees WHERE department = 'IT';
```

Smaller result set → faster response.

---

### Example 3: Use EXPLAIN / EXPLAIN ANALYZE

Before optimizing, **inspect your query plan:**

```sql
EXPLAIN SELECT * FROM Orders WHERE customer_id = 101;
```

This shows whether indexes are used or not.
If you see `ALL` in the output → full table scan (index not used).

---

##  Part 6 — Practice Exercises

---

### Q1.

Create an index on the `department` column of the `Employees` table.
Then compare performance of this query before and after indexing:

```sql
SELECT * FROM Employees WHERE department = 'IT';
```

 What changes do you observe using `EXPLAIN`?

---

### Q2.

Write a query to create a **composite index** on `(customer_id, order_date)` and use it to fetch:

```sql
SELECT * FROM Orders WHERE customer_id = 101 AND order_date = '2025-10-03';
```

Why is the **column order** in the index important?

---

### Q3.

Explain what happens if you create too many indexes on a table that undergoes frequent updates.

---

### Q4.

Identify which query can **use an index** efficiently:

1. `WHERE YEAR(order_date) = 2025`
2. `WHERE order_date BETWEEN '2025-01-01' AND '2025-12-31'`

 **Answer:** Query 2 (function-free condition allows index usage)

---

### Q5.

Given a table `Sales(region, product, amount)`,
create an index that optimizes:

```sql
SELECT * FROM Sales WHERE region = 'East' AND product = 'Laptop';
```

 Solution:

```sql
CREATE INDEX idx_region_product ON Sales(region, product);
```

---

##  Summary Table

| Concept             | Key Takeaway                                            |
| ------------------- | ------------------------------------------------------- |
| Index               | Speeds up searches using internal structures            |
| Clustered Index     | Sorts data physically; one per table                    |
| Non-Clustered Index | Stores pointer references; many per table               |
| Composite Index     | Multi-column search optimization                        |
| Drawback            | Slower writes, more storage                             |
| Optimization        | Avoid `SELECT *`, avoid functions in WHERE, use EXPLAIN |

---
