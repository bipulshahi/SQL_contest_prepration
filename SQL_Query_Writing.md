**SQL Query Writing**
**GROUP BY queries** and **error identification**.

---

# **3. SQL Query Writing & Output Prediction**

---

##  Part 1 — Writing and Predicting GROUP BY Queries

###  Concept

The `GROUP BY` clause groups rows that have the same values in specified columns and lets you apply **aggregate functions** (like `SUM`, `COUNT`, `AVG`, `MIN`, `MAX`) to each group.

---

###  Example 1 — Basic GROUP BY

**Input Table – Sales**

| sale_id | region | product | amount |
| ------- | ------ | ------- | ------ |
| 1       | East   | Pen     | 200    |
| 2       | East   | Pencil  | 150    |
| 3       | West   | Pen     | 100    |
| 4       | West   | Pencil  | 200    |
| 5       | East   | Pen     | 300    |

---

**Query 1:**

```sql
SELECT region, SUM(amount) AS total_sales
FROM Sales
GROUP BY region;
```

**Step-by-Step**

1. The rows are grouped by `region`.
2. `SUM(amount)` is applied within each group.

**Output:**

| region | total_sales |
| ------ | ----------- |
| East   | 650         |
| West   | 300         |

 Each region’s sales aggregated correctly.

---

###  Example 2 — GROUP BY Multiple Columns

```sql
SELECT region, product, SUM(amount) AS total_sales
FROM Sales
GROUP BY region, product;
```

**Output:**

| region | product | total_sales |
| ------ | ------- | ----------- |
| East   | Pen     | 500         |
| East   | Pencil  | 150         |
| West   | Pen     | 100         |
| West   | Pencil  | 200         |

 Separate totals for each (region, product) pair.

---

###  Example 3 — Using Aggregate Functions

```sql
SELECT region,
       COUNT(*) AS order_count,
       AVG(amount) AS avg_sale,
       MAX(amount) AS max_sale
FROM Sales
GROUP BY region;
```

**Output:**

| region | order_count | avg_sale | max_sale |
| ------ | ----------- | -------- | -------- |
| East   | 3           | 216.67   | 300      |
| West   | 2           | 150.00   | 200      |

 Different aggregates computed per region.

---

##  Part 2 — Error Identification in GROUP BY Queries

###  Common Error 1: Column Not in GROUP BY or Aggregate

**Incorrect Query**

```sql
SELECT region, product, SUM(amount)
FROM Sales
GROUP BY region;
```

**Error**

```
ERROR: column 'product' must appear in the GROUP BY clause or be used in an aggregate function
```

**Fix**
Add `product` to `GROUP BY` or aggregate it:

```sql
SELECT region, MAX(product), SUM(amount)
FROM Sales
GROUP BY region;
```

---

###  Common Error 2: Alias Used in GROUP BY

```sql
SELECT region AS area, SUM(amount)
FROM Sales
GROUP BY area;
```

**Error**

```
ERROR: column "area" does not exist
```

**Fix**
Use the original column name:

```sql
GROUP BY region;
```

---

###  Correct Example (Contest-Style Prediction)

**Input Table – Orders**

| order_id | customer | region | amount |
| -------- | -------- | ------ | ------ |
| 1        | Neha     | East   | 500    |
| 2        | Arjun    | West   | 800    |
| 3        | Neha     | East   | 200    |
| 4        | Meena    | West   | 700    |
| 5        | Arjun    | West   | 300    |

**Query:**

```sql
SELECT region, COUNT(DISTINCT customer) AS cust_count, SUM(amount) AS total
FROM Orders
GROUP BY region
ORDER BY total DESC;
```

**Output:**

| region | cust_count | total |
| ------ | ---------- | ----- |
| West   | 2          | 1800  |
| East   | 1          | 700   |

 Predicted output matches aggregated totals.

---

##  Part 3 — GROUP BY with HAVING (Multiple Conditions)

###  Concept

`HAVING` filters aggregated groups — like a `WHERE` for groups.

---

###  Example – Simple HAVING

```sql
SELECT region, SUM(amount) AS total_sales
FROM Sales
GROUP BY region
HAVING SUM(amount) > 400;
```

**Output:**

| region | total_sales |
| ------ | ----------- |
| East   | 650         |

 Only East has total sales > 400.

---

###  Example – Multiple HAVING Conditions

```sql
SELECT region, COUNT(*) AS orders, SUM(amount) AS total
FROM Sales
GROUP BY region
HAVING COUNT(*) >= 2 AND SUM(amount) > 300;
```

**Output:**

| region | orders | total |
| ------ | ------ | ----- |
| East   | 3      | 650   |
| West   | 2      | 300   |

 Both conditions evaluated **after grouping**.

---

###  Example – Using WHERE and HAVING Together

```sql
SELECT region, SUM(amount) AS total
FROM Sales
WHERE amount > 100
GROUP BY region
HAVING SUM(amount) > 400;
```

**Step-by-Step:**

1. `WHERE amount > 100` filters rows **before grouping**.
2. `HAVING SUM(amount) > 400` filters **group totals**.

**Output:**

| region | total |
| ------ | ----- |
| East   | 650   |

 WHERE → pre-filter, HAVING → post-aggregate filter.

---

##  Practice Exercises

---

### Q1.

Using the `Sales` table, find each product’s total sales and only display those whose total > 200.

 **Expected Output:**

| product | total_sales |
| ------- | ----------- |
| Pen     | 600         |
| Pencil  | 350         |

---

### Q2.

Identify the error:

```sql
SELECT region, product, SUM(amount)
FROM Sales
HAVING SUM(amount) > 100;
```

 **Answer:** `GROUP BY` missing — you can’t use `HAVING` without `GROUP BY`.

---

### Q3.

Predict the output:

```sql
SELECT region, COUNT(*) AS cnt
FROM Sales
WHERE amount > 150
GROUP BY region
HAVING COUNT(*) > 1;
```

**Expected:**

| region | cnt |
| ------ | --- |
| East   | 2   |

---

### Q4.

What’s the difference between `WHERE` and `HAVING`?

| Clause | Filters When    | Works On |
| ------ | --------------- | -------- |
| WHERE  | Before grouping | Rows     |
| HAVING | After grouping  | Groups   |

---
