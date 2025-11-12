Excellent — that’s exactly the right next step after mastering normalization. 👌
Let’s now begin **“CTEs & Views”** using the same structured, example-driven approach (with input data, query, and output).

---

#  **Section 4: CTEs & Views**

---

## **1️ What is a CTE (Common Table Expression)?**

A **CTE (Common Table Expression)** is a *temporary result set* that you can reference within a `SELECT`, `INSERT`, `UPDATE`, or `DELETE` statement.

It behaves like a *temporary named table* that exists only during the execution of the query.

---

###  **Syntax**

```sql
WITH cte_name AS (
    SELECT column1, column2
    FROM table_name
    WHERE condition
)
SELECT * FROM cte_name WHERE column1 > 100;
```

---

###  **Characteristics of a CTE**

| Property             | Description                                                            |
| -------------------- | ---------------------------------------------------------------------- |
| **Scope**            | Exists only during the query execution.                                |
| **Temporary**        | Not stored in the database permanently.                                |
| **Readable**         | Makes complex queries cleaner and more modular.                        |
| **Recursive Option** | Can call itself to handle hierarchical data (like employee hierarchy). |

---

###  **Simple Example**

#### Input Table: `Sales`

| sale_id | customer | amount | region |
| ------- | -------- | ------ | ------ |
| 1       | Asha     | 5000   | East   |
| 2       | Raj      | 8000   | West   |
| 3       | Meena    | 3000   | East   |
| 4       | Amit     | 7000   | North  |
| 5       | Riya     | 6000   | East   |

---

#### **Query**

```sql
WITH EastSales AS (
  SELECT customer, amount
  FROM Sales
  WHERE region = 'East'
)
SELECT customer, amount
FROM EastSales
WHERE amount > 4000;
```

---

#### **Output**

| customer | amount |
| -------- | ------ |
| Asha     | 5000   |
| Riya     | 6000   |

 CTE creates a *temporary subquery* for East region and filters further.

---

## **2️⃣ CTE vs Subquery**

| Feature         | CTE                                     | Subquery                        |
| --------------- | --------------------------------------- | ------------------------------- |
| **Readability** | More readable and reusable              | Hard to read in complex nesting |
| **Reusability** | Can be used multiple times within query | Cannot be reused                |
| **Performance** | Sometimes optimized better by engine    | Re-executed each time           |
| **Scope**       | Exists only in the query                | Part of query syntax directly   |

---

### Example Comparison

####  Using Subquery:

```sql
SELECT customer, amount
FROM (
  SELECT customer, amount FROM Sales WHERE region = 'East'
) AS EastSales
WHERE amount > 4000;
```

####  Using CTE:

```sql
WITH EastSales AS (
  SELECT customer, amount FROM Sales WHERE region = 'East'
)
SELECT customer, amount FROM EastSales WHERE amount > 4000;
```

 Output is same — but the CTE version is easier to maintain and expand.

---

## **3️ Recursive CTE Example (Hierarchical Data)**

Recursive CTEs are powerful when working with **self-referencing data**, like *employee → manager relationships*.

---

### Input Table: `Employees`

| emp_id | emp_name | manager_id |
| ------ | -------- | ---------- |
| 1      | Asha     | NULL       |
| 2      | Riya     | 1          |
| 3      | Mohit    | 1          |
| 4      | Neha     | 2          |
| 5      | Arjun    | 2          |

---

### **Query – Find all employees under manager Asha**

```sql
WITH RecursiveCTE AS (
  SELECT emp_id, emp_name, manager_id
  FROM Employees
  WHERE manager_id IS NULL  -- Asha (top manager)
  
  UNION ALL
  
  SELECT e.emp_id, e.emp_name, e.manager_id
  FROM Employees e
  INNER JOIN RecursiveCTE r
  ON e.manager_id = r.emp_id
)
SELECT * FROM RecursiveCTE;
```

---

### **Output**

