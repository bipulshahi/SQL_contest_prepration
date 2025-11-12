**(Normalization)** — starting with **1NF**

---

# **2. Data Normalization & Integrity (1NF with examples)**

---

##  1. Why Normalization?

Let’s start with a quick analogy.

 **Imagine a storeroom** where you dump all kinds of items (books, clothes, tools) in one box — messy and hard to update.
Normalization is like **organizing items into labelled drawers** — easier to find, update, and maintain.

In databases, normalization organizes data into multiple related tables to:

* Reduce duplication
* Avoid anomalies
* Maintain integrity

---

##  Part 1: First Normal Form (1NF)

A table is in **1NF** if:

1. Every column contains **atomic (single)** values.
2. Each column holds values of **a single data type**.
3. Each record is **uniquely identifiable** (usually with a primary key).

---

###  **Example: Not in 1NF**

**Input Table – Students**

| student_id | name  | phones     |
| ---------- | ----- | ---------- |
| 1          | Asha  | 9876, 7654 |
| 2          | Rohit | 9988       |

 **Problem:**
Column `phones` contains multiple values (not atomic).

---

###  **Convert to 1NF**

Split repeated values into separate rows.

**Output Table 1 – Students**

| student_id | name  |
| ---------- | ----- |
| 1          | Asha  |
| 2          | Rohit |

**Output Table 2 – StudentPhones**

| student_id | phone |
| ---------- | ----- |
| 1          | 9876  |
| 1          | 7654  |
| 2          | 9988  |

Now:

* Each column holds one value per cell.
* Relationships handled through `student_id`.

---

###  SQL Hands-On

**Unnormalized table**

```sql
CREATE TABLE Students_Unnormalized (
  student_id INT,
  name VARCHAR(50),
  phones VARCHAR(50)
);

INSERT INTO Students_Unnormalized VALUES
(1, 'Asha', '9876,7654'),
(2, 'Rohit', '9988');
```

**Normalized version**

```sql
CREATE TABLE Students (
  student_id INT PRIMARY KEY,
  name VARCHAR(50)
);

CREATE TABLE StudentPhones (
  student_id INT,
  phone VARCHAR(15),
  FOREIGN KEY (student_id) REFERENCES Students(student_id)
);

INSERT INTO Students VALUES
(1, 'Asha'),
(2, 'Rohit');

INSERT INTO StudentPhones VALUES
(1, '9876'),
(1, '7654'),
(2, '9988');
```

**Query result:**

```sql
SELECT s.name, p.phone
FROM Students s
JOIN StudentPhones p ON s.student_id = p.student_id;
```

**Output**

| name  | phone |
| ----- | ----- |
| Asha  | 9876  |
| Asha  | 7654  |
| Rohit | 9988  |

 Each cell atomic → table now in **1NF**.

---

##  3. Data Anomaly Example (Before Normalization)

**Input Table – EmployeeRecords**

| emp_id | emp_name | dept_name | manager_name |
| ------ | -------- | --------- | ------------ |
| 101    | Ravi     | Sales     | Priya        |
| 102    | Meena    | Sales     | Priya        |
| 103    | Raj      | IT        | Arjun        |

###  Update Anomaly:

If “Priya” becomes “Priya Sharma”, we must update **two rows** → risk of inconsistency.

###  Delete Anomaly:

If we delete both sales employees, we lose info that “Sales” department exists.

---

###  Fix by Normalization

**Output Table 1 – Departments**

| dept_id | dept_name | manager_name |
| ------- | --------- | ------------ |
| D01     | Sales     | Priya        |
| D02     | IT        | Arjun        |

**Output Table 2 – Employees**

| emp_id | emp_name | dept_id |
| ------ | -------- | ------- |
| 101    | Ravi     | D01     |
| 102    | Meena    | D01     |
| 103    | Raj      | D02     |

Now:

* Department info appears once.
* No redundant manager name.

---

##  4. 1NF Practice Question (with output)

**Given Table**

| order_id | customer | items         |
| -------- | -------- | ------------- |
| 1        | Neha     | Pen,Pencil    |
| 2        | Arjun    | Notebook      |
| 3        | Neha     | Eraser,Marker |

**Question:** Convert this table into 1NF and show result.

**Solution Output**

**Orders Table**

| order_id | customer |
| -------- | -------- |
| 1        | Neha     |
| 2        | Arjun    |
| 3        | Neha     |

**OrderItems Table**

| order_id | item     |
| -------- | -------- |
| 1        | Pen      |
| 1        | Pencil   |
| 2        | Notebook |
| 3        | Eraser   |
| 3        | Marker   |

 Each value atomic → **1NF achieved**.

