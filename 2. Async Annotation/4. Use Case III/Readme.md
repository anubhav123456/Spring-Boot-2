
---

# 🚀 USE CASE 3

**Custom Plain Java `ThreadPoolExecutor` + `@Async`**

In this case, we are **NOT** using Spring’s `ThreadPoolTaskExecutor`.
Instead, we define **Java’s own** `ThreadPoolExecutor` as a bean.

Here is your **complete working setup**:

---

## 🔹 1. Main Application

```java
@SpringBootApplication
@EnableAsync
public class SpringbootApplication {

    public static void main(String args[]){
        SpringApplication.run(SpringbootApplication.class, args);
    }
}
```

`@EnableAsync` activates Spring’s async processing.

---

## 🔹 2. Custom Java Executor Bean (IMPORTANT)

```java
@Configuration
public class AppConfig {

    @Bean(name = "javaExecutor")
    public Executor javaExecutor() {

        AtomicInteger count = new AtomicInteger(1);

        return new ThreadPoolExecutor(
                2,   // core threads
                4,   // max threads
                60,  // idle time
                TimeUnit.SECONDS,
                new LinkedBlockingQueue<>(50),   // queue capacity
                r -> new Thread(r, "JavaThread-" + count.getAndIncrement()),
                new ThreadPoolExecutor.CallerRunsPolicy() // rejection policy
        );
    }
}
```

This is **pure Java thread pool**, not Spring’s.

---

## 🔹 3. Controller

```java
@RestController
@RequestMapping(value = "/api/")
public class UserController {

    @Autowired
    UserService userServiceObj;

    @GetMapping(path = "/getuser")
    public String getUserMethod(){
        System.out.println("inside getUserMethod: " + Thread.currentThread().getName());
        userServiceObj.asyncMethodTest();
        return "Request Accepted";
    }
}
```

This runs on a **Tomcat request thread** like:

```
http-nio-8080-exec-1
```

---

## 🔹 4. Async Service

```java
@Component
public class UserService {

    @Async("javaExecutor")   // 🔥 MUST specify name
    public void asyncMethodTest() {
        System.out.println("inside asyncMethodTest: " + Thread.currentThread().getName());
    }
}
```

---

# 🧠 What Happens Internally

### ✅ Step 1 — Spring Startup

Spring sees:

> “Oh, there is an `Executor` bean defined (`javaExecutor`).”

So Spring **does NOT create its own default `ThreadPoolTaskExecutor`.**

BUT — Spring still does **not automatically connect this executor to `@Async`.**

---

### ✅ Step 2 — API Request Comes

Request hits:

```
inside getUserMethod: http-nio-8080-exec-1
```

This is the **main request thread**.

---

### ✅ Step 3 — Async Method Called

Because of:

```java
@Async("javaExecutor")
```

Spring’s async proxy intercepts the call and does:

```java
javaExecutor.execute(task);
```

Task goes to **your custom thread pool**.

Output:

```
inside asyncMethodTest: JavaThread-1
```

Now the method runs in a **separate thread**.

---

# ❗ Why the Executor Name is MANDATORY Here

If you wrote:

```java
@Async
```

Spring’s lookup order:

1. Is there a `TaskExecutor` bean? ❌
2. Is there a default from `AsyncConfigurer`? ❌
3. Use fallback →

💀 `SimpleAsyncTaskExecutor`

Your custom executor would be ignored.

That’s why in **Use Case 3**, this is required:

```java
@Async("javaExecutor")
```

---

# 🔥 What Happens When Pool Is Full?

Your pool:

| Setting      | Value |
| ------------ | ----- |
| Core threads | 2     |
| Max threads  | 4     |
| Queue size   | 50    |

If:

* 4 threads busy
* Queue full

Then this kicks in:

```java
new ThreadPoolExecutor.CallerRunsPolicy()
```

Meaning:

> The **caller thread** (controller thread) will run the task.

So you may see:

```
inside asyncMethodTest: http-nio-8080-exec-1
```

This prevents crashes and slows the system gracefully. This is called **backpressure**.

---

# 📊 Execution Flow

```
Client Request
      ↓
Tomcat Thread (http-nio)
      ↓
Controller
      ↓
@Async Proxy
      ↓
javaExecutor Thread Pool
      ↓
JavaThread-* runs method
```

---

# 🎯 Use Case 3 in One Line

> If you define a plain Java `ThreadPoolExecutor` bean, Spring will not automatically wire it to `@Async`. You must specify the executor name in `@Async`, or Spring falls back to `SimpleAsyncTaskExecutor`.

---

