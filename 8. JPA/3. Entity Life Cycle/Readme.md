
---

# 🔄 Life Cycle of Entity in Persistence Context (JPA)

![Life Cycle of Entity in Persistence Context](Cycle.jpg)

## 🧠 Sabse Pehle Important Concept

Jab bhi tum:

```java
EntityManager em = entityManagerFactory.createEntityManager();
```

create karte ho,
➡️ internally ek **Persistence Context** bhi create hota hai.

👉 Ye Persistence Context ek **first-level cache** hota hai.
👉 Ye manage karta hai entity ke different states ko.
👉 Ye decide karta hai kab DB ko hit karna hai.

---

# 🟢 1. Transient (New) State

```java
User user = new User();
```

Abhi:

* Object create hua hai
* DB me nahi hai
* Persistence Context me nahi hai
* EntityManager ko pata bhi nahi hai is object ke baare me

👉 Is state ko bolte hain **Transient / New State**

---

# 🟢 2. Managed (Persistent) State

```java
em.persist(user);
```

⚠️ Important:

* Ye directly DB me nahi jaata
* Pehle Persistence Context me add hota hai
* Ab ye **Managed state** me hai

Ab:

* JPA is object ko track karega
* Agar tum isme change karoge → automatically detect karega
* Flush/commit par DB me sync karega

---

### 🔥 Flush kab hota hai?

* Transaction commit par
* Manually `em.flush()` call karne par
* Kabhi kabhi query execute hone par

Tab jaake DB update hota hai.

---

# 🟢 3. Managed via Fetch (Second Way to Enter)

Dusra tareeka entity Persistence Context me aane ka:

```java
User user = em.find(User.class, 1);
```

Flow:

1. Pehle Persistence Context check karega
2. Agar waha mila → DB hit nahi karega
3. Agar nahi mila → DB se fetch karega
4. Phir Persistence Context me store karega

👉 Isliye isko bolte hain **First Level Cache**

---

# 🟢 4. Removed State

```java
em.remove(user);
```

Ab:

* Entity Managed state se → Removed state me chali gayi
* Abhi DB se delete nahi hui hai
* Sirf Persistence Context me marked for deletion hai

Actual deletion tab hoga jab:

```java
em.flush(); 
// ya transaction commit
```

---

### 🔥 Interesting Point

Agar tum:

```java
em.remove(user);
```

ke baad:

```java
em.persist(user);
```

kar do (flush se pehle),
toh entity wapas **Managed state** me aa sakti hai.

---

# 🟢 5. Detached State

Detached ka matlab:

* Persistence Context ab entity ko manage nahi kar raha
* JPA changes track nahi karega

Detached hone ke tareeke:

### 1️⃣ Manually detach

```java
em.detach(user);
```

### 2️⃣ Close EntityManager

```java
em.close();
```

Ab:

* Entity independent ho gayi
* Changes DB me auto sync nahi honge

---

# 🟢 6. Merge (Detached → Managed)

Agar detached entity ko wapas JPA ke control me lana ho:

```java
user = em.merge(user);
```

Ab:

* Ye phir se Managed state me aa gayi
* Ab JPA changes track karega



---