---

##  5. Hands-On Verification (Try This)

```sql
CREATE TABLE Orders_Unnormalized (
  order_id INT,
  customer VARCHAR(50),
  items VARCHAR(100)
);

INSERT INTO Orders_Unnormalized VALUES
(1, 'Neha', 'Pen,Pencil'),
(2, 'Arjun', 'Notebook'),
(3, 'Neha', 'Eraser,Marker');
```

 Try to **query items individually** — not possible because data is mixed.
Then create `Orders` and `OrderItems` as above, and query:

```sql
SELECT o.customer, i.item
FROM Orders o
JOIN OrderItems i ON o.order_id = i.order_id;
```

**Output:**

| customer | item     |
| -------- | -------- |
| Neha     | Pen      |
| Neha     | Pencil   |
| Arjun    | Notebook |
| Neha     | Eraser   |
| Neha     | Marker   |

---

 **Summary:**

| Rule                     | Example Fix                              |
| ------------------------ | ---------------------------------------- |
| Atomic values only       | Split list columns (e.g., phones, items) |
| One data type per column | Don’t mix numbers and text               |
| Unique identifier        | Add primary key                          |

---

Next up, we’ll move to **2NF**:
---

# **Part 2: Second Normal Form (2NF)**

---

##  1. What is 2NF?

A table is in **Second Normal Form (2NF)** if:

1. It is already in **1NF** (no repeating groups, atomic columns).
2. **No partial dependency exists** — meaning **no non-key attribute depends on part of a composite primary key**.

 **In simple words:**
If a table has a **composite key (multiple columns as the primary key)**, every non-key column must depend on **the entire key**, not just one part of it.

---

##  2. Why 2NF is Needed?

If you have a table where some columns depend only on part of a composite key,
you’ll end up repeating data unnecessarily and risk inconsistencies during updates.

---

##  3. Example: Not in 2NF

### **Input Table – OrderDetails**

| order_id | product_id | product_name | order_date | quantity |
| -------- | ---------- | ------------ | ---------- | -------- |
| 101      | P01        | Keyboard     | 2024-11-01 | 2        |
| 101      | P02        | Mouse        | 2024-11-01 | 1        |
| 102      | P01        | Keyboard     | 2024-11-05 | 3        |

**Primary key:** `(order_id, product_id)`
**Non-key attributes:** `product_name`, `order_date`, `quantity`

---

###  **Dependencies:**

* `order_id → order_date`
  (order_date depends only on part of the key → partial dependency (not accptable))
* `product_id → product_name`
  (depends on product_id only → partial dependency (not accptable))
* `order_id, product_id → quantity` (accptable) (depends on both → fine)

So this table is **not in 2NF**.

---

###  **Problem**

* If we update a product’s name (say, “Mouse” → “Optical Mouse”), we must update it in many rows.
* If we delete all orders for a product, we lose product info.

---

##  4. Convert to 2NF

### Step 1: Identify independent entities

→ Orders, Products, and OrderDetails.

---

###  **Normalized Tables**

**Table 1 – Orders**

| order_id | order_date |
| -------- | ---------- |
| 101      | 2024-11-01 |
| 102      | 2024-11-05 |

**Table 2 – Products**

| product_id | product_name |
| ---------- | ------------ |
| P01        | Keyboard     |
| P02        | Mouse        |

**Table 3 – OrderDetails**

| order_id | product_id | quantity |
| -------- | ---------- | -------- |
| 101      | P01        | 2        |
| 101      | P02        | 1        |
| 102      | P01        | 3        |

 Now each non-key column depends on **the whole key**, not part of it → **2NF achieved**.

---

###  **SQL Implementation**

```sql
-- Unnormalized
CREATE TABLE OrderDetails_Unnormalized (
  order_id INT,
  product_id VARCHAR(10),
  product_name VARCHAR(50),
  order_date DATE,
  quantity INT,
  PRIMARY KEY(order_id, product_id)
);

INSERT INTO OrderDetails_Unnormalized VALUES
(101, 'P01', 'Keyboard', '2024-11-01', 2),
(101, 'P02', 'Mouse', '2024-11-01', 1),
(102, 'P01', 'Keyboard', '2024-11-05', 3);
```

---

**Normalized version (2NF):**

```sql
CREATE TABLE Orders (
  order_id INT PRIMARY KEY,
  order_date DATE
);

CREATE TABLE Products (
  product_id VARCHAR(10) PRIMARY KEY,
  product_name VARCHAR(50)
);

CREATE TABLE OrderDetails (
  order_id INT,
  product_id VARCHAR(10),
  quantity INT,
  PRIMARY KEY (order_id, product_id),
  FOREIGN KEY (order_id) REFERENCES Orders(order_id),
  FOREIGN KEY (product_id) REFERENCES Products(product_id)
);
```

