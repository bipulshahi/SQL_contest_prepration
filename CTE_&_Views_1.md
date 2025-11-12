# **CTEs & Views**

---

## Part 1 — Characteristics of CTEs (Common Table Expressions)

###  What is a CTE?

A **CTE (Common Table Expression)** is a **temporary result set** defined within the execution of a single SQL query.
You can think of it as a **temporary table** that only exists for the duration of that query.

It’s defined using the **`WITH`** clause.

---

### Syntax

```sql
WITH cte_name AS (
    SELECT ...
)
SELECT ...
FROM cte_name;
```

---

### Analogy

Imagine a CTE as a **scratchpad** — you do a quick calculation or filtering on it, then use it in your final query, but once the query is done, it disappears.

---

### Example 1 – Simple CTE

**Input Table – Employees**

| emp_id | emp_name | department | salary |
| ------ | -------- | ---------- | ------ |
| 1      | Asha     | HR         | 40000  |
| 2      | Rohit    | HR         | 45000  |
| 3      | Meena    | IT         | 60000  |
| 4      | Arjun    | IT         | 70000  |
| 5      | Ravi     | Sales      | 30000  |

---

**CTE Query:**

```sql
WITH dept_avg AS (
  SELECT department, AVG(salary) AS avg_salary
  FROM Employees
  GROUP BY department
)
SELECT e.emp_name, e.department, e.salary, d.avg_salary
FROM Employees e
JOIN dept_avg d ON e.department = d.department
WHERE e.salary > d.avg_salary;
```

---

**Step-by-Step:**

1. The **CTE (`dept_avg`)** calculates average salary for each department.
2. The main query joins back and selects employees earning **above average** in their department.

---

**CTE Output (Temporary):**

| department | avg_salary |
| ---------- | ---------- |
| HR         | 42500      |
| IT         | 65000      |
| Sales      | 30000      |

**Final Query Output:**

| emp_name | department | salary | avg_salary |
| -------- | ---------- | ------ | ---------- |
| Arjun    | IT         | 70000  | 65000      |

 Only **Arjun** earns above department average → CTE makes query cleaner and readable.

---

###  Key Characteristics of CTEs

| Property    | Description                                                  |
| ----------- | ------------------------------------------------------------ |
| Scope       | Exists only during query execution                           |
| Readability | Improves clarity for nested queries                          |
| Reusability | Can reference the same logic multiple times in one query     |
| Recursion   | Can be **recursive** (e.g., for hierarchies like org charts) |

---

###  Example 2 – Recursive CTE

**Input Table – Employees**

| emp_id | emp_name | manager_id |
| ------ | -------- | ---------- |
| 1      | Asha     | NULL       |
| 2      | Rohit    | 1          |
| 3      | Meena    | 2          |
| 4      | Arjun    | 3          |

We want to find **the hierarchy from Asha downwards**.

---

**Query:**

```sql
WITH EmployeeHierarchy AS (
    SELECT emp_id, emp_name, manager_id
    FROM Employees
    WHERE emp_name = 'Asha'
  
    UNION ALL

    SELECT e.emp_id, e.emp_name, e.manager_id
    FROM Employees e
    INNER JOIN EmployeeHierarchy h
    ON e.manager_id = h.emp_id
)
SELECT * FROM EmployeeHierarchy;
```

---

**Output:**

| emp_id | emp_name | manager_id |
| ------ | -------- | ---------- |
| 1      | Asha     | NULL       |
| 2      | Rohit    | 1          |
| 3      | Meena    | 2          |
| 4      | Arjun    | 3          |

Recursive CTE traversed the hierarchy from Asha → Rohit → Meena → Arjun.

---

##  Part 2 — CTE vs Views

| Feature     | CTE                                  | View                              |
| ----------- | ------------------------------------ | --------------------------------- |
| Lifetime    | Temporary (exists only during query) | Permanent (stored in DB)          |
| Storage     | Not stored                           | Stored as a definition            |
| Performance | Created each time query runs         | May be optimized/stored once      |
| Usage       | Used once within a query             | Can be reused in multiple queries |
| Recursion   | Supports recursive queries           | Does not support recursion        |

