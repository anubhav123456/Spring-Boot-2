
---

# 🧠 Topic: `@Async` Annotation — Conditions & Transaction Pitfall

---

## ✅ PART 1 — Conditions for `@Async` to Work Properly

### 📌 Condition 1: Method must be in a **different class**

* Spring uses **AOP Proxy**
* Proxy only works when **one bean calls another bean**
* **Self-invocation (same class calling its own method)** bypasses proxy → `@Async` ignored

### 📌 Condition 2: Method must be **public**

* Spring AOP works on **public methods**
* `private/protected` methods cannot be proxied → `@Async` won’t run

---

## ❓ WHY these conditions exist?

Because **`@Async` works using AOP interception**

Flow internally:

```
Caller → Proxy → Interceptor → New Thread → Actual Method
```

If:

* Same class → proxy not involved ❌
* Method not public → proxy cannot intercept ❌

So no async behavior.

---

# ❌ WRONG USE CASE (Self Invocation)

### Same class calling its own async method

```java
@RestController
@RequestMapping(value = "/api")
public class UserController {

    @PostMapping(path = "/updateuser")
    public String updateUserMethod() {
        test();   // calling async method inside same class ❌
        return null;
    }

    @Async
    public void test() {
        // Expectation: new thread
        // Reality: runs in same thread
    }
}
```

### 🧨 Result:

Thread name remains same
➡ No new thread created
➡ `@Async` NOT working

---

# ✅ CORRECT APPROACH

Move async method to another Spring Bean.

---

# 🧨 PART 2 — Interview Question: `@Async` + `@Transactional`

> **Use Case 1: Transactional method calling Async method**

---

## 🔹 Controller

```java
@RestController
@RequestMapping(value = "/api")
public class UserController {

    @Autowired
    UserService userService;

    @PostMapping(path = "/updateuser")
    public String updateUserMethod() {
        userService.updateUser();
        return null;
    }
}
```

---

## 🔹 Service (Transactional)

```java
@Component
public class UserService {

    @Autowired
    UserUtility userUtility;

    @Transactional
    public void updateUser() {

        // 1. Update user status
        // 2. Update user first name
        // 3. Update user

        userUtility.updateUserBalance(); // async call
    }
}
```

---

## 🔹 Async Utility Class

```java
@Component
public class UserUtility {

    @Async
    public void updateUserBalance() {
        // Updating user balance amount
    }
}
```

---

# 🚨 BIG PROBLEM HERE

### ❗ Transaction context DOES NOT pass to async thread

When `updateUser()` runs:

* Transaction starts in **Caller Thread**

When `updateUserBalance()` runs:

* Runs in **New Thread**
* ❌ No transaction context available

---

## 🧠 What does Transaction Context contain?

* Propagation level
* Isolation level
* Rollback rules
* Connection binding to thread

All of this is **ThreadLocal based** → not shared across threads.

---

## 🔥 Result of Failure Scenario

If exception happens:

| Operation              | Rollback? |
| ---------------------- | --------- |
| Update user status     | ✅ YES     |
| Update first name      | ✅ YES     |
| Update user            | ✅ YES     |
| Update balance (Async) | ❌ NO      |

➡ Database inconsistency 😱

---

# ❌ Conclusion

**Avoid this pattern:**

```java
@Transactional
public void method() {
    asyncMethod();   // ❌ Dangerous
}
```

Because:

> **Transaction context does not transfer from caller thread to async thread**

---

# 💬 Interview One-Line Answer

> “Spring `@Async` runs in a different thread and transactions are ThreadLocal based, so transaction context does not propagate to async methods, which can lead to partial commits.”

---
