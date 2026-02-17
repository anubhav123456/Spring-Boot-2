

---

# 🟢 1️⃣ Simple One-To-One (Basic Version)

### 🧠 Concept

Ek user ka **sirf ek address** hoga
Aur ek address sirf **ek user ka** hoga.

👉 This is One-To-One relationship.

---

## 🧾 Entity 1: `UserDetails`

```java
@Entity
@Table(name = "user_details")
public class UserDetails {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String phone;

    @OneToOne(cascade = CascadeType.ALL)
    private UserAddress userAddress;

    public UserDetails() {
    }

    // getters and setters
}
```

---

## 🧾 Entity 2: `UserAddress`

```java
@Entity
@Table(name = "user_address")
public class UserAddress {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String street;
    private String city;
    private String state;
    private String country;
    private String pinCode;

    public UserAddress() {
    }

    // getters and setters
}
```

---

### 🔍 What Happens in DB?

Hibernate will create:

```
user_details
-------------
id | name | phone | user_address_id

user_address
-------------
id | street | city | state | country | pin_code
```

⚠️ But since we did NOT define `@JoinColumn`, Hibernate auto generate karega foreign key column.

---

# 🟢 2️⃣ Proper One-To-One with `@JoinColumn` (Recommended Way)

Ab hum explicitly foreign key define karenge.

---

## 🧾 `UserDetails`

```java
@Entity
@Table(name = "user_details")
public class UserDetails {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String phone;

    @OneToOne(cascade = CascadeType.ALL)
    @JoinColumn(name = "address_id", referencedColumnName = "id")
    private UserAddress userAddress;

    public UserDetails() {
    }

    // getters and setters
}
```

---

## 🧾 `UserAddress`

```java
@Entity
@Table(name = "user_address")
public class UserAddress {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String street;
    private String city;
    private String state;
    private String country;
    private String pinCode;

    public UserAddress() {
    }

    // getters and setters
}
```

---

## 🧠 What `@JoinColumn` Does?

```java
@JoinColumn(name = "address_id", referencedColumnName = "id")
```

Means:

* `address_id` column banega in `user_details`
* It references `id` column of `user_address`

---

## 🗂 DB Structure

```
user_details
---------------------------------
id | name | phone | address_id

user_address
---------------------------------
id | street | city | state | country | pin_code
```

👉 `address_id` is Foreign Key

---

# 🟢 3️⃣ One-To-One with Composite Key (`@EmbeddedId`)

Ab thoda advanced concept 🔥

Yaha address ka primary key single id nahi hai.

Instead:

```
street + pinCode = primary key
```

---

## 🧾 Step 1: Create Composite Key Class

### `UserAddressCK`

```java
@Embeddable
public class UserAddressCK {

    private String street;
    private String pinCode;

    // getters and setters
}
```

---

## 🧾 Step 2: Use `@EmbeddedId` in Address

```java
@Entity
@Table(name = "user_address")
public class UserAddress {

    @EmbeddedId
    private UserAddressCK id;

    private String city;
    private String state;
    private String country;

    // getters and setters
}
```

---

## 🧾 Step 3: Map Composite Join in UserDetails

```java
@Entity
@Table(name = "user_details")
public class UserDetails {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String phone;

    @OneToOne(cascade = CascadeType.ALL)
    @JoinColumns({
        @JoinColumn(name = "address_street", referencedColumnName = "street"),
        @JoinColumn(name = "address_pin_code", referencedColumnName = "pinCode")
    })
    private UserAddress userAddress;

    public UserDetails() {
    }

    // getters and setters
}
```

---

## 🧠 What Happens Here?

Since Address primary key = (street + pinCode)

UserDetails table will contain:

```
address_street
address_pin_code
```

Both together act as Foreign Key.



---

# 🧠 Pro Tip (Interview Level)

If interviewer asks:

👉 Which side owns relationship?

Answer:

* The side with `@JoinColumn` is the **owning side**
* That side controls foreign key

---

