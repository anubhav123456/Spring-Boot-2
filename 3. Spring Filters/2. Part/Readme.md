
---

# 🧠 **Spring Boot – Creating a Servlet Filter (Notes)**

## ✅ 1. Why do we annotate Filter with `@Component`?

Because:

✔ Spring needs to **detect and register** the filter as a bean
✔ Otherwise filter won’t be part of the application context
✔ And it will **never run**

> 💥 Most common mistake: Forgetting `@Component`

---

## ✅ 2. Which interface do we implement?

```
jakarta.servlet.Filter
```

But ⚠ version matters:

| Spring Boot Version  | Package Used        |
| -------------------- | ------------------- |
| **Spring Boot 3.x+** | `jakarta.servlet.*` |
| **Spring Boot 2.x**  | `javax.servlet.*`   |

👉 Spring 3 migrated from **Javax → Jakarta**

---

# 🧩 3. Methods in `Filter` Interface

```java
void init(FilterConfig filterConfig)
void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
void destroy()
```

### 🔹 `init()`

Called when filter is **put into service** by container.

### 🔹 `destroy()`

Called when filter is **removed from service**.

👉 Rarely used.

### 🔥 `doFilter()` (MOST IMPORTANT)

This method:

* Intercepts every request
* Lets you:

  * Log
  * Authenticate
  * Modify request/response
  * Block request

---

# 🚀 4. Problem: `ServletRequest` doesn’t have headers

So we must cast:

```java
HttpServletRequest httpRequest = (HttpServletRequest) request;
```

---

# 🔍 5. Getting All Headers

There is no Map of headers. We use:

```java
Enumeration<String> headerNames = httpRequest.getHeaderNames();
```

Convert to list:

```java
Collections.list(headerNames)
```

---

# ⚠️ 6. VERY IMPORTANT — Filter Chain

If you don’t call:

```java
chain.doFilter(request, response);
```

👉 Request processing **stops**
👉 No controller is called
👉 No response returned

---

# 💻 FULL FILTER CODE

## ✅ For Spring Boot 3.x (Jakarta)

```java
package com.example.demo.filter;

import jakarta.servlet.Filter;
import jakarta.servlet.FilterChain;
import jakarta.servlet.ServletException;
import jakarta.servlet.ServletRequest;
import jakarta.servlet.ServletResponse;
import jakarta.servlet.http.HttpServletRequest;

import org.springframework.stereotype.Component;

import java.io.IOException;
import java.util.Collections;

@Component
public class LoggingFilter implements Filter {

    @Override
    public void init(jakarta.servlet.FilterConfig filterConfig) throws ServletException {
        System.out.println("Filter initialized");
    }

    @Override
    public void doFilter(ServletRequest request,
                         ServletResponse response,
                         FilterChain chain) throws IOException, ServletException {

        HttpServletRequest httpRequest = (HttpServletRequest) request;

        Collections.list(httpRequest.getHeaderNames())
                .forEach(headerName -> {
                    String headerValue = httpRequest.getHeader(headerName);
                    System.out.println("Header: " + headerName + " = " + headerValue);
                });

        // VERY IMPORTANT: continue request processing
        chain.doFilter(request, response);
    }

    @Override
    public void destroy() {
        System.out.println("Filter destroyed");
    }
}
```

---

## 🧓 For Spring Boot 2.x (Javax)

Only change imports:

```java
import javax.servlet.*;
import javax.servlet.http.HttpServletRequest;
```

Everything else stays SAME.

---

# 🔄 Request Flow With Filter

```
Client → Filter → Controller → Response → Filter → Client
```

If `chain.doFilter()` is missing:

```
Client → Filter ❌ BLOCKED
```

---

# 🎯 Interview Points

✔ Filters work at **Servlet container level**
✔ Used for:

* Logging
* Security
* CORS
* Rate limiting
* Request validation

✔ Difference from Interceptor:

| Filter                   | Interceptor             |
| ------------------------ | ----------------------- |
| Servlet level            | Spring MVC level        |
| Before DispatcherServlet | After DispatcherServlet |
| Can modify request       | Mostly handler logic    |

---

# 🧠 Key Takeaway

> **Filter intercepts requests before controller, and MUST pass request to filter chain or the request dies there.**

---
