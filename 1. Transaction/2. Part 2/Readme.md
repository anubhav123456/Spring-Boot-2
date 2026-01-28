
---

# 🧠 Spring `@Transactional` 

## 🔹 1. Transaction at **Class Level**

```java
@Component
@Transactional
public class CarService {

    public void updateCar() {
        // Executed within a transaction
    }

    public void updateBulkCars() {
        // Executed within a transaction
    }

    private void helperMethod() {
        // NOT affected by @Transactional
    }
}
```

### ✅ What happens?

| Method Type       | Transaction Applied? | Why                                    |
| ----------------- | -------------------- | -------------------------------------- |
| `public` methods  | ✅ YES                | Spring AOP proxies only public methods |
| `private` methods | ❌ NO                 | Proxy cannot intercept private methods |

👉 Class level = **default transaction for all public methods**

---

## 🔹 2. Transaction at **Method Level**

```java
@Component
public class CarService {

    @Transactional
    public void updateCar() {
        // Runs inside transaction
    }

    public void updateBulkCars() {
        // No transaction
    }
}
```

### ✅ What happens?

Only the method having `@Transactional` runs in a transaction.

---