
---

# 🧠 What is `@Async` in Spring Boot?

`@Async` is used to run a method **asynchronously** — meaning:

> The method runs in a **separate thread**
> and the **main request thread does NOT wait** for it to finish.

So:

| Type             | Behavior                                                |
| ---------------- | ------------------------------------------------------- |
| **Synchronous**  | Caller waits until method finishes                      |
| **Asynchronous** | Caller continues immediately, method runs in background |

---

# 🚦 How Spring Normally Works (Without `@Async`)

1. App starts → runs on **main thread**
2. HTTP request comes → handled by **request thread**
3. Method is called → runs on **same thread**
4. Thread stays **blocked** until method completes

---

# ⚡ What `@Async` Changes

When Spring sees `@Async`:

* It **does NOT run method in the caller thread**
* Instead, it submits the task to a **Thread Pool**
* Caller thread becomes **free immediately**

---

# 🏁 Step 1 — Enable Async in Spring Boot

```java
@SpringBootApplication
@EnableAsync   // 🔥 Tells Spring to enable async processing
public class SpringbootApplication {

    public static void main(String args[]){
        SpringApplication.run(SpringbootApplication.class, args);
    }
}
```

### 🧩 Why `@EnableAsync` is required?

Spring needs to create internal beans like:

* Async proxy
* Task executor
* Interceptors

Without this annotation → `@Async` **will not work**

---

# 🎮 Step 2 — Controller

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
        return "Request Completed";
    }
}
```

👉 This method runs on **request thread** (like `http-nio-8080-exec-1`)

---

# ⚙️ Step 3 — Service with `@Async`

```java
@Component
public class UserService {

    @Async
    public void asyncMethodTest() {
        System.out.println("inside asyncMethodTest: " + Thread.currentThread().getName());
    }
}
```

👉 This method runs in **separate thread** from a **thread pool**

---

# 🧵 Thread Flow Diagram

```
Client Request
      |
      v
Controller (Main Request Thread)
      |
      | calls async method
      v
Async Thread Pool Thread  <-- runs asyncMethodTest()
```

---

# 🖥️ OUTPUT (First Request)

```
inside getUserMethod: http-nio-8080-exec-1
inside asyncMethodTest: task-1
```

| Line             | Thread           |
| ---------------- | ---------------- |
| Controller log   | Request thread   |
| Async method log | Different thread |

---

# 🖥️ OUTPUT (Multiple Requests)

```
inside getUserMethod: http-nio-8080-exec-2
inside asyncMethodTest: task-2

inside getUserMethod: http-nio-8080-exec-3
inside asyncMethodTest: task-3

inside getUserMethod: http-nio-8080-exec-4
inside asyncMethodTest: task-4
```

---

# ❗ IMPORTANT CLARIFICATION (Very Interview Important)

You said:

> "Every time new thread is created"

🛑 **Not exactly true**

Spring does **NOT create infinite threads**.
It uses a **Thread Pool** (like a reusable worker team).

So:

* Threads like `task-1`, `task-2` are **reused**
* Pool manages performance

---

# 🧠 What Actually Happens Internally

When you call:

```java
userServiceObj.asyncMethodTest();
```

Spring does:

1. Creates **proxy** of `UserService`
2. Intercepts call to `asyncMethodTest()`
3. Submits task to **TaskExecutor**
4. Method runs in **background thread**

---

# 🧨 Why Main Thread Doesn't Wait?

Because method is handed off to executor:

```text
Controller Thread: "Okay async task submitted. I'm free now ✌️"
```

So response returns fast.

---

# 🎯 Real Life Use Cases

| Use Case             | Why Async?          |
| -------------------- | ------------------- |
| Sending Email        | User shouldn’t wait |
| Logging              | Background task     |
| Report generation    | Heavy task          |
| Payment notification | Fire-and-forget     |

---

# ⚠️ Important Rules of `@Async`

| Rule                                               | Explanation          |
| -------------------------------------------------- | -------------------- |
| Must use `@EnableAsync`                            | Otherwise won't work |
| Method must be `public`                            | Proxy requirement    |
| Should not call async method **inside same class** | Proxy won't trigger  |
| Return type can be `Future`, `CompletableFuture`   | For result handling  |

---

# 🔥 Final Summary

| Thing                | Thread             |
| -------------------- | ------------------ |
| Controller method    | Request thread     |
| `@Async` method      | Thread pool thread |
| Main thread blocked? | ❌ No               |
| Performance benefit? | ✅ Yes              |

---