| emp_id | emp_name | manager_id |
| ------ | -------- | ---------- |
| 1      | Asha     | NULL       |
| 2      | Riya     | 1          |
| 3      | Mohit    | 1          |
| 4      | Neha     | 2          |
| 5      | Arjun    | 2          |

 The recursive part expands the hierarchy automatically — no manual joins needed.

---

## **4️ Multiple CTEs in One Query**

You can define **multiple CTEs** separated by commas.

---

### Example

```sql
WITH
EastSales AS (
  SELECT * FROM Sales WHERE region = 'East'
),
HighValue AS (
  SELECT * FROM EastSales WHERE amount > 4000
)
SELECT * FROM HighValue;
```

**Output**

| sale_id | customer | amount | region |
| ------- | -------- | ------ | ------ |
| 1       | Asha     | 5000   | East   |
| 5       | Riya     | 6000   | East   |

 Each CTE builds upon the previous one.

---

## **5️ What is a View?**

A **View** is a *virtual table* created using a SQL query.
Unlike a CTE, a **View is stored permanently** in the database and can be used later like a normal table.

---

###  **Syntax**

```sql
CREATE VIEW EastSalesView AS
SELECT customer, amount
FROM Sales
WHERE region = 'East';
```

Now, you can simply query:

```sql
SELECT * FROM EastSalesView WHERE amount > 4000;
```

**Output**

| customer | amount |
| -------- | ------ |
| Asha     | 5000   |
| Riya     | 6000   |

---

###  **Key Differences — CTE vs View**

| Feature          | CTE                              | View                               |
| ---------------- | -------------------------------- | ---------------------------------- |
| **Storage**      | Temporary (exists only in query) | Permanent (stored in DB)           |
| **Usage**        | Defined and used in same query   | Reusable across multiple queries   |
| **Performance**  | Not cached                       | Can be indexed (materialized view) |
| **Modification** | Cannot be updated directly       | Some views allow DML operations    |
| **Purpose**      | Query simplification             | Data abstraction/reuse             |

---

## **6️ Complex Example — Multiple Joins and CTEs**

### Input Tables:

**Customers**

| cust_id | cust_name |
| ------- | --------- |
| 1       | Asha      |
| 2       | Raj       |
| 3       | Meena     |

**Orders**

| order_id | cust_id | amount | region |
| -------- | ------- | ------ | ------ |
| 101      | 1       | 5000   | East   |
| 102      | 2       | 8000   | West   |
| 103      | 3       | 3000   | East   |
| 104      | 1       | 2000   | East   |

---

### **Query**

```sql
WITH
EastOrders AS (
  SELECT cust_id, amount
  FROM Orders
  WHERE region = 'East'
),
CustomerTotals AS (
  SELECT cust_id, SUM(amount) AS total_amount
  FROM EastOrders
  GROUP BY cust_id
)
SELECT c.cust_name, ct.total_amount
FROM Customers c
JOIN CustomerTotals ct
ON c.cust_id = ct.cust_id;
```

---

### **Output**

| cust_name | total_amount |
| --------- | ------------ |
| Asha      | 7000         |
| Meena     | 3000         |

 Step 1 (EastOrders) → filtered orders
 Step 2 (CustomerTotals) → aggregated per customer
 Step 3 → joined with Customers for names

---

## **7️ Limitations of Views**

| Limitation           | Description                                            |
| -------------------- | ------------------------------------------------------ |
| **No parameters**    | Views cannot accept parameters like stored procedures. |
| **Performance**      | Every query on a view re-runs its definition query.    |
| **DML Restrictions** | Some views (with joins, aggregates) are not updatable. |
| **Dependency risk**  | Dropping a base table breaks dependent views.          |
| **No recursion**     | Views can’t be recursive; CTEs can.                    |

---

## **8️ Practice Exercises**

### **Exercise 1: Predict output of CTE**

**Table – Orders**

| order_id | region | amount |
| -------- | ------ | ------ |
| 1        | East   | 4000   |
| 2        | West   | 5000   |
| 3        | East   | 7000   |

**Query:**

