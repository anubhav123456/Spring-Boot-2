
---

# ✅ Prerequisites for Understanding `@Async`

Before `@Async`, you must know:

1. **Thread creation & life cycle**
2. **Thread Pool & ThreadPoolExecutor**
3. **Future, Callable, CompletableFuture**

---

# 🧵 What is a Thread Pool?

**Definition:**
A **Thread Pool** is a collection of **pre-created threads** that are reused to execute tasks.

### Why Thread Pool?

* Creating threads is expensive 💸
* Reusing threads = better performance 🚀
* Controls resource usage (CPU, memory)

### Flow:

```
Application → submits task → Queue → Thread picks task → Executes → Returns to pool
```

After finishing a task, the thread **does not die** ❌
It goes back to the pool and waits for the next task ✅

---

# 🧠 ThreadPoolExecutor – Core Concepts

ThreadPoolExecutor controls how threads behave.

### 🔑 3 Most Important Parameters

| Parameter                              | Meaning                             |
| -------------------------------------- | ----------------------------------- |
| **Core Pool Size (Minimum Pool Size)** | Minimum threads always kept alive   |
| **Maximum Pool Size**                  | Maximum threads that can be created |
| **Queue Size**                         | How many tasks can wait in queue    |

---

# 📦 Components Involved

1. **Thread Pool** → Contains worker threads
2. **Task Queue** → Holds waiting tasks
3. **Application** → Submits tasks

---

# ⚙️ Example Configuration

* Core Pool Size = **2**
* Max Pool Size = **4**
* Queue Size = **3**

---

# 🔄 How ThreadPoolExecutor Works (Step-by-Step)

### Initial State

* 2 threads created → `Thread-1`, `Thread-2` (idle)
* Queue capacity = 3 tasks

---

### 🟢 Task 1 arrives

* Picked by Thread-1
* Thread-1 becomes busy

### 🟢 Task 2 arrives

* Picked by Thread-2
* Thread-2 becomes busy

➡ Now **all core threads are busy**

---

### 🟡 Task 3 arrives

* No thread free → Goes to Queue

### 🟡 Task 4 arrives

* Queue has space → Added

### 🟡 Task 5 arrives

* Queue full after this

---

### 🔥 Task 6 arrives

* No free thread
* Queue full
* Check max pool size (4 allowed, currently 2)
* Create **Thread-3**
* Task 6 assigned to Thread-3

---

### 🔥 Task 7 arrives

* No free thread
* Queue full
* Max not reached (3 < 4)
* Create **Thread-4**
* Task 7 assigned to Thread-4

---

### ❌ Task 8 arrives

* No free thread
* Queue full
* Max pool size reached (4/4)
* **Task REJECTED** 🚫

---

### 🔄 When a thread finishes

Example: Thread-1 completes Task 1

* Thread-1 goes back to pool
* Picks next task from queue (Task 3)

---

# 📊 Execution Order Logic

1. Use **core threads first**
2. Then **fill queue**
3. Then **create extra threads (up to max)**
4. If queue full + max threads reached → **Reject task**

---

# ⏳ Other Important Terms

| Term                | Meaning                                                         |
| ------------------- | --------------------------------------------------------------- |
| **Keep Alive Time** | Extra threads (beyond core) are destroyed if idle for this time |
| **Time Unit**       | Unit for keep alive time (seconds, ms, etc.)                    |

---

# 🧩 How This Connects to `@Async`

Spring’s `@Async` internally uses a **ThreadPoolTaskExecutor**
Which is built on top of **ThreadPoolExecutor**

So when you use `@Async`, these decide:

* How many async tasks run together
* Whether tasks wait
* Whether tasks get rejected

---

# 🎯 Interview Summary (Golden Lines)

* Thread pool = reusable threads for task execution
* ThreadPoolExecutor manages **core size, max size, and queue**
* Order of execution:
  **Core Threads → Queue → Extra Threads → Reject**
* Improves performance and resource control

---


