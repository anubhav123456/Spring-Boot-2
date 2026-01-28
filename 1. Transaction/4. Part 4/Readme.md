
---

## 🧱 1. Your NORMAL Business Code - You only write this

### Entity

```java
@Entity
public class Account {

    @Id
    private Long id;
    private double balance;

    // getters & setters
}
```

---

### Repository

```java
@Repository
public interface AccountRepository extends JpaRepository<Account, Long> {
}
```

---

### Service (Your Logic)

```java
@Service
public class BankService {

    @Autowired
    private AccountRepository repo;

    @Transactional   // 👈 THIS triggers AOP
    public void transferMoney(Long fromId, Long toId, double amount) {

        Account sender = repo.findById(fromId).get();
        Account receiver = repo.findById(toId).get();

        // Step 1: Deduct money
        sender.setBalance(sender.getBalance() - amount);
        repo.save(sender);

        // Simulating system failure 💥
        if (true) {
            throw new RuntimeException("Something went wrong!");
        }

        // Step 2: Add money
        receiver.setBalance(receiver.getBalance() + amount);
        repo.save(receiver);
    }
}
```

---

## 🧠 2. What YOU Think Happens

You think Spring calls this directly:

```text
Controller → BankService.transferMoney() → DB
```

❌ WRONG

---

## 🎭 3. What ACTUALLY Happens (AOP Proxy)

Spring creates a **proxy class** behind the scenes.

Something like this (simplified version of what Spring generates):

```java
public class BankServiceProxy extends BankService {

    private TransactionManager txManager;

    @Override
    public void transferMoney(Long fromId, Long toId, double amount) {

        try {
            txManager.beginTransaction();   // 🟢 BEFORE method

            super.transferMoney(fromId, toId, amount);  // 👉 Your real method

            txManager.commit();  // 🟢 AFTER success

        } catch (Exception e) {
            txManager.rollback();  // 🔴 IF ERROR
            throw e;
        }
    }
}
```

---

## 🔄 4. Flow When Method is Called

### Controller

```java
@RestController
public class BankController {

    @Autowired
    private BankService bankService;

    @PostMapping("/transfer")
    public String transfer() {
        bankService.transferMoney(1L, 2L, 500);
        return "Done";
    }
}
```

### Actual Call Flow:

```text
Controller
   ↓
BankServiceProxy.transferMoney()   👈 AOP PROXY
   ↓
🟢 Transaction STARTS
   ↓
Real BankService.transferMoney()
   ↓
💥 Exception happens
   ↓
🔴 Transaction ROLLBACK
```

---

## 📌 What Happens in Database?

| Step           | Sender Balance | Receiver Balance |
| -------------- | -------------- | ---------------- |
| Before         | 1000           | 500              |
| Deduct step    | 500            | 500              |
| Crash happens  | ❌              | ❌                |
| After rollback | 1000           | 500              |

👉 Because proxy said: **“Error? Undo everything.”**

---

## 🎯 Mapping to AOP Terms

| AOP Term       | Here It Means               |
| -------------- | --------------------------- |
| **Aspect**     | Transaction system          |
| **Advice**     | Begin, Commit, Rollback     |
| **Join Point** | `transferMoney()` execution |
| **Proxy**      | `BankServiceProxy`          |

---

## 🧠 Final One-Line Understanding

When you write:

```java
@Transactional
public void transferMoney() { }
```

Spring secretly changes it to:

```java
beginTransaction();
yourMethod();
commit OR rollback;
```

using an **AOP proxy wrapper**.

---
