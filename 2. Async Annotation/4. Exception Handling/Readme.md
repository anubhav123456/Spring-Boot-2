
---

# 🔥 Why Exception Handling is Important in `@Async`?

`@Async` method **separate thread** me run hota hai.

👉 Matlab:

* Main thread method call karta hai
* Aur turant aage badh jata hai
* Agar async thread me exception aaya → **main thread ko automatically pata nahi chalega**

Isliye handling **method return type** par depend karti hai.

---

# 🧠 CASE 1: Async Method **HAS RETURN TYPE**

Yahaan hum `Future` / `CompletableFuture` use karte hain.

Exception **tab throw hoti hai jab `.get()` call karte ho**.

---

## ✅ Controller

```java
@RestController
@RequestMapping(value = "/api/")
public class UserController {

    @Autowired
    UserService userService;

    @GetMapping(path = "/getuser")
    public String getUserMethod() {

        CompletableFuture<String> result = userService.performTaskAsync();
        String output = null;

        try {
            output = result.get();   // 🔥 Exception yahi throw hogi
            System.out.println(output);
        } catch (Exception e) {
            System.out.println("some exception");
        }

        return output;
    }
}
```

---

## ✅ Service

```java
@Component
public class UserService {

    @Async
    public CompletableFuture<String> performTaskAsync() {

        return CompletableFuture.completedFuture("async task result");
    }
}
```

---

### 💡 What Happens Here?

| Step                   | Explanation              |
| ---------------------- | ------------------------ |
| `@Async` method called | Runs in new thread       |
| Main thread continues  | Doesn't wait             |
| `result.get()` called  | Now main thread waits    |
| If exception occurred  | It is thrown at `.get()` |
| We can catch & handle  | Logging / rethrow etc    |

---

# ⚡ CASE 2: Async Method **RETURNS VOID**

Ab dikkat shuru hoti hai 😈

```java
@Async
public void performTaskAsync() { }
```

Yahaan:

* No Future
* No `.get()`
* Exception **caller tak nahi pahunchti**

---

## ❌ Controller

```java
@RestController
@RequestMapping(value = "/api/")
public class UserController {

    @Autowired
    UserService userService;

    @GetMapping(path = "/getuser")
    public String getUserMethod() {

        userService.performTaskAsync();  // Fire and forget
        return "";
    }
}
```

---

## ❌ Service (Problem)

```java
@Component
public class UserService {

    @Async
    public void performTaskAsync() {
        // perform some task
    }
}
```

Agar yaha:

```java
int x = 1 / 0;
```

👉 Exception async thread me crash ho jayegi
👉 Main thread ko pata hi nahi chalega

---

# 🛠 HOW TO HANDLE THIS?

## ✅ WAY 1: Handle Exception **Inside Async Method** (Most Common)

```java
@Component
public class UserService {

    @Async
    public void performTaskAsync() {

        try {
            // perform some task
            int x = 1 / 0;
        } catch (Exception e) {
            // handle the exception here
            System.out.println("Exception occurred: " + e.getMessage());
        }
    }
}
```

✔ Logging
✔ Retry
✔ Alert system
✔ DB entry

Sab yahi karna padega.

---

## ✅ WAY 2 (Advanced – Interview Bonus 💥): Global Handler for Async

Spring provides:

### Step 1: Create Config

```java
@Configuration
@EnableAsync
public class AsyncConfig implements AsyncConfigurer {

    @Override
    public Executor getAsyncExecutor() {
        return Executors.newCachedThreadPool();
    }

    @Override
    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
        return new CustomAsyncExceptionHandler();
    }
}
```

---

### Step 2: Custom Exception Handler

```java
public class CustomAsyncExceptionHandler implements AsyncUncaughtExceptionHandler {

    @Override
    public void handleUncaughtException(Throwable ex, Method method, Object... params) {
        System.out.println("Exception in async method: " + method.getName());
        System.out.println("Exception message: " + ex.getMessage());
    }
}
```

🔥 Ye **sirf VOID returning @Async methods** ke liye kaam karta hai.

---

# 🎯 Interview Summary (Important)

| Scenario                 | Exception Handling                  |
| ------------------------ | ----------------------------------- |
| `@Async` + Return Type   | Catch using `future.get()`          |
| `@Async` + void          | Must handle inside method           |
| `@Async` + void (Global) | Use `AsyncUncaughtExceptionHandler` |

---

# 💬 One Line Interview Answer

> "If an `@Async` method returns a Future, exceptions are propagated when calling `.get()`. But for void-returning async methods, exceptions are not propagated to the caller thread, so they must be handled inside the method or using a custom `AsyncUncaughtExceptionHandler`."

---