---

###  Example

**CTE Example**

```sql
WITH HighEarners AS (
  SELECT emp_name, salary FROM Employees WHERE salary > 50000
)
SELECT * FROM HighEarners;
```

**View Example**

```sql
CREATE VIEW HighEarners AS
SELECT emp_name, salary FROM Employees WHERE salary > 50000;

SELECT * FROM HighEarners;
```

Both return the same result, but:

* CTE disappears after execution.
* View stays until dropped.

---

##  Part 3 — Complex CTE Example (Multiple CTEs)

**Input Table – Orders**

| order_id | customer | amount | region |
| -------- | -------- | ------ | ------ |
| 1        | Neha     | 1000   | East   |
| 2        | Arjun    | 1500   | West   |
| 3        | Neha     | 2000   | East   |
| 4        | Meena    | 1200   | North  |
| 5        | Arjun    | 800    | West   |

---

**Query:**

```sql
WITH
RegionTotal AS (
  SELECT region, SUM(amount) AS total_sales
  FROM Orders
  GROUP BY region
),
TopCustomers AS (
  SELECT customer, SUM(amount) AS total_spent
  FROM Orders
  GROUP BY customer
)
SELECT t.customer, t.total_spent, r.region, r.total_sales
FROM Orders o
JOIN RegionTotal r ON o.region = r.region
JOIN TopCustomers t ON o.customer = t.customer
WHERE t.total_spent > 1500
GROUP BY t.customer, r.region, t.total_spent, r.total_sales;
```

---

**Output:**

| customer | total_spent | region | total_sales |
| -------- | ----------- | ------ | ----------- |
| Neha     | 3000        | East   | 3000        |
| Arjun    | 2300        | West   | 2300        |

 Multiple CTEs can be chained to simplify logic before the main SELECT.

---

##  Part 4 — Limitations of Views

| Limitation       | Description                                                              |
| ---------------- | ------------------------------------------------------------------------ |
| 1. No Parameters | You cannot pass parameters to a view.                                    |
| 2. Performance   | Complex views (nested joins, aggregates) may perform poorly.             |
| 3. Updatability  | Not all views are updatable (especially those with joins or aggregates). |
| 4. Dependency    | Dropping base tables invalidates dependent views.                        |
| 5. Recursion     | Views do not support recursion.                                          |

---

### Example — Non-Updatable View

**Input Table – Employees**

| emp_id | emp_name | department | salary |
| ------ | -------- | ---------- | ------ |
| 1      | Asha     | HR         | 40000  |
| 2      | Rohit    | IT         | 60000  |

**View with Join**

```sql
CREATE VIEW EmpDept AS
SELECT emp_name, department, salary
FROM Employees
WHERE salary > 50000;
```

If you try:

```sql
UPDATE EmpDept SET salary = 70000 WHERE emp_name = 'Rohit';
```

 Some databases (like MySQL) will throw:

```
ERROR 1471 (HY000): The target view is not updatable
```

Because it’s **derived** (filtered, not direct table).

---

## Practice Exercises

### Q1.

Create a CTE that lists each department’s total salary and find employees who earn **more than 60% of the department total**.

**Hint:**
Use a CTE to get department total first, then join back.

---

### Q2.

Write a **recursive CTE** to display an organization chart starting from the CEO.

---

### Q3.

Create a **View** for “TopPerformers” that shows employees whose salary is above the company average.
Then try to **update salary** through that view — observe if your DBMS allows it.

---

### Q4.

Explain why using a **CTE** is better than subqueries for readability in large queries.

---

### Q5.

Identify the correct difference:

> A CTE is ___ (temporary/permanent) and a View is ___ (temporary/permanent).

**Answer:** Temporary, Permanent