```sql
WITH East AS (
  SELECT * FROM Orders WHERE region = 'East'
),
HighValue AS (
  SELECT * FROM East WHERE amount > 5000
)
SELECT COUNT(*) AS total FROM HighValue;
```

 **Output:** `1` (only order_id 3 meets condition)

---

### **Exercise 2: Create a View**

Create a view `HighValueOrders` that shows orders with amount > 6000.
Then query all records from it.

```sql
CREATE VIEW HighValueOrders AS
SELECT * FROM Orders WHERE amount > 6000;

SELECT * FROM HighValueOrders;
```

 **Output**

| order_id | region | amount |
| -------- | ------ | ------ |
| 3        | East   | 7000   |

---

### **Exercise 3: Recursive CTE Prediction**

**Table – Managers**

| emp_id | emp_name | manager_id |
| ------ | -------- | ---------- |
| 1      | Priya    | NULL       |
| 2      | Ravi     | 1          |
| 3      | Meena    | 2          |

**Query**

```sql
WITH Chain AS (
  SELECT emp_id, emp_name, manager_id
  FROM Managers WHERE manager_id IS NULL
  UNION ALL
  SELECT m.emp_id, m.emp_name, m.manager_id
  FROM Managers m
  JOIN Chain c ON m.manager_id = c.emp_id
)
SELECT COUNT(*) FROM Chain;
```

 **Output:** `3` (Priya → Ravi → Meena chain)

---

### **Exercise 4: CTE vs View Concept Check**

| Statement                             | True/False |
| ------------------------------------- | ---------- |
| CTE is stored permanently             | X          |
| View can be indexed                   | ✓          |
| Recursive query possible with CTE     | ✓          |
| View can reference itself recursively | X          |

---

 **Summary Table**

| Concept              | CTE | View             |
| -------------------- | --- | ---------------- |
| Temporary result set | ✓   | X               |
| Stored in database   | X   | ✓               |
| Recursive possible   | ✓   | X                |
| Performance cache    | X   | ✓ (materialized) |

---
#  **Advanced Practice Problems: CTEs, Views, and Joins**

---

## **Problem 1: CTE with Aggregation & Filtering**

### **Tables**

**Orders**

| order_id | customer_id | amount | region |
| -------- | ----------- | ------ | ------ |
| 101      | 1           | 4000   | East   |
| 102      | 2           | 6000   | West   |
| 103      | 3           | 7000   | East   |
| 104      | 1           | 8000   | East   |
| 105      | 2           | 3000   | North  |

**Customers**

| customer_id | customer_name |
| ----------- | ------------- |
| 1           | Asha          |
| 2           | Raj           |
| 3           | Meena         |

---

### **Query**

```sql
WITH EastOrders AS (
  SELECT customer_id, SUM(amount) AS total_amt
  FROM Orders
  WHERE region = 'East'
  GROUP BY customer_id
),
HighValue AS (
  SELECT * FROM EastOrders WHERE total_amt > 7000
)
SELECT c.customer_name, h.total_amt
FROM Customers c
JOIN HighValue h
ON c.customer_id = h.customer_id;
```

---

### **Step-by-Step Reasoning**

1. `EastOrders`
   → Selects East region and groups by customer.

   | customer_id | total_amt           |
   | ----------- | ------------------- |
   | 1           | 4000 + 8000 = 12000 |
   | 3           | 7000                |

2. `HighValue`
   → Keeps total_amt > 7000 → only customer_id 1.

3. Join with Customers → Asha.

 **Output**

| customer_name | total_amt |
| ------------- | --------- |
| Asha          | 12000     |

---

## **Problem 2: Recursive CTE – Manager Hierarchy**

### **Table: Employees**

| emp_id | emp_name | manager_id |
| ------ | -------- | ---------- |
| 1      | Priya    | NULL       |
| 2      | Riya     | 1          |
| 3      | Neha     | 1          |
| 4      | Mohit    | 2          |
| 5      | Arjun    | 2          |
| 6      | Ravi     | 3          |

---

