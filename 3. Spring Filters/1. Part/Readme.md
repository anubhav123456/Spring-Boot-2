

---

## 🌐 **What is a Filter in Java / Spring?**

A **Filter** is a Java class that:

> Intercepts **every incoming request** and **outgoing response** in a web application.

It allows us to run logic:

* ✅ **Before request reaches Spring (Controller)**
* ✅ **Before response goes back to client**

---

## 📍 **Where Does a Filter Sit in the Flow?**

**Request Flow:**

```
Client
  ↓
Web Server (Tomcat / JBoss / Undertow)
  ↓
🚧 Filter(s)  ← YOU WRITE LOGIC HERE
  ↓
DispatcherServlet (Spring Front Controller)
  ↓
Handler Mapping
  ↓
Controller
  ↓
Business Logic
  ↓
Response
  ↑
🚧 Filter(s) again (on the way back)
  ↑
Client
```

👉 **Filter sits between Web Container and DispatcherServlet**

---

## ⚙️ **Filter is NOT Spring-specific**

Even though Spring uses filters a lot:

* Filter is part of **Servlet API**
* It belongs to **Web Container concept**
* Works in:

  * Tomcat
  * JBoss
  * Undertow
  * Any Servlet container

So Filters are **not a Spring feature**, but Spring heavily depends on them.

---

## 🔁 **Filter Chain**

* In real apps, we don’t have just one filter.
* Multiple filters are connected → called **Filter Chain**

```
Request → Filter1 → Filter2 → Filter3 → DispatcherServlet
```

Each filter decides:

* Pass request forward → `chain.doFilter(request, response)`
* OR block it ❌

---

## 🛡️ **Why Filters are VERY Popular?**

Because they are perfect for **cross-cutting concerns**:

| Use Case               | Why Filter?                         |
| ---------------------- | ----------------------------------- |
| 🔐 Authentication      | Check if user is logged in          |
| 🔑 Authorization       | Check if user has permission        |
| 🧾 Logging             | Log request & response              |
| ⏱ Performance tracking | Measure execution time              |
| 🌍 CORS handling       | Add CORS headers                    |
| 🧹 Centralized logic   | Avoid duplicate code in controllers |

---

## 🚫 **Filter Can Block a Request**

At any point, a filter can:

* Stop request from going further
* Return error response directly

Common cases:

* **401 Unauthorized** → User not logged in
* **403 Forbidden** → User logged in but not allowed

So the request **never reaches Controller**.

---

## 🧠 **Key Interview Points**

✔ Filter runs on **every request & response**
✔ It works at **container level**, before Spring MVC
✔ Spring Security is built heavily using **filters**
✔ Filters are part of **Servlet API**, not Spring core
✔ Multiple filters form a **Filter Chain**
✔ A filter can **terminate request early**

---

## 🆚 Filter vs Controller

| Filter                      | Controller                                 |
| --------------------------- | ------------------------------------------ |
| Runs before Spring MVC      | Runs inside Spring MVC                     |
| Handles cross-cutting logic | Handles business logic                     |
| Can block request           | Cannot stop request before reaching itself |
| Works for all URLs          | Works for mapped endpoints                 |

---

If interviewer asks:
**“Why do we use Filters instead of putting logic in controllers?”**

👉 Because Filters provide:

* Centralization
* Reusability
* Security enforcement before business logic

---

