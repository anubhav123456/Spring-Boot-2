
---

## ✅ Why `@Id` is used?

It tells JPA:

> 👉 “This field uniquely identifies each row in the database table.”

Every entity must have **one primary key**, and that field is marked with `@Id`.

---

## 🔹 Basic Example

```java
import jakarta.persistence.Entity;
import jakarta.persistence.Id;

@Entity
public class User {

    @Id
    private Long id;

    private String name;
    private String email;

    // getters & setters
}
```

Here:

* `@Entity` → Maps class to DB table
* `@Id` → Marks `id` as **Primary Key**

---

## 🔥 Important Rules

1. Every `@Entity` **must** have an `@Id`
2. It should be:

   * Unique
   * Not null
   * Ideally immutable
3. Can be:

   * `Long`
   * `Integer`
   * `UUID`
   * Composite key

---

## 🧠 Real-Life Analogy

Think of `@Id` like:

* **Aadhar Card number**
* **Passport number**
* **Employee ID**

It uniquely identifies a person in a system.

---