### **Query**

```sql
WITH Hierarchy AS (
  SELECT emp_id, emp_name, manager_id
  FROM Employees
  WHERE manager_id IS NULL
  
  UNION ALL
  
  SELECT e.emp_id, e.emp_name, e.manager_id
  FROM Employees e
  JOIN Hierarchy h
  ON e.manager_id = h.emp_id
)
SELECT COUNT(*) AS total_employees
FROM Hierarchy;
```

---

### **Reasoning**

* Start → Priya (emp_id=1)
* Direct reports of Priya → Riya (2), Neha (3)
* Reports of Riya → Mohit (4), Arjun (5)
* Reports of Neha → Ravi (6)

All connected → total 6 employees.

 **Output**

| total_employees |
| --------------- |
| 6               |

---

## **Problem 3: View-Based Query**

### **Table: Sales**

| sale_id | product | region | revenue |
| ------- | ------- | ------ | ------- |
| 1       | Laptop  | East   | 50000   |
| 2       | Laptop  | West   | 40000   |
| 3       | Phone   | East   | 25000   |
| 4       | Laptop  | East   | 30000   |
| 5       | Phone   | West   | 35000   |

---

### **View Definition**

```sql
CREATE VIEW EastSales AS
SELECT product, SUM(revenue) AS total_rev
FROM Sales
WHERE region = 'East'
GROUP BY product;
```

---

### **Query**

```sql
SELECT * FROM EastSales WHERE total_rev > 40000;
```

---

### **Reasoning**

East region sales:

| product | total_rev             |
| ------- | --------------------- |
| Laptop  | 50000 + 30000 = 80000 |
| Phone   | 25000                 |

Filter `> 40000` → only Laptop.

 **Output**

| product | total_rev |
| ------- | --------- |
| Laptop  | 80000     |

---

## **Problem 4: Multi-CTE Chain**

### **Table: Orders**

| order_id | cust_id | amount | region |
| -------- | ------- | ------ | ------ |
| 1        | 1       | 4000   | East   |
| 2        | 2       | 8000   | West   |
| 3        | 1       | 3000   | East   |
| 4        | 3       | 9000   | East   |

**Table: Customers**

| cust_id | cust_name |
| ------- | --------- |
| 1       | Asha      |
| 2       | Raj       |
| 3       | Riya      |

---

### **Query**

```sql
WITH
EastOrders AS (
  SELECT cust_id, amount FROM Orders WHERE region = 'East'
),
CustomerTotals AS (
  SELECT cust_id, SUM(amount) AS total_amt
  FROM EastOrders
  GROUP BY cust_id
),
RichEastCustomers AS (
  SELECT cust_id FROM CustomerTotals WHERE total_amt > 5000
)
SELECT c.cust_name, t.total_amt
FROM Customers c
JOIN CustomerTotals t ON c.cust_id = t.cust_id
WHERE c.cust_id IN (SELECT cust_id FROM RichEastCustomers);
```

---

### **Step-by-Step**

1. EastOrders → region = East

   | cust_id | amount |
   | ------- | ------ |
   | 1       | 4000   |
   | 1       | 3000   |
   | 3       | 9000   |

2. CustomerTotals → SUM by cust_id

   | cust_id | total_amt |
   | ------- | --------- |
   | 1       | 7000      |
   | 3       | 9000      |

3. RichEastCustomers → total_amt > 5000

   | cust_id |
   | ------- |
   | 1       |
   | 3       |

4. Join with Customers → Asha and Riya.

 **Output**

| cust_name | total_amt |
| --------- | --------- |
| Asha      | 7000      |
| Riya      | 9000      |

---

## **Problem 5: Predict Output – CTE vs View**

### **Table: Products**

| prod_id | prod_name | price | category    |
| ------- | --------- | ----- | ----------- |
| 1       | Laptop    | 60000 | Electronics |
| 2       | Mouse     | 800   | Electronics |
| 3       | Chair     | 2500  | Furniture   |

---

### **CTE Query**

