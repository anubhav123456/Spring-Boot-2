# Composite Primary Key

In **Spring Boot (JPA / Hibernate)**, `@Embeddable` and `@EmbeddedId` are mainly used for **composite primary keys** (when a table has more than one column as primary key).

Let’s understand clearly 👇

---

# 🔹 1. `@Embeddable`

`@Embeddable` is used to define a **class that can be embedded inside an entity**.

It represents a reusable value type.

### ✅ Example: Composite Key Class

```java
import jakarta.persistence.Embeddable;
import java.io.Serializable;
import java.util.Objects;

@Embeddable
public class OrderId implements Serializable {

    private Long orderId;
    private Long customerId;

    // Default constructor
    public OrderId() {}

    public OrderId(Long orderId, Long customerId) {
        this.orderId = orderId;
        this.customerId = customerId;
    }

    // getters and setters

    // IMPORTANT: override equals() and hashCode()
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof OrderId)) return false;
        OrderId that = (OrderId) o;
        return Objects.equals(orderId, that.orderId) &&
               Objects.equals(customerId, that.customerId);
    }

    @Override
    public int hashCode() {
        return Objects.hash(orderId, customerId);
    }
}
```

### 🔥 Important Rules for `@Embeddable`

* Must implement `Serializable`
* Must have default constructor
* Must override `equals()` and `hashCode()`

---

# 🔹 2. `@EmbeddedId`

`@EmbeddedId` is used inside an **Entity** to specify that the primary key is a composite key defined using `@Embeddable`.

---

## ✅ Entity Using `@EmbeddedId`

```java
import jakarta.persistence.*;

@Entity
@Table(name = "orders")
public class Order {

    @EmbeddedId
    private OrderId id;

    private String productName;
    private int quantity;

    // getters and setters
}
```




---

# 🚀 When Should You Use It?

Use `@Embeddable` + `@EmbeddedId` when:

* Many-to-Many join table with extra fields
* Composite primary key required
* Business key contains multiple columns
* Legacy database already designed with composite keys

---

# ⚠️ Common Mistakes

❌ Forgetting `equals()` and `hashCode()`

❌ Not implementing `Serializable`

❌ No default constructor

❌ Trying to use both `@Id` and `@EmbeddedId` together

---

