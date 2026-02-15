

---

# ✅ What is First Level Cache in JPA?

In **Jakarta Persistence (JPA)**, First Level Cache is:

> A cache that exists **inside the Persistence Context** and is enabled by default.

It is also called:

👉 **Persistence Context Cache**
👉 **Session Cache**

If you're using Hibernate (which Spring Boot uses by default), it’s managed inside:

* **Hibernate ORM**
* via `EntityManager`
* internally via Hibernate `Session`

---

# 🧠 Where Does It Live?

Each time you use:

```java
@Transactional
```

Spring creates a **Persistence Context**.

Inside that Persistence Context → there is a **First Level Cache**.

⚠ Important:

* It is **per transaction**
* It is **not shared**
* It is destroyed when transaction ends

---

# 🏦 Real World Example (Bank Locker Example)

Imagine:

You go to a bank.

* The bank vault = Database
* Your personal locker = First Level Cache
* Bank manager = EntityManager
* Bank system = Hibernate

Now:

1️⃣ First time you request your file → Manager goes to vault (DB)
2️⃣ File is put in your locker (First Level Cache)
3️⃣ If you request same file again → Manager gives it from locker
4️⃣ No need to go to vault again

💥 That locker is your First Level Cache.

---

# 🔍 Technical Flow Example

Let’s say we have:

```java
@Entity
public class User {
    @Id
    private Long id;
    private String name;
}
```

Now in service:

```java
@Transactional
public void test() {
    User user1 = entityManager.find(User.class, 1L);
    User user2 = entityManager.find(User.class, 1L);
}
```

---

## 🧾 What Happens Internally?

### Step 1:

```java
entityManager.find(User.class, 1L);
```

* JPA checks First Level Cache
* Not found
* Executes SQL:

```sql
select * from user where id=1;
```

* Stores entity in Persistence Context

---

### Step 2:

```java
entityManager.find(User.class, 1L);
```

Now:

* JPA checks First Level Cache
* Finds entity already there
* DOES NOT HIT DATABASE
* Returns same object reference

---

# 🔥 Important Proof

If you log:

```java
System.out.println(user1 == user2);
```

Output:

```
true
```

Why?

Because both references point to SAME object in Persistence Context.

---

# 📊 Diagram Flow

```
Application
     ↓
EntityManager
     ↓
Persistence Context (First Level Cache)
     ↓
Database (Only if needed)
```

---

# 🚀 Why Is This Powerful?

### 1️⃣ Prevents Multiple DB Hits

Improves performance

### 2️⃣ Ensures Data Consistency

Within a transaction, same entity = same object

### 3️⃣ Enables Dirty Checking

If you modify entity:

```java
user1.setName("Anubhav");
```

You don't even call `save()`.

At transaction commit:

Hibernate automatically detects change and runs:

```sql
update user set name='Anubhav' where id=1;
```

This feature is called:

👉 Dirty Checking (very important interview question)

---

# ⚠ When Does First Level Cache NOT Work?

## ❌ Case 1: New Transaction

```java
@Transactional
public void method1() {
    entityManager.find(User.class, 1L);
}

@Transactional
public void method2() {
    entityManager.find(User.class, 1L);
}
```

These are different transactions.

So database will be hit twice.

---

## ❌ Case 2: You Clear the Cache

```java
entityManager.clear();
```

This removes everything from Persistence Context.

Next `find()` will hit DB again.

---

## ❌ Case 3: You Use Native Query

Sometimes native queries bypass cache.


---

# 🧠 Interview Questions They May Ask You

1. Is First Level Cache enabled by default?
   → Yes

2. Can we disable it?
   → Not directly (you can clear it manually)

3. Is it shared across users?
   → No

4. Is it thread-safe?
   → EntityManager is not thread-safe

5. Does Spring Boot use Hibernate by default?
   → Yes

---

# 💥 Most Important Line

> First Level Cache = Persistence Context = One Transaction = One Memory Map of Entities

---

# 🧘 Simple One-Line Memory Trick

Think:

> "Transaction ke andar sab memory me hi chalega, DB ko disturb mat karo baar baar."

---