```sql
WITH HighPrice AS (
  SELECT * FROM Products WHERE price > 2000
)
SELECT COUNT(*) FROM HighPrice;
```

**Output:** `2` (Laptop, Chair)

---

### **Equivalent View**

```sql
CREATE VIEW HighPrice AS
SELECT * FROM Products WHERE price > 2000;

SELECT COUNT(*) FROM HighPrice;
```

**Output:** `2`

 Output same
 But:

* CTE temporary
* View permanent and reusable

---

## **Problem 6: CTE with Join and Filter**

### **Tables**

**Departments**

| dept_id | dept_name |
| ------- | --------- |
| D1      | HR        |
| D2      | IT        |
| D3      | Sales     |

**Employees**

| emp_id | emp_name | dept_id | salary |
| ------ | -------- | ------- | ------ |
| 1      | Asha     | D1      | 40000  |
| 2      | Raj      | D2      | 60000  |
| 3      | Neha     | D3      | 30000  |
| 4      | Mohit    | D3      | 70000  |

---

### **Query**

```sql
WITH DeptAvg AS (
  SELECT dept_id, AVG(salary) AS avg_sal
  FROM Employees
  GROUP BY dept_id
),
AboveAvg AS (
  SELECT e.emp_name, e.salary, d.dept_name
  FROM Employees e
  JOIN DeptAvg a ON e.dept_id = a.dept_id
  JOIN Departments d ON e.dept_id = d.dept_id
  WHERE e.salary > a.avg_sal
)
SELECT * FROM AboveAvg;
```

---

### **Step-by-Step**

1. DeptAvg:

   | dept_id | avg_sal                 |
   | ------- | ----------------------- |
   | D1      | 40000                   |
   | D2      | 60000                   |
   | D3      | (30000+70000)/2 = 50000 |

2. AboveAvg:

   * D1: 40000 not > 40000 → none
   * D2: 60000 not > 60000 → none
   * D3: Mohit (70000 > 50000)

 **Output**

| emp_name | salary | dept_name |
| -------- | ------ | --------- |
| Mohit    | 70000  | Sales     |

---

## **Problem 7: Limitation Check — View Update**

**Table: Employees**

| emp_id | emp_name | dept | salary |
| ------ | -------- | ---- | ------ |
| 1      | Asha     | HR   | 40000  |
| 2      | Raj      | IT   | 50000  |

---

**View:**

```sql
CREATE VIEW HighSalary AS
SELECT emp_id, emp_name
FROM Employees
WHERE salary > 45000;
```

**Question:**
Can we update a record in `HighSalary` view?

```sql
UPDATE HighSalary
SET emp_name = 'Rajan'
WHERE emp_id = 2;
```

 **Answer:**
Yes — because the view is **simple (no joins, no aggregates)**.
So the change reflects in the base `Employees` table.

But if the view had **JOIN or GROUP BY**, the update would fail.

---

## **Problem 8: CTE vs View – Conceptual MCQ**

| Statement                                  | True / False |
| ------------------------------------------ | ------------ |
| A CTE can be recursive.                    | ✓ True       |
| A View can be recursive.                   | X False      |
| A View persists beyond query execution.    | ✓ True       |
| A CTE can be indexed.                      | X False      |
| A View can be indexed (materialized view). | ✓ True       |

---

##  **Summary of Key Takeaways**

| Concept                | CTE                                   | View                           |
| ---------------------- | ------------------------------------- | ------------------------------ |
| Temporary or permanent | Temporary                             | Permanent                      |
| Can be recursive       | ✓                                     | X                             |
| Exists across queries  | X                                     | ✓                              |
| Can be indexed         | X                                     | ✓ (if materialized)            |
| Typical use case       | Complex query simplification          | Reusable data abstraction      |
| Best for               | Query readability, hierarchical logic | Reusable logic, access control |

---

Would you like the **next set** to focus on **“Query Optimization & Indexing”** (with examples, visual I/O, and exercises like this)?
That’s the next logical continuation after CTEs & Views.

