
---

# 🌟 Spring Boot `@Transactional` — Complete Notes

---

## 🔥 1. Why Do We Need Transactions?

When multiple requests access **shared data** at the same time, **data inconsistency** can happen.

### 👉 Critical Section

A **critical section** is:

> A code segment where **shared resources are accessed & modified**

**Example (Cab Booking System)**
Car in DB:

| Car ID | Status    |
| ------ | --------- |
| 1001   | Available |

If **4 users book simultaneously**:

* All read → "Available"
* All update → "Booked"

❌ Same cab booked 4 times → **Inconsistent Data**

✅ Solution → **Transactions**

---

# 🧠 2. ACID Properties (Guaranteed by Transactions)

| Property            | Meaning                                     |
| ------------------- | ------------------------------------------- |
| **A – Atomicity**   | All operations succeed OR all rollback      |
| **C – Consistency** | DB remains valid before & after transaction |
| **I – Isolation**   | Parallel transactions don’t interfere       |
| **D – Durability**  | Once committed, data is permanent           |

---

### 🔹 Atomicity Example

Initial State:

| A   | B   |
| --- | --- |
| ₹10 | ₹20 |

Transaction:

1. Debit A ₹5
2. Credit B ₹5

If **credit fails**, debit also rolls back.

Final State remains:

| A   | B   |
| --- | --- |
| ₹10 | ₹20 |

---

### 🔹 Isolation

Even if multiple transactions run:

```
T1 → Debit A ₹5
T2 → Debit A ₹2
T3 → Credit A ₹1
```

Spring/DB ensures **locking & sequencing** so they behave like independent executions.

---

### 🔹 Durability

Once **commit happens**, data stays even after crash.

---

# 🔁 3. Traditional Transaction Handling (Verbose)

```java
beginTransaction();
try {
    debit(A);
    credit(B);
    commit();
} catch (Exception e) {
    rollback();
}
endTransaction();
```

⚠️ Problem:

* Must repeat in **every DB method**
* Boilerplate code everywhere
* Business logic gets buried

---

# 🚀 4. Spring Solution → `@Transactional`

Spring uses **AOP (Aspect Oriented Programming)** to:

* Begin transaction
* Commit if success
* Rollback if failure
  Automatically around your method.

You only write **business logic**.

---

# 📦 5. Required Dependencies

### ✅ For Relational DB (JPA)

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

### ✅ Database Driver (Example: MySQL)

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <scope>runtime</scope>
</dependency>
```

📌 Driver dependency depends on the DB you use.

---

# ⚙️ 6. application.properties

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=pass

spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
```

---

# 🔧 7. Enable Transaction Management

Usually **Auto-configured by Spring Boot**.

Optional manual enable:

```java
@SpringBootApplication
@EnableTransactionManagement
public class SpringbootApplication {

    public static void main(String args[]) {
        SpringApplication.run(SpringbootApplication.class, args);
    }
}
```

---

# 🧩 8. Using `@Transactional`

### ✅ Method Level

```java
@Service
public class CarService {

    @Transactional
    public void bookCar(Long carId) {
        Car car = carRepo.findById(carId).get();

        if(car.getStatus().equals("AVAILABLE")) {
            car.setStatus("BOOKED");
            carRepo.save(car);
        }
    }
}
```

---

### ✅ Class Level (applies to all public methods)

```java
@Component
@Transactional
public class CarService {

    public void updateCar() {
        // Transactional
    }

    public void updateBulkCars() {
        // Transactional
    }

    private void helperMethod() {
        // ❌ NOT transactional (private methods not proxied)
    }
}
```

---

# 🧠 9. What Happens Internally?

Spring creates a **proxy** around your class.

When method is called:

```
Proxy → Start Transaction
       → Execute Method
       → Commit (if success)
       → Rollback (if exception)
```

---

# 🎯 10. Key Points to Remember

✔ Transactions prevent **race conditions**
✔ Provide **ACID guarantees**
✔ Implemented using **AOP Proxy**
✔ Only works on **public methods**
✔ Internal method calls **bypass proxy**
✔ No need to write manual commit/rollback

---

# 💡 Interview One-Liner

> "`@Transactional` in Spring Boot uses AOP to wrap methods in a database transaction and ensures ACID properties like atomicity, isolation, and consistency automatically."

---
