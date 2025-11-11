

# **Core Database Properties (ACID)**

These are the core guarantees any reliable transactional database must provide.

---

##  1. **Atomicity**

### **Meaning**

A transaction must execute **all or nothing**.
If any step fails, the entire transaction is rolled back.

- Example (Real-life analogy):
You use UPI to transfer ₹500.

* Amount deducted from your bank
* Amount credited to receiver
  If credit fails, deduction must also be reversed.

- Example (Database)

```sql
START TRANSACTION;
UPDATE Accounts SET balance = balance - 500 WHERE account_id = 101;
UPDATE Accounts SET balance = balance + 500 WHERE account_id = 202;
COMMIT;
```

If the second update fails, the first one must roll back — no half-payment.

---

##  2. **Consistency**

### **Meaning**

Before and after a transaction, **data must remain valid** according to rules, constraints, triggers.

- Example:

* Marks cannot be >100
* Age cannot be negative
* Balance cannot go below minimum for some banks

If a transaction violates rules → entire transaction fails.

```sql
INSERT INTO Students (name, marks) VALUES ('Ravi', 120);   -- Fails
```

---

##  3. **Isolation**

### **Meaning**

Transactions running at the same time **should not interfere** with each other.

- Example:
Two people booking the last movie ticket.
Database must ensure both don’t get it.

- Why needed?
Prevent issues like:

* Dirty Read (read uncommitted data)
* Lost Updates
* Non-repeatable Read
* Phantom Read

DB uses **transaction isolation levels**:

* Read Uncommitted (least safe)
* Read Committed
* Repeatable Read
* Serializable (most strict)

---

##  4. **Durability**

### **Meaning**

Once a transaction is committed, data **will not be lost**, even if system crashes.

- Example:

* Money transferred
* Server crashed
  - When restarted → changes still there (written to disk / logs / persistent storage)

---

#  ACID in Short

| Property    | Guarantee                   |
| ----------- | --------------------------- |
| Atomicity   | All or none                 |
| Consistency | Data stays valid            |
| Isolation   | Transactions don’t conflict |
| Durability  | Data survives crash         |

---

#  Practice Exercises (With Solutions)

##  **Exercise 1**

A student enrollment system has these steps in a transaction:

1. Reduce total available seats
2. Insert record in enrollment table
3. Deduct enrollment fee from student wallet

If step 3 fails, what ACID property ensures rollback?

 **Answer:** **Atomicity**

---

##  **Exercise 2**

Bank database must not allow account balance to go negative.
Which ACID property ensures this rule is always maintained?

 **Answer:** **Consistency**

---

##  **Exercise 3**

Two users try to purchase the last laptop from an ecommerce site.
Which property prevents both purchases from succeeding?

 **Answer:** **Isolation**

---

##  **Exercise 4**

A transaction was marked COMMITTED, server crashed after 2 seconds.
Will data exist? Which property guarantees this?

 **Answer:** **Durability**

---

##  **Exercise 5 – Output Prediction**

Consider:

```sql
START TRANSACTION;
UPDATE products SET quantity = quantity - 1 WHERE product_id = 10;
UPDATE products SET sold = sold + values('x');   -- syntax error
COMMIT;
```

Will quantity reduce by 1?

 **Answer:**
**No**, because the second query fails → entire transaction rolls back → Atomicity protects data.

---

#  Hands-On Task (Try On Your DB)

### Create a simple bank table

```sql
CREATE TABLE Accounts (
    account_id INT PRIMARY KEY,
    balance INT
);

INSERT INTO Accounts VALUES (1, 1000), (2, 500);
```

### Perform a safe money transfer using a transaction

```sql
START TRANSACTION;
UPDATE Accounts SET balance = balance - 200 WHERE account_id = 1;
UPDATE Accounts SET balance = balance + 200 WHERE account_id = 2;
COMMIT;
```

### Test rollback scenario

```sql
START TRANSACTION;
UPDATE Accounts SET balance = balance - 200 WHERE account_id = 1;
-- Mistake: wrong column name (forces error)
UPDATE Accounts SET bal = balance + 200 WHERE account_id = 2;
ROLLBACK;
```

 Check balances → unchanged → Atomicity works.

