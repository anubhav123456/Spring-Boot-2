
---

# 🔥 What is `@Transactional`?

`@Transactional` (Spring) = **“Run this method inside a database transaction.”**

A **transaction** means:

* ✅ All DB operations succeed → **COMMIT**
* ❌ If any error happens → **ROLLBACK**

```java
@Transactional
public void placeOrder() {
    saveOrder();        // DB insert
    updateInventory();  // DB update
}
```

If `updateInventory()` fails → `saveOrder()` also rolls back. No half data. 💪

---

# 🧠 But what is **Propagation**?

Propagation decides:

> **If a method with @Transactional is called from another transactional method, what should happen?**

Matlab:
**Existing transaction join kare ya new banaye ya error de?**

---

# 🚦 All Propagation Types (with real examples)

---

## 1️⃣ `REQUIRED` (Default) ⭐

> **Join existing transaction. If none exists → create new.**

```java
@Transactional(propagation = Propagation.REQUIRED)
public void addToCart() { }

@Transactional
public void checkout() {
    addToCart();  // joins same transaction
}
```

### Flow:

```
checkout() TX-1 START
   → addToCart() joins TX-1
checkout() COMMIT
```

❌ If `addToCart()` fails → whole transaction rolls back.

👉 **Used in 90% of cases**

---

## 2️⃣ `REQUIRES_NEW`

> **Always start a NEW transaction. Suspend old one.**

```java
@Transactional
public void placeOrder() {
    saveOrder();          // TX-1
    sendEmailLog();       // TX-2 (new)
}

@Transactional(propagation = Propagation.REQUIRES_NEW)
public void sendEmailLog() {
    saveLog();
}
```

### Flow:

```
TX-1 START (placeOrder)
   saveOrder()
   TX-1 PAUSED
   TX-2 START (sendEmailLog)
   TX-2 COMMIT
TX-1 RESUME
TX-1 FAIL → ROLLBACK
```

👉 Even if main order fails, **email log stays saved**.

**Use case:** Logging, audit history, payment records.

---

## 3️⃣ `SUPPORTS`

> If transaction exists → join
> If not → run WITHOUT transaction

```java
@Transactional(propagation = Propagation.SUPPORTS)
public List<Product> getProducts() { }
```

Used for **read-only methods**.

If called inside TX → joins
If called alone → no transaction

---

## 4️⃣ `NOT_SUPPORTED`

> Always run WITHOUT transaction. Suspend existing one.

```java
@Transactional(propagation = Propagation.NOT_SUPPORTED)
public void callExternalAPI() { }
```

DB transaction is paused.
Useful when doing **long API calls** (don’t keep DB locked).

---

## 5️⃣ `MANDATORY`

> Must have existing transaction. Otherwise → **Exception**

```java
@Transactional(propagation = Propagation.MANDATORY)
public void updateStock() { }
```

If called without TX:

```
IllegalTransactionStateException
```

Used in strict layered architectures.

---

## 6️⃣ `NEVER`

> Must NOT run inside transaction. If TX exists → Exception.

```java
@Transactional(propagation = Propagation.NEVER)
public void systemHealthCheck() { }
```

---

## 7️⃣ `NESTED` (Savepoint magic) 🧙‍♂️

> Runs inside parent TX but creates a **savepoint**

```java
@Transactional
public void parent() {
    methodA();
    methodB(); // nested
}

@Transactional(propagation = Propagation.NESTED)
public void methodB() {
    // failure here rolls back only methodB
}
```

If `methodB()` fails → rollback to savepoint
Parent TX can still continue.

⚠ Works only with JDBC savepoints + specific DBs.

---

# 🎯 Summary Table

| Propagation   | Existing TX? | What Happens  |
| ------------- | ------------ | ------------- |
| REQUIRED      | Yes          | Join it       |
| REQUIRED      | No           | Create new    |
| REQUIRES_NEW  | Yes/No       | Always new TX |
| SUPPORTS      | Yes          | Join          |
| SUPPORTS      | No           | No TX         |
| NOT_SUPPORTED | Yes          | Suspend TX    |
| MANDATORY     | No           | Exception     |
| NEVER         | Yes          | Exception     |
| NESTED        | Yes          | Savepoint TX  |

---

# 💥 Interview Question Trick

**Q:** Why use `REQUIRES_NEW` for logging?

**A:** Because even if main transaction rolls back, logs must still be committed.

---

# ⚠ Important Real-World Gotchas

### ❗1. Only works on **public methods**

Spring proxy limitation.

### ❗2. Self-calling won’t work

```java
this.methodB(); // Transaction ignored
```

### ❗3. Only works on **runtime exceptions** by default

Checked exception → no rollback unless:

```java
@Transactional(rollbackFor = Exception.class)
```

---
