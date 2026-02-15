

---

# 📌 JPA Architecture

![JPA Architecture](Architecture.jpg)


---

# 🏗 1️⃣ Persistence Unit (PU)

👉 Ye configuration block hota hai (persistence.xml ya Spring Boot config me)

Isme define hota hai:

* Database connection
* Hibernate provider
* Entities ka package
* Dialect
* Other properties

💡 Simple words me:

> Persistence Unit = "Database ka setting file"

Diagram me:

```
Persistence Unit1 → EntityManagerFactory
(1:1 relation)
```

Ek Persistence Unit ek EntityManagerFactory banata hai.

---

# 🏭 2️⃣ EntityManagerFactory (EMF)

Ye ek **factory** hai jo multiple EntityManager create karti hai.

Diagram me:

```
EntityManagerFactory → EntityManager 1
                     → EntityManager 2
                     → EntityManager N
(1 : Many)
```

💡 Important:

* EMF heavy object hota hai
* Usually application me ek hi hota hai
* Thread-safe hota hai

---

# 👨‍💼 3️⃣ EntityManager (EM)

Ye sabse important banda hai 😎

Ye karta kya hai?

* Persist (save)
* Find
* Remove
* Update
* Query

Har request / transaction ke liye usually ek EntityManager use hota hai.

Diagram me:

```
EntityManager → Persistence Context
(1 : 1)
```

---

# 🧠 4️⃣ Persistence Context (First Level Cache)

Ye ek memory area hai jahan:

* Entities temporarily stored rehti hain
* Changes track hote hain
* Dirty checking hoti hai

💡 Matlab:

> Jab tum entity ko load karte ho, wo pehle yahan aati hai, DB me directly nahi jaati.

Isliye agar same entity dubara fetch karo to DB hit nahi hota.

---

# 🧾 5️⃣ Entities (1 : Many)

Diagram me:

```
Persistence Context → Entity1
                    → Entity2
                    → EntityN
```

Entities kya hoti hain?

* Java classes
* @Entity annotation
* Table se mapped hoti hain

Example:

```java
@Entity
public class User {
   @Id
   private Long id;
   private String name;
}
```

---

# 🗣 6️⃣ JPQL (JPA Query Language)

Diagram me:

```
Entities → JQL → Dialect
```

JPQL SQL nahi hoti.
Ye entity-based hoti hai.

Example:

```java
SELECT u FROM User u WHERE u.name = 'Anubhav'
```

Note:

* Table name nahi
* Entity name use hota hai

---

# 🌍 7️⃣ Dialect

Dialect ka kaam:

👉 JPQL ko specific database SQL me convert karna

Example:

* MySQL dialect
* PostgreSQL dialect
* Oracle dialect

Har database ka SQL thoda different hota hai.

---

# 🔌 8️⃣ JDBC Driver

Dialect se generate hua SQL:

👉 JDBC ke through database ko bheja jata hai.

Diagram:

```
Dialect → SQL → JDBC → DB
```

---

# 🛢 9️⃣ Database

Finally data yahan store hota hai.

---

# 🎬 Ab Real-World Example Se Samjho (Food Delivery App 🍔)

Socho tum Swiggy jaisa app bana rahe ho.

---

## 🏢 Persistence Unit = Restaurant ka Kitchen Setup

* Gas kaha hai
* Fridge kaha hai
* Kis type ka stove hai

Ye sab configuration hai.

---

## 🏭 EntityManagerFactory = Head Chef 👨‍🍳

* Ek hi hota hai
* Sab junior chefs ko assign karta hai

---

## 👨‍🍳 EntityManager = Junior Chef

Har customer order ke liye ek junior chef:

* Order leta hai
* Banata hai
* Serve karta hai

---

## 🧠 Persistence Context = Chef ka Working Table

* Ingredients yahan rakhe hain
* Jo bana rahe ho wahi temporarily stored hai
* Final serve hone se pehle sab yahin hai

---

## 🧾 Entity = Dish (Burger, Pizza, etc.)

Each entity = ek dish

---

## 🗣 JPQL = Customer ka Order

Customer bolta hai:

> "Mujhe Chicken Burger do"

Wo table ka naam nahi bolta.
Wo dish ka naam bolta hai.

---

## 🌍 Dialect = Language Translator

Agar chef Italian hai aur customer Hindi me bol raha hai:

Translator convert karega.

Same way:
JPQL → Database specific SQL

---

## 🔌 JDBC = Waiter

Waiter order lekar kitchen se customer tak jata hai.

---

## 🛢 Database = Restaurant Store Room

Final storage yahi hai.

---

# 🔥 Complete Flow Example (Spring Boot)

```java
@Autowired
EntityManager em;

@Transactional
public void saveUser() {
   User user = new User();
   user.setName("Anubhav");
   em.persist(user);
}
```

Internally kya hota hai:

1. EntityManagerFactory se EM milta hai
2. EM persistence context create karta hai
3. user object PC me jata hai
4. Transaction commit pe:
5. JPQL/SQL generate hoti hai
6. Dialect convert karta hai
7. JDBC execute karta hai
8. DB me insert hota hai

---

# 🎯 Important Interview Points

✔ EntityManagerFactory = Thread-safe
✔ EntityManager = Not thread-safe
✔ Persistence Context = First Level Cache
✔ Flush vs Commit difference
✔ Dirty Checking automatically hota hai

---

# 🧠 Ek Line Me Summary

> JPA ek abstraction layer hai jo Java Objects ko database tables me convert karta hai through EntityManager, Persistence Context, JPQL, Dialect aur JDBC.

---

