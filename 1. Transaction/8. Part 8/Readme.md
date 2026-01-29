

---

# 🔹 What is `@Transactional` in Spring?

`@Transactional` is used to **manage database transactions** in Spring.

A **transaction** is a group of operations that must happen **all together or not at all**.

👉 If anything fails → **rollback**
👉 If everything succeeds → **commit**

### Example

```java
@Service
public class OrderService {

    @Transactional
    public void placeOrder() {
        deductMoney();      // DB Operation 1
        createOrder();      // DB Operation 2
        updateInventory();  // DB Operation 3
    }
}
```

If `updateInventory()` fails → all previous DB changes are rolled back.

---

# 🔹 What is Isolation Level?

When **multiple transactions run at the same time**, they can interfere with each other.

**Isolation Level** defines:

> *How much one transaction is allowed to see another transaction’s data.*

Configured like:

```java
@Transactional(isolation = Isolation.READ_COMMITTED)
```

---

# 🔥 The 3 Data Problems (VERY IMPORTANT)

| Problem                 | Meaning                                                     |
| ----------------------- | ----------------------------------------------------------- |
| **Dirty Read**          | Reading data that another transaction has not committed yet |
| **Non-Repeatable Read** | Same row read twice, value changed in between               |
| **Phantom Read**        | Same query returns new rows on second execution             |

Let’s understand with examples 👇

---

## 🟥 1. Dirty Read

**Meaning:** You read data that might later be rolled back.

### Scenario

**Transaction A**

```sql
UPDATE account SET balance = 5000 WHERE id = 1;  -- not committed yet
```

**Transaction B**

```sql
SELECT balance FROM account WHERE id = 1;  -- reads 5000
```

Then **Transaction A rolls back**.

Now the real balance is still **10000**, but B saw **fake data (5000)**.

❌ This is a **Dirty Read**

---

## 🟧 2. Non-Repeatable Read

**Meaning:** You read the same row twice but get different values.

### Scenario

**Transaction A**

```sql
SELECT balance FROM account WHERE id = 1;  -- returns 10000
```

**Transaction B**

```sql
UPDATE account SET balance = 7000 WHERE id = 1;
COMMIT;
```

**Transaction A again**

```sql
SELECT balance FROM account WHERE id = 1;  -- returns 7000
```

Same row, different value → ❌ **Non-Repeatable Read**

---

## 🟨 3. Phantom Read

**Meaning:** Same query returns **new rows**.

### Scenario

**Transaction A**

```sql
SELECT * FROM orders WHERE amount > 1000;  -- returns 5 rows
```

**Transaction B**

```sql
INSERT INTO orders(amount) VALUES(2000);
COMMIT;
```

**Transaction A again**

```sql
SELECT * FROM orders WHERE amount > 1000;  -- returns 6 rows
```

A new row "appeared" → 👻 **Phantom Read**

---

# 🔹 Isolation Levels in Spring

| Isolation Level      | Dirty Read  | Non-Repeatable Read | Phantom Read | Speed   |
| -------------------- | ----------- | ------------------- | ------------ | ------- |
| **READ_UNCOMMITTED** | ❌ Possible  | ❌ Possible          | ❌ Possible   | Fastest |
| **READ_COMMITTED**   | ✅ Prevented | ❌ Possible          | ❌ Possible   | Default |
| **REPEATABLE_READ**  | ✅ Prevented | ✅ Prevented         | ❌ Possible   | Slower  |
| **SERIALIZABLE**     | ✅ Prevented | ✅ Prevented         | ✅ Prevented  | Slowest |

---

## 🔹 1. READ_UNCOMMITTED

```java
@Transactional(isolation = Isolation.READ_UNCOMMITTED)
```

* Transactions can see **uncommitted changes**
* Dirty reads allowed ❌
* Rarely used

---

## 🔹 2. READ_COMMITTED (Most Common)

```java
@Transactional(isolation = Isolation.READ_COMMITTED)
```

* Can only read **committed data**
* Prevents Dirty Read ✅
* Non-repeatable & Phantom still possible

---

## 🔹 3. REPEATABLE_READ

```java
@Transactional(isolation = Isolation.REPEATABLE_READ)
```

* Same row will always have same value inside a transaction
* Prevents:

  * Dirty Read ✅
  * Non-Repeatable Read ✅
* Phantom Read still possible

---

## 🔹 4. SERIALIZABLE (Most Strict)

```java
@Transactional(isolation = Isolation.SERIALIZABLE)
```

* Transactions run as if **one after another**
* Prevents all problems ✅
* Slow due to locking

---

# 🔹 Real Spring Boot Example

```java
@Transactional(
    isolation = Isolation.REPEATABLE_READ,
    propagation = Propagation.REQUIRED,
    rollbackFor = Exception.class
)
public void transferMoney(Long from, Long to, double amount) {
    Account a = accountRepo.findById(from).get();
    Account b = accountRepo.findById(to).get();

    a.setBalance(a.getBalance() - amount);
    b.setBalance(b.getBalance() + amount);
}
```

Here:

* Data remains consistent
* No dirty or non-repeatable reads

---

# 🧠 Interview One-Liner

> Isolation levels control how transactions see each other’s data to prevent dirty reads, non-repeatable reads, and phantom reads.

---
