

---

# 🔥 What is One-To-One Bidirectional Mapping?

Image me likha hai:

* **Both entities hold reference to each other**

  * `UserDetails` → reference to `UserAddress`
  * `UserAddress` → reference back to `UserDetails` (sirf object level pe, DB me alag FK nahi banega)

Matlab:

👉 Java objects ek dusre ko jaante hain
👉 But **database me sirf ek hi table me foreign key hoti hai**

---

# 🏗 Example Scenario

1 User ka 1 Address
1 Address sirf 1 User ka

```
UserDetails  <------->  UserAddress
```

---

# 🧠 Owner Side vs Inverse Side

Image me do parts dikhaye gaye hain:

## ✅ 1️⃣ Owner Side

* Foreign key isi table me banti hai
* `@JoinColumn` yahi likhte hain
* DB relationship control karta hai

👉 Is example me: **UserDetails is Owner**

---

## ❌ 2️⃣ Inverse Side (Non-owner)

* Yaha foreign key nahi banti
* Sirf object reference hota hai
* `mappedBy` use karte hain

👉 Is example me: **UserAddress is Inverse Side**

---

# 💾 Database Structure

Sirf `user_details` table me foreign key hogi:

```
user_details
------------
id
name
phone
address_id   (FK)

user_address
------------
id
street
city
state
country
pin_code
```

👉 Notice: `user_address` table me koi `user_id` nahi hai.

---

# 💻 Code Implementation

## 🟢 UserDetails (Owner Side)

```java
import jakarta.persistence.*;

@Entity
@Table(name = "user_details")
public class UserDetails {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String phone;

    // OWNER SIDE
    @OneToOne(cascade = CascadeType.ALL)
    @JoinColumn(name = "address_id", referencedColumnName = "id")
    private UserAddress userAddress;

    public UserDetails() {}

    // getters and setters
}
```

### 🔥 Important:

* `@JoinColumn` → Foreign key create karega
* `address_id` → user_details table me banega

---

## 🔵 UserAddress (Inverse Side)

```java
import jakarta.persistence.*;

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

    // INVERSE SIDE
    @OneToOne(mappedBy = "userAddress", fetch = FetchType.EAGER)
    private UserDetails userDetails;

    public UserAddress() {}

    // getters and setters
}
```

### 🔥 Important:

* `mappedBy = "userAddress"`

  * Yeh batata hai ki relationship ka owner `UserDetails` hai
  * `"userAddress"` = owner class ka variable name

---

# ⚙️ How It Works Internally

When you do:

```java
UserDetails user = new UserDetails();
UserAddress address = new UserAddress();

user.setUserAddress(address);
address.setUserDetails(user);

userRepository.save(user);
```

### Hibernate karega:

1. Pehle address save karega
2. Phir user save karega with `address_id` FK

---

# 🎯 Why Bidirectional?

Because now you can access:

```java
user.getUserAddress()
address.getUserDetails()
```

👉 Dono side se navigation possible hai.

---

# 🧨 Common Mistakes

❌ Don’t put `@JoinColumn` on both sides
❌ Don’t forget `mappedBy`
❌ Don’t mismatch variable name in `mappedBy`

---

# 🆚 Unidirectional vs Bidirectional

| Feature     | Unidirectional | Bidirectional   |
| ----------- | -------------- | --------------- |
| Reference   | One side       | Both sides      |
| Foreign Key | One table      | One table       |
| mappedBy    | No             | Yes             |
| Navigation  | One direction  | Both directions |

---

# 🧠 Real Life Analogy

Think like Husband ↔ Wife relationship:

* Husband has reference to Wife
* Wife also has reference to Husband
* But marriage certificate ek hi jagah register hota hai 😄

---

# 🚀 Summary

✔ One-to-one relationship
✔ Both entities reference each other
✔ Only ONE foreign key in DB
✔ Owner side → has `@JoinColumn`
✔ Inverse side → has `mappedBy`

---
