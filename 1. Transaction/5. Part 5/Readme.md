

---

# 💰 First: What is a Transaction?

A **transaction** is a group of DB operations that should act like **ONE UNIT**.

Either:

✅ Everything succeeds → **COMMIT**
❌ Anything fails → **ROLLBACK (undo all)**

Example:

> Transfer ₹1000 from A → B
> If money is deducted from A but not added to B = 💥 disaster
> So both steps must succeed together.

---

# 🚦 Two Ways to Manage Transactions in Spring

| Type             | Style            | Who controls it?           | Used in real projects? |
| ---------------- | ---------------- | -------------------------- | ---------------------- |
| **Declarative**  | `@Transactional` | Spring handles everything  | ⭐⭐⭐⭐⭐ YES              |
| **Programmatic** | Manual code      | YOU handle commit/rollback | Rare / special cases   |

---

# 🟢 1. Declarative Transaction (Using `@Transactional`)

This is the **modern, easy, industry standard** way.

You just write:

```java
@Transactional
public void transferMoney() {
   withdraw();
   deposit();
}
```

Spring secretly does all this:

1. Opens transaction before method
2. Runs method
3. If success → COMMIT
4. If exception → ROLLBACK

You don’t write commit/rollback code.

---

## 🔍 How Spring Does This? → AOP

Spring uses **AOP Proxy** behind the scenes.

Think like this:

```
You call → transferMoney()
      ↓
Spring proxy intercepts
      ↓
START TRANSACTION
      ↓
Run method
      ↓
If no error → COMMIT
If error → ROLLBACK
```

You don't see this magic. It happens automatically.

---

## 🧠 Example

```java
@Service
public class BankService {

    @Autowired
    private AccountRepository repo;

    @Transactional
    public void transfer(Long fromId, Long toId, double amount) {

        Account from = repo.findById(fromId).get();
        Account to   = repo.findById(toId).get();

        from.setBalance(from.getBalance() - amount);
        repo.save(from);

        // Suppose error happens here
        if (true) throw new RuntimeException("Something failed");

        to.setBalance(to.getBalance() + amount);
        repo.save(to);
    }
}
```

### What happens?

Even though `from` balance was deducted:

❌ Exception occurs
➡ Spring rolls back
➡ DB remains unchanged

🔥 That’s power of declarative transactions.

---

## 🎛 You can control behavior

```java
@Transactional(
    rollbackFor = Exception.class,
    isolation = Isolation.READ_COMMITTED,
    propagation = Propagation.REQUIRED,
    timeout = 5
)
```

But 90% of time → just `@Transactional` is enough.

---

# 🔵 2. Programmatic Transaction (Manual Control)

Here **YOU** control transaction logic.

Used when:

* Complex flow
* Dynamic transaction handling
* Legacy systems

---

## Example

```java
@Autowired
private PlatformTransactionManager txManager;

public void transfer() {

    TransactionStatus status = txManager.getTransaction(new DefaultTransactionDefinition());

    try {
        withdraw();
        deposit();

        txManager.commit(status);   // SUCCESS
    } catch (Exception e) {
        txManager.rollback(status); // FAILURE
    }
}
```

You are manually doing what Spring did automatically earlier.

---

# ⚖️ Declarative vs Programmatic

| Feature          | Declarative | Programmatic |
| ---------------- | ----------- | ------------ |
| Code complexity  | Very low    | High         |
| Control          | Medium      | Full         |
| Readability      | Clean       | Messy        |
| Used in industry | 95% cases   | Rare         |

---

# 🏆 Interview Line (Use this 😏)

> "In Spring Boot, transaction management is usually declarative using `@Transactional`, where Spring uses AOP proxies to wrap the method, start a transaction before execution, and automatically commit or roll back depending on whether the method completes normally or throws an exception."

---

# 🔥 Rule to Remember

> **If you're writing commit/rollback yourself → you're doing programmatic transactions.
> If you're using `@Transactional` → it's declarative.**

---
