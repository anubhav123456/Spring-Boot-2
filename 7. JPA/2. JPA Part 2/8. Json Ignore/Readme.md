

---

# 🔹 What is `@JsonIgnore` in Spring Boot?

`@JsonIgnore` is an annotation from:

```
com.fasterxml.jackson.annotation.JsonIgnore
```

Spring Boot uses **Jackson** internally for converting:

* Java Object ➝ JSON (Serialization)
* JSON ➝ Java Object (Deserialization)

👉 `@JsonIgnore` kisi field ko JSON response/request se **exclude** kar deta hai.

---

# 🔥 Why We Use It?

1. Sensitive data hide karne ke liye (password, tokens)
2. Infinite recursion avoid karne ke liye (JPA relationships)
3. Unwanted fields remove karne ke liye API response clean rakhne ke liye

---

# ✅ Basic Example (Password Hide Karna)

## 👨‍💻 Without `@JsonIgnore`

```java
import lombok.Data;

@Data
public class User {

    private Long id;
    private String name;
    private String email;
    private String password;
}
```

### Controller

```java
@RestController
public class UserController {

    @GetMapping("/user")
    public User getUser() {
        return new User(1L, "Anubhav", "anu@gmail.com", "123456");
    }
}
```

### 🔴 Output JSON

```json
{
  "id": 1,
  "name": "Anubhav",
  "email": "anu@gmail.com",
  "password": "123456"
}
```

❌ Password expose ho gaya — Security issue.

---

## ✅ With `@JsonIgnore`

```java
import com.fasterxml.jackson.annotation.JsonIgnore;
import lombok.Data;

@Data
public class User {

    private Long id;
    private String name;
    private String email;

    @JsonIgnore
    private String password;
}
```

### 🟢 Now Output

```json
{
  "id": 1,
  "name": "Anubhav",
  "email": "anu@gmail.com"
}
```

✔ Password JSON me nahi aayega
✔ Object me internally available rahega

---

# 🔥 Real JPA Example (Very Important for Interviews)

Assume:

* One User
* Many Orders

### User Entity

```java
@Entity
public class User {

    @Id
    @GeneratedValue
    private Long id;

    private String name;

    @OneToMany(mappedBy = "user")
    private List<Order> orders;
}
```

### Order Entity

```java
@Entity
public class Order {

    @Id
    @GeneratedValue
    private Long id;

    private String product;

    @ManyToOne
    private User user;
}
```

### ❌ Problem: Infinite Recursion

User ➝ Orders ➝ User ➝ Orders ➝ User
StackOverflowError 💥

---

## ✅ Solution Using `@JsonIgnore`

```java
@ManyToOne
@JsonIgnore
private User user;
```

Now:

* User ➝ Orders
* Orders ➝ User (ignored in JSON)

✔ No infinite loop
✔ Clean JSON response

---

# 🔥 Serialization vs Deserialization Important Concept

`@JsonIgnore` applies to:

* Serialization (Object → JSON)
* Deserialization (JSON → Object)

Means:

* Field response me nahi aayega
* Client agar field bheje bhi, to ignore ho jayega

---

# 🔥 Alternative: Only Hide in Response

If you want:

* Allow deserialization
* But hide only in response

Use:

```java
@JsonProperty(access = JsonProperty.Access.WRITE_ONLY)
private String password;
```

✔ Client password bhej sakta hai
✔ Response me password nahi aayega

This is best practice for login/register APIs.

---

# 🎯 Interview Points (Must Remember)

If interviewer pooche:

> Difference between `@JsonIgnore` and `@JsonIgnoreProperties`?

| Annotation              | Use Case               |
| ----------------------- | ---------------------- |
| `@JsonIgnore`           | Single field ignore    |
| `@JsonIgnoreProperties` | Multiple fields ignore |

Example:

```java
@JsonIgnoreProperties({"password", "token"})
public class User { }
```

---

# 🧠 When NOT To Use `@JsonIgnore`

❌ When you need custom response
👉 Instead use DTO pattern (Best Practice)

You are already learning Spring Boot seriously, so remember:

✔ Entities ≠ API Response
✔ Always use DTO in production apps

---

# 🚀 Final Summary

`@JsonIgnore`:

* Hides field from JSON
* Prevents infinite recursion
* Protects sensitive data
* Used in serialization & deserialization

---
