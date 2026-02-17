
---

# 🔥 What is Cascade in Spring Boot?

In **Spring Boot**, cascading is handled by **JPA (Hibernate)**.

Cascade means:

> When we perform an operation on a parent entity, the same operation automatically applies to the child entity.

Think of it like:

👨‍👦 Parent does something → Child automatically follows.

---

# 🧠 Real Life Example

Imagine:

* `Order` (Parent)
* `OrderItem` (Child)

If you delete the Order:

* Should the OrderItems also delete automatically?
  👉 That’s what **cascade** controls.

---

# 📦 Cascade Types in JPA

These are defined in:

```java
import jakarta.persistence.CascadeType;
```

Main Cascade Types:

1. `PERSIST`
2. `MERGE`
3. `REMOVE`
4. `REFRESH`
5. `DETACH`
6. `ALL`

---

# 1️⃣ CascadeType.PERSIST

👉 When you save parent, child also gets saved automatically.

### Example

```java
@Entity
public class Order {

    @OneToMany(mappedBy = "order", cascade = CascadeType.PERSIST)
    private List<OrderItem> items;
}
```

### What happens?

```java
orderRepository.save(order);
```

✔ Order saved
✔ OrderItems saved automatically

Without `PERSIST`, you would need to manually save each child.

---

# 2️⃣ CascadeType.MERGE

👉 When you update parent, child also updates.

Example:

```java
@OneToMany(mappedBy = "order", cascade = CascadeType.MERGE)
```

```java
orderRepository.save(order); // update
```

✔ Order updated
✔ OrderItems updated

Used when updating existing entities.

---

# 3️⃣ CascadeType.REMOVE

👉 When parent is deleted, child is also deleted.

```java
@OneToMany(mappedBy = "order", cascade = CascadeType.REMOVE)
```

```java
orderRepository.delete(order);
```

✔ Order deleted
✔ OrderItems deleted

⚠ Very important in OneToMany relationships.

Without this → child records remain in DB (orphan records).

---

# 4️⃣ CascadeType.REFRESH

👉 When parent is refreshed from DB, child also refreshes.

```java
entityManager.refresh(order);
```

✔ Reloads order
✔ Reloads order items

Rarely used in normal projects.

---

# 5️⃣ CascadeType.DETACH

👉 When parent is detached from persistence context, child also detaches.

```java
entityManager.detach(order);
```

Mostly used in advanced Hibernate use cases.

---

# 6️⃣ CascadeType.ALL

👉 Applies ALL operations.

```java
@OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
```

Equivalent to:

```java
cascade = {
    PERSIST,
    MERGE,
    REMOVE,
    REFRESH,
    DETACH
}
```

🔥 Most commonly used in real projects.

---

# 🏗 Complete Working Example

## Parent Entity

```java
@Entity
public class Order {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
    private List<OrderItem> items;
}
```

## Child Entity

```java
@Entity
public class OrderItem {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne
    @JoinColumn(name = "order_id")
    private Order order;
}
```

---


# 🚨 Important Production Advice

❌ Never blindly use `CascadeType.ALL` everywhere.

Example:

If you use `REMOVE` in ManyToMany (like User–Role):

Deleting one user might delete roles too 😱

Better to control carefully.

---

# 💡 Super Simple Analogy

Parent = Spring Boot Project
Child = Database Tables

Cascade = “Jo parent karega, child bhi karega.”

---
