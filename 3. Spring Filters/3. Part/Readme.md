
---

# 🌊 Filters in Spring — OncePerRequestFilter Notes

## 🧠 How Request Flows in Spring

```
Client → Filter Chain → DispatcherServlet → Controller
```

A request goes through **multiple filters** before reaching your controller.

---

## ❗ Problem with Normal Filters

If you implement:

```java
implements Filter
```

Your filter may run **multiple times for the same request** because of:

* FORWARD dispatch
* INCLUDE dispatch
* ERROR dispatch
* ASYNC dispatch

Most of the time we want:

> ✅ **Filter logic should run only once per request**

---

## ✅ Spring Solution: `OncePerRequestFilter`

Spring provides:

```
org.springframework.web.filter.OncePerRequestFilter
```

It ensures:

> 🔥 Your filter executes **only once per HTTP request**

---

## 🆚 Normal Filter vs OncePerRequestFilter

| Normal Filter                | OncePerRequestFilter          |
| ---------------------------- | ----------------------------- |
| Implements `Filter`          | Extends a Spring class        |
| Override `doFilter()`        | Override `doFilterInternal()` |
| Uses `ServletRequest`        | Uses `HttpServletRequest`     |
| Might run multiple times     | Runs once per request         |
| Needs `init()` & `destroy()` | Not required                  |

---

# 🧩 COMPLETE WORKING CODE EXAMPLE

### ✅ 1️⃣ Main Spring Boot Application

```java
package com.example.filtersdemo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class FiltersDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(FiltersDemoApplication.class, args);
    }
}
```

---

### ✅ 2️⃣ Controller (To Test the Filter)

```java
package com.example.filtersdemo.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        return "Hello from Controller!";
    }
}
```

---

### ✅ 3️⃣ The Filter (FROM IMAGE + COMPLETE)

```java
package com.example.filtersdemo.filter;

import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;
import org.springframework.web.filter.OncePerRequestFilter;

import jakarta.servlet.FilterChain;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

import java.io.IOException;
import java.util.Collections;

@Component
@Slf4j
public class HeadersLoggingFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(
            final HttpServletRequest request,
            final HttpServletResponse response,
            final FilterChain chain)
            throws ServletException, IOException {

        log.info("🔵 Filter Executed — Logging Request Headers");

        Collections.list(request.getHeaderNames())
                .forEach(header ->
                        log.info("Header: {} = {}", header, request.getHeader(header))
                );

        // VERY IMPORTANT — continue the filter chain
        chain.doFilter(request, response);
    }
}
```

---

# 🚀 What Happens When You Call:

```
GET http://localhost:8080/hello
```

### 🧾 Console Output:

```
🔵 Filter Executed — Logging Request Headers
Header: host = localhost:8080
Header: user-agent = PostmanRuntime/7.36.0
Header: accept = */*
```

### 🌐 Response:

```
Hello from Controller!
```

---

## 🔑 Important Line

```java
chain.doFilter(request, response);
```

Without this:

❌ Request never reaches controller
❌ Client hangs

---

## 🧠 Where This Is Used in Real Systems

* JWT Authentication Filters
* Logging Filters
* Rate Limiting
* Request Tracing
* Security Checks

---

## 🏁 Final Interview Definition

> **OncePerRequestFilter is a Spring-provided filter base class that guarantees a filter runs only once per HTTP request, preventing duplicate execution during internal dispatches like forward, include, async, or error handling.**

---

