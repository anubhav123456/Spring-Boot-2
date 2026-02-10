
---

# 🌪️ Spring Filters – Order of Filters (Super Important Topic)

Jab request aati hai, wo **direct controller pe nahi jaati**. Pehle wo **filter chain** se pass hoti hai:

```
Client → Filter 1 → Filter 2 → Filter 3 → DispatcherServlet → Controller
```

Aur response wapas aate time **reverse order** me jata hai:

```
Controller → Filter 3 → Filter 2 → Filter 1 → Client
```

👉 Isliye filter order matter karta hai **A LOT**.

---

## 🧠 Why Filter Order Matters?

Socho tumhare paas ye filters hain:

| Filter           | Kaam              |
| ---------------- | ----------------- |
| LoggingFilter    | Request log karna |
| AuthFilter       | JWT token check   |
| AdminCheckFilter | Role check        |

Agar order galat hua:

❌ Role check pehle aur authentication baad me
→ Role kaise check karega jab user hi verify nahi hua?

Correct flow hona chahiye:

```
1. Logging
2. Authentication
3. Authorization
```

---

# 🥇 Ways to Set Filter Order in Spring Boot

Spring Boot me filters order karne ke **3 main tareeke** hote hain.

---

## ✅ **1. Using `@Order` Annotation (Easiest)**

```java
@Component
@Order(1)
public class LoggingFilter implements Filter {
    public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
            throws IOException, ServletException {
        System.out.println("Logging Filter");
        chain.doFilter(req, res);
    }
}
```

```java
@Component
@Order(2)
public class AuthFilter implements Filter {
    public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
            throws IOException, ServletException {
        System.out.println("Auth Filter");
        chain.doFilter(req, res);
    }
}
```

🔹 **Smaller number = runs earlier**

| Order Value | Position   |
| ----------- | ---------- |
| 1           | First      |
| 2           | After 1    |
| 100         | Much later |

---

## ✅ **2. Using `FilterRegistrationBean` (Most Powerful)**

Yeh tab use karte hain jab:

* URL pattern control chahiye
* Specific servlet ke liye filter chahiye
* Full manual control

```java
@Configuration
public class FilterConfig {

    @Bean
    public FilterRegistrationBean<LoggingFilter> loggingFilter() {
        FilterRegistrationBean<LoggingFilter> reg = new FilterRegistrationBean<>();
        reg.setFilter(new LoggingFilter());
        reg.setOrder(1);   // 👈 order here
        reg.addUrlPatterns("/*");
        return reg;
    }

    @Bean
    public FilterRegistrationBean<AuthFilter> authFilter() {
        FilterRegistrationBean<AuthFilter> reg = new FilterRegistrationBean<>();
        reg.setFilter(new AuthFilter());
        reg.setOrder(2);
        reg.addUrlPatterns("/*");
        return reg;
    }
}
```

👉 This **overrides `@Order`**

---

## ✅ **3. Spring Security Filters (Special Case)**

Spring Security apna **separate internal filter chain** banata hai.

Order hota hai something like:

```
SecurityContextPersistenceFilter
UsernamePasswordAuthenticationFilter
ExceptionTranslationFilter
FilterSecurityInterceptor
```

Custom filter add karna ho toh:

```java
http.addFilterBefore(new JwtFilter(), UsernamePasswordAuthenticationFilter.class);
```

OR

```java
http.addFilterAfter(new CustomFilter(), BasicAuthenticationFilter.class);
```

Yeh normal filter order se **different system** hai.

---

# 🔄 Execution Flow Example

Assume:

| Filter        | Order |
| ------------- | ----- |
| LoggingFilter | 1     |
| JwtFilter     | 2     |
| RoleFilter    | 3     |

### Request Flow:

```
Logging → JWT → Role → Controller
```

### Response Flow:

```
Controller → Role → JWT → Logging
```

---

# ⚠️ Common Mistakes

❌ Forgetting order → unpredictable behavior
❌ Using both `@Order` & `FilterRegistrationBean` wrongly
❌ JWT filter after security filter (token useless ho jayega)

---

# 🧩 Real-World Best Practice Order

| Order | Filter               |
| ----- | -------------------- |
| 1     | CORS                 |
| 2     | Logging              |
| 3     | Authentication (JWT) |
| 4     | Authorization        |
| 5     | Business filters     |

---

# 🔥 Golden Rule

> **Lower order number = higher priority = earlier execution**

---
