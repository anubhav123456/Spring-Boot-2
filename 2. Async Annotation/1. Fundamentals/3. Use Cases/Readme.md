
---

# 🔥 SPRING `@Async` – INTERNAL

---

## ✅ Step 1: What `@Async` Actually Does

`@Async` ka matlab:

> Method **caller thread** pe execute nahi hoga.
> Spring us method ko **separate thread** me execute karega using an **Executor**.

But important question:

> **Ye new thread aata kahan se hai??**

Spring khud thread create nahi karta —
Spring **Executor (Thread Pool)** ko bolta hai:
👉 “Ye task run karo”

---

# 🧠 INTERNAL FLOW (Very Important)

When a method annotated with `@Async` is called:

```
Caller Thread  →  Spring Proxy  →  AsyncExecutionInterceptor
                →  Executor.submit(task)
                →  Thread from thread pool runs method
```

So:

🔹 `@Async` = **Proxy + Executor based execution**
🔹 It does NOT directly create threads

---

# ⚙️ HOW SPRING CHOOSES EXECUTOR

Spring ka internal logic:

```
1️⃣ Kya application me koi ThreadPoolTaskExecutor bean hai?
      YES → use that

2️⃣ Agar nahi hai → Spring Boot creates its DEFAULT ThreadPoolTaskExecutor

3️⃣ Agar even that not possible → fallback to SimpleAsyncTaskExecutor
```

So statement:

> ❌ "Spring always uses SimpleAsyncTaskExecutor by default"

**NOT FULLY CORRECT**.

Spring Boot mostly creates **its own default ThreadPoolTaskExecutor**.

---

# 🧩 USE CASE 1 — NO CUSTOM EXECUTOR

---

## 🔹 Code

### Main Application

```java
@SpringBootApplication
@EnableAsync
public class SpringbootApplication {

    public static void main(String args[]){
        SpringApplication.run(SpringbootApplication.class, args);
    }
}
```

---

### Config (Empty)

```java
@Configuration
public class AppConfig {

}
```

---

### Controller

```java
@RestController
@RequestMapping(value = "/api/")
public class UserController {

    @Autowired
    UserService userServiceObj;

    @GetMapping(path = "/getuser")
    public String getUserMethod(){
        System.out.println("inside getUserMethod: " + Thread.currentThread().getName() );
        userServiceObj.asyncMethodTest();
        return null;
    }
}
```

---

### Service

```java
@Component
public class UserService {

    @Async
    public void asyncMethodTest() {
        System.out.println("inside asyncMethodTest: " + Thread.currentThread().getName());
    }
}
```

---

## 🚀 What Happens Internally

During startup:

✔ Spring checks → **Is any ThreadPoolTaskExecutor bean present?**
❌ No

➡ Spring Boot creates **default ThreadPoolTaskExecutor**

### Default Config (Spring Boot)

| Property       | Value                 |
| -------------- | --------------------- |
| Core Pool Size | **8**                 |
| Max Pool Size  | **Integer.MAX_VALUE** |
| Queue Capacity | **Integer.MAX_VALUE** |

---

### Why thread names repeat (task1…task8…again task1)?

Because:

✔ Threads are reused from **pool of 8 core threads**

NOT new thread every time.

---

## ❌ Problems With Default Executor

| Problem           | Reason                                 |
| ----------------- | -------------------------------------- |
| Underutilization  | Huge queue → new threads never created |
| High latency      | Tasks wait in queue                    |
| Memory usage      | Too many queued tasks                  |
| Thread exhaustion | If max reached → crash possible        |

👉 **Not recommended for production**

---

# 🧩 USE CASE 2 — CUSTOM EXECUTOR (BEST PRACTICE)

---

### Main

```java
@SpringBootApplication
@EnableAsync
public class SpringbootApplication {

    public static void main(String args[]){
        SpringApplication.run(SpringbootApplication.class, args);
    }
}
```

---

### Custom Thread Pool

```java
@Configuration
public class AppConfig {

    @Bean(name = "myThreadPoolExecutor")
    public Executor taskPoolExecutor() {

        int minPoolSize = 2;
        int maxPoolSize = 4;
        int queueSize = 3;

        ThreadPoolTaskExecutor poolTaskExecutor = new ThreadPoolTaskExecutor();
        poolTaskExecutor.setCorePoolSize(minPoolSize);
        poolTaskExecutor.setMaxPoolSize(maxPoolSize);
        poolTaskExecutor.setQueueCapacity(queueSize);
        poolTaskExecutor.setThreadNamePrefix("MyThread-");
        poolTaskExecutor.initialize();
        return poolTaskExecutor;
    }
}
```

---

### Service

```java
@Component
public class UserService {

    @Async("myThreadPoolExecutor") // OR just @Async
    public void asyncMethodTest() {
        System.out.println("inside asyncMethodTest: " + Thread.currentThread().getName());
    }
}
```

---

## 🔥 What Happens Now

Spring startup:

✔ Finds `ThreadPoolTaskExecutor` bean
✔ Uses **your executor** as default
❌ Does NOT create its own

---

## 🎯 Thread Creation Flow (2 core, 4 max, queue=3)

| Step   | Action                     |
| ------ | -------------------------- |
| Task 1 | Thread-1 created           |
| Task 2 | Thread-2 created           |
| Task 3 | Goes to queue              |
| Task 4 | Goes to queue              |
| Task 5 | Goes to queue (queue full) |
| Task 6 | Thread-3 created           |
| Task 7 | Thread-4 created           |
| Task 8 | ❌ Rejected                 |

---

## 💡 Key Learning

| Myth                               | Reality                                    |
| ---------------------------------- | ------------------------------------------ |
| @Async creates thread              | Executor creates thread                    |
| Default is SimpleAsyncTaskExecutor | Spring Boot creates ThreadPoolTaskExecutor |
| @Async always new thread           | Threads reused from pool                   |

---

# 🧠 FINAL ARCHITECTURE

```
@Async Method
     ↓
Spring Proxy
     ↓
AsyncExecutionInterceptor
     ↓
Executor.submit()
     ↓
ThreadPoolTaskExecutor
     ↓
Java ThreadPoolExecutor
     ↓
Worker Thread runs method
```

---

# ✅ INTERVIEW GOLD LINE

> “@Async does not create threads directly. It delegates execution to an Executor. Spring Boot auto-configures a ThreadPoolTaskExecutor if none is provided, otherwise it uses the user-defined one.”

---