**Insert data**

```sql
INSERT INTO Orders VALUES (101, '2024-11-01'), (102, '2024-11-05');
INSERT INTO Products VALUES ('P01', 'Keyboard'), ('P02', 'Mouse');
INSERT INTO OrderDetails VALUES (101, 'P01', 2), (101, 'P02', 1), (102, 'P01', 3);
```

---

**Query to verify**

```sql
SELECT o.order_id, o.order_date, p.product_name, d.quantity
FROM Orders o
JOIN OrderDetails d ON o.order_id = d.order_id
JOIN Products p ON p.product_id = d.product_id;
```

**Output**

| order_id | order_date | product_name | quantity |
| -------- | ---------- | ------------ | -------- |
| 101      | 2024-11-01 | Keyboard     | 2        |
| 101      | 2024-11-01 | Mouse        | 1        |
| 102      | 2024-11-05 | Keyboard     | 3        |

 Same data, but now **without redundancy or anomalies**.

---

##  5. Another Example (Simpler)

### **Input Table – StudentCourses**

| student_id | course_id | student_name | course_name  |
| ---------- | --------- | ------------ | ------------ |
| 1          | C1        | Ravi         | SQL Basics   |
| 1          | C2        | Ravi         | Python Intro |
| 2          | C1        | Meena        | SQL Basics   |

**Composite key:** (student_id, course_id)

Here:

* `student_name` depends on `student_id` → partial dependency.
* `course_name` depends on `course_id` → partial dependency.

---

###  **Normalized (2NF) Tables**

**Students**

| student_id | student_name |
| ---------- | ------------ |
| 1          | Ravi         |
| 2          | Meena        |

**Courses**

| course_id | course_name  |
| --------- | ------------ |
| C1        | SQL Basics   |
| C2        | Python Intro |

**Enrollments**

| student_id | course_id |
| ---------- | --------- |
| 1          | C1        |
| 1          | C2        |
| 2          | C1        |

 Now each table stores independent data, and relationships are maintained cleanly.

---

##  6. Practice Exercises (With Solutions)

### **Exercise 1: Identify 2NF Violation**

Table:

| order_id | product_id | order_date | product_price | quantity |
| -------- | ---------- | ---------- | ------------- | -------- |

Primary key = (order_id, product_id)

Which columns violate 2NF?

 **Solution:**

* `order_date` depends on `order_id` only → partial dependency.
* `product_price` depends on `product_id` only → partial dependency.
  Hence, **violates 2NF**.

---

### **Exercise 2: Predict the Normalized Tables**

Input Table – `LibraryRecords`

| student_id | book_id | student_name | book_title | issue_date |
| ---------- | ------- | ------------ | ---------- | ---------- |

 **Answer:**

**Students(student_id, student_name)**
**Books(book_id, book_title)**
**Issues(student_id, book_id, issue_date)**

All non-key attributes depend on full key → **2NF achieved**.

---

### **Exercise 3: Fill in the output**

Input:

| emp_id | project_id | emp_name | project_name   | hours |
| ------ | ---------- | -------- | -------------- | ----- |
| 1      | P1         | Ravi     | Billing System | 40    |
| 1      | P2         | Ravi     | HR System      | 30    |
| 2      | P1         | Meena    | Billing System | 35    |

Normalize to 2NF and show tables.

 **Solution Output**

**Employees**

| emp_id | emp_name |
| ------ | -------- |
| 1      | Ravi     |
| 2      | Meena    |

**Projects**

| project_id | project_name   |
| ---------- | -------------- |
| P1         | Billing System |
| P2         | HR System      |

**EmpProjects**

| emp_id | project_id | hours |
| ------ | ---------- | ----- |
| 1      | P1         | 40    |
| 1      | P2         | 30    |
| 2      | P1         | 35    |

---

 **Summary:**

| Concept            | Description                                           |
| ------------------ | ----------------------------------------------------- |
| Partial Dependency | Non-key attribute depends on part of composite key    |
| Rule               | Every non-key attribute must depend on **entire key** |
| Result             | Removes redundancy, prevents anomalies                |



# **Part 3: Third Normal Form (3NF)**

---

##  1. What is 3NF?

A table is in **Third Normal Form (3NF)** if:

1. It is already in **2NF**, and
2. **No transitive dependency** exists —
   i.e., **non-key attributes must depend only on the key**, not on other non-key attributes.

