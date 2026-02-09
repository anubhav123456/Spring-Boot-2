

---

# 🚀 Async Method Return Type in Spring Boot (Future)

## 🧠 Why do we need Future / CompletableFuture?

When we use:

```java
@Async
```

➡ Spring runs that method in a **separate thread** (background thread).

So now:

| Thread       | Work                 |
| ------------ | -------------------- |
| Main Thread  | Continues execution  |
| Async Thread | Does heavy/slow task |

But question is:

👉 **How will the main thread get the result produced by async thread?**

That’s where **Future / CompletableFuture** comes in.

---

# ⚙️ Flow Understanding

```
Main Thread  ----calls---->  @Async Method  ----runs in----> New Thread
     |                                              |
     |<----------- Future Object (reference) -------|
```

Main thread doesn’t wait immediately.
It only waits when we call:

```java
future.get()
```

---

# 🧩 Controller

```java
@RestController
@RequestMapping(value = "/api/")
public class UserController {

    @Autowired
    UserService userService;

    @GetMapping(path = "/getuser")
    public String getUserMethod() {

        Future<String> result = userService.performTaskAsync();
        String output = null;

        try {
            output = result.get();   // Main thread waits here
            System.out.println(output);
        } catch (Exception e) {
            System.out.println("some exception");
        }

        return output;
    }
}
```

### 🔍 What’s happening here?

1. Controller calls async service.
2. Immediately gets **Future reference**.
3. When `.get()` is called:

   * Main thread **blocks**
   * Waits until async task finishes
4. Returns the value.

---

# 🧩 Service

```java
@Component
public class UserService {

    @Async
    public Future<String> performTaskAsync() {
        return new AsyncResult<>("async task result");
    }
}
```

---

# 🧠 What `@Async` Does

* Creates **new thread**
* Executes method there
* Does **not block main thread**
* But if you call `.get()` → blocking happens

---

# 📦 What is `AsyncResult`?

```java
new AsyncResult<>("async task result");
```

This wraps the result inside a **Future** object.

---

# 📘 Methods in `Future` Interface

| Method                                  | Purpose                                     |
| --------------------------------------- | ------------------------------------------- |
| `cancel(boolean mayInterruptIfRunning)` | Tries to cancel task                        |
| `isCancelled()`                         | Returns true if cancelled                   |
| `isDone()`                              | True if task finished (success/fail/cancel) |
| `get()`                                 | Waits and returns result                    |
| `get(timeout, TimeUnit)`                | Waits only for given time                   |

---

# ⛔ Important Industry Note

👉 `Future` is **old style**
👉 Now mostly replaced with:

```
CompletableFuture
```

Because it supports:

* Chaining
* Non-blocking processing
* Multiple async combinations
* Better exception handling

---

# 🧵 Thread Behavior Summary

| Situation           | Behavior              |
| ------------------- | --------------------- |
| Call async method   | New thread starts     |
| Store in `Future`   | Just a reference      |
| Call `future.get()` | Main thread **waits** |
| No `get()` call     | Main thread continues |

---

# 🧠 Key Interview Line

> “`@Async` runs method in separate thread, and to capture the result we use `Future` or `CompletableFuture`. Calling `get()` makes the caller thread wait until completion.”

---

# 🔥 Real Meaning in Simple Words

Imagine:

🧍 You = Main Thread
👨‍🍳 Cook = Async Thread

You order food → you get **token (Future)**
You don’t wait at counter.
Only when you show token to collect food → you wait.

---