---

###  Quick Definition:

If **A → B** and **B → C**, then **A → C** is a *transitive dependency*.
In 3NF, such transitive dependencies must be removed.

---

###  Real-Life Analogy:

Suppose you record both *City* and *Pincode* in an address table.

* City → Pincode
* CustomerID → City
  Hence CustomerID → Pincode (transitive dependency).
  If the pincode for a city changes, you’ll have to update many rows — that’s what 3NF fixes.

---

##  2. Example — Not in 3NF

### **Input Table – Employees**

| emp_id | emp_name | dept_id | dept_name | manager_name |
| ------ | -------- | ------- | --------- | ------------ |
| 101    | Ravi     | D1      | Sales     | Priya        |
| 102    | Meena    | D1      | Sales     | Priya        |
| 103    | Arjun    | D2      | HR        | Rohan        |

**Primary key:** emp_id
**Functional dependencies:**

* emp_id → emp_name, dept_id, dept_name, manager_name
* dept_id → dept_name, manager_name

 `emp_id → dept_id` and `dept_id → dept_name`
 Therefore `emp_id → dept_name` is a **transitive dependency**.

---

###  **Problem:**

* Changing a department’s manager means updating multiple employee rows.
* Inconsistent data possible (e.g., two managers for same dept_id).

---

##  3. Convert to 3NF

### Step 1 — Identify dependent entities:

* Employee info
* Department info

### Step 2 — Separate them.

---

###  **Output Tables**

**Table 1 – Departments**

| dept_id | dept_name | manager_name |
| ------- | --------- | ------------ |
| D1      | Sales     | Priya        |
| D2      | HR        | Rohan        |

**Table 2 – Employees**

| emp_id | emp_name | dept_id |
| ------ | -------- | ------- |
| 101    | Ravi     | D1      |
| 102    | Meena    | D1      |
| 103    | Arjun    | D2      |

 Each non-key attribute now depends **only on the key** (emp_id).
 Department details are stored once → **3NF achieved**.

---

###  **SQL Implementation**

```sql
-- Before Normalization
CREATE TABLE Employees_Unnormalized (
  emp_id INT PRIMARY KEY,
  emp_name VARCHAR(50),
  dept_id VARCHAR(10),
  dept_name VARCHAR(50),
  manager_name VARCHAR(50)
);

INSERT INTO Employees_Unnormalized VALUES
(101, 'Ravi', 'D1', 'Sales', 'Priya'),
(102, 'Meena', 'D1', 'Sales', 'Priya'),
(103, 'Arjun', 'D2', 'HR', 'Rohan');
```

---

**Normalized Version (3NF)**

```sql
CREATE TABLE Departments (
  dept_id VARCHAR(10) PRIMARY KEY,
  dept_name VARCHAR(50),
  manager_name VARCHAR(50)
);

CREATE TABLE Employees (
  emp_id INT PRIMARY KEY,
  emp_name VARCHAR(50),
  dept_id VARCHAR(10),
  FOREIGN KEY (dept_id) REFERENCES Departments(dept_id)
);

INSERT INTO Departments VALUES
('D1', 'Sales', 'Priya'),
('D2', 'HR', 'Rohan');

INSERT INTO Employees VALUES
(101, 'Ravi', 'D1'),
(102, 'Meena', 'D1'),
(103, 'Arjun', 'D2');
```

---

**Query to verify**

```sql
SELECT e.emp_id, e.emp_name, d.dept_name, d.manager_name
FROM Employees e
JOIN Departments d ON e.dept_id = d.dept_id;
```

**Output**

| emp_id | emp_name | dept_name | manager_name |
| ------ | -------- | --------- | ------------ |
| 101    | Ravi     | Sales     | Priya        |
| 102    | Meena    | Sales     | Priya        |
| 103    | Arjun    | HR        | Rohan        |

 Data is same but no redundancy.
 Easy to update manager info in one place.

---

##  4. Another Example — Students & Cities

### **Input Table**

| student_id | student_name | city  | pincode |
| ---------- | ------------ | ----- | ------- |
| 1          | Asha         | Pune  | 411001  |
| 2          | Rohit        | Delhi | 110001  |
| 3          | Meera        | Pune  | 411001  |

 Functional dependencies:

* student_id → city
* city → pincode
   Hence, `student_id → pincode` (transitive) → violates 3NF.

---

###  **Normalized Output**

**Cities**

| city  | pincode |
| ----- | ------- |
| Pune  | 411001  |
| Delhi | 110001  |

**Students**

| student_id | student_name | city  |
| ---------- | ------------ | ----- |
| 1          | Asha         | Pune  |
| 2          | Rohit        | Delhi |
| 3          | Meera        | Pune  |

 City–pincode stored once → cleaner, consistent data.

---

###  SQL Example

```sql
CREATE TABLE Students_Unnormalized (
  student_id INT PRIMARY KEY,
  student_name VARCHAR(50),
  city VARCHAR(50),
  pincode VARCHAR(10)
);

INSERT INTO Students_Unnormalized VALUES
(1, 'Asha', 'Pune', '411001'),
(2, 'Rohit', 'Delhi', '110001'),
(3, 'Meera', 'Pune', '411001');
```

Normalized tables:

```sql
CREATE TABLE Cities (
  city VARCHAR(50) PRIMARY KEY,
  pincode VARCHAR(10)
);

CREATE TABLE Students (
  student_id INT PRIMARY KEY,
  student_name VARCHAR(50),
  city VARCHAR(50),
  FOREIGN KEY (city) REFERENCES Cities(city)
);

INSERT INTO Cities VALUES ('Pune', '411001'), ('Delhi', '110001');
INSERT INTO Students VALUES (1, 'Asha', 'Pune'), (2, 'Rohit', 'Delhi'), (3, 'Meera', 'Pune');
```

**Query**

```sql
SELECT s.student_name, s.city, c.pincode
FROM Students s
JOIN Cities c ON s.city = c.city;
```

**Output**

| student_name | city  | pincode |
| ------------ | ----- | ------- |
| Asha         | Pune  | 411001  |
| Rohit        | Delhi | 110001  |
| Meera        | Pune  | 411001  |

 3NF successfully achieved.

---

##  5. Practice Exercises (with output reasoning)

### **Exercise 1 — Identify transitive dependency**

| emp_id | emp_name | dept_id | dept_name | dept_location |
| ------ | -------- | ------- | --------- | ------------- |
| 1      | Ravi     | D1      | Sales     | Mumbai        |
| 2      | Meena    | D2      | HR        | Delhi         |

**Question:** What’s the transitive dependency?

**Answer:**

* emp_id → dept_id → dept_name, dept_location
  → Transitive dependency exists (via dept_id).

**Fix:** Separate `Departments(dept_id, dept_name, dept_location)`.

---

### **Exercise 2 — Normalize and show final tables**

**Input Table – Courses**

| course_id | course_name  | instructor | instructor_email                      |
| --------- | ------------ | ---------- | ------------------------------------- |
| C1        | SQL Basics   | Priya      | [priya@abc.com](mailto:priya@abc.com) |
| C2        | Python Intro | Raj        | [raj@abc.com](mailto:raj@abc.com)     |
| C3        | SQL Basics   | Priya      | [priya@abc.com](mailto:priya@abc.com) |

**Solution (3NF):**

**Instructors**

| instructor | instructor_email                      |
| ---------- | ------------------------------------- |
| Priya      | [priya@abc.com](mailto:priya@abc.com) |
| Raj        | [raj@abc.com](mailto:raj@abc.com)     |

**Courses**

| course_id | course_name  | instructor |
| --------- | ------------ | ---------- |
| C1        | SQL Basics   | Priya      |
| C2        | Python Intro | Raj        |
| C3        | SQL Basics   | Priya      |

 No duplication of email.
 instructor_email depends only on instructor → **3NF achieved**.

---

### **Exercise 3 — Predict output after normalization**

Input Table:

| emp_no | emp_name | branch | branch_city | branch_pin |
| ------ | -------- | ------ | ----------- | ---------- |
| 1      | Asha     | B1     | Pune        | 411001     |
| 2      | Rohit    | B2     | Delhi       | 110001     |
| 3      | Neha     | B1     | Pune        | 411001     |

**Normalized Tables:**

**Branches**

| branch | branch_city | branch_pin |
| ------ | ----------- | ---------- |
| B1     | Pune        | 411001     |
| B2     | Delhi       | 110001     |

**Employees**

| emp_no | emp_name | branch |
| ------ | -------- | ------ |
| 1      | Asha     | B1     |
| 2      | Rohit    | B2     |
| 3      | Neha     | B1     |

 Transitive dependency (branch_city, branch_pin) resolved.

---

##  6. Quick Recap Table

| Normal Form | Eliminates            | Key Rule                    |
| ----------- | --------------------- | --------------------------- |
| 1NF         | Repeating groups      | Make columns atomic         |
| 2NF         | Partial dependency    | Non-key depends on full key |
| 3NF         | Transitive dependency | Non-key depends only on key |


