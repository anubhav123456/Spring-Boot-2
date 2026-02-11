
---

# 🔥 What is `ResponseEntity` in Spring Boot?

`ResponseEntity` = **Full HTTP Response control**

Tum decide kar sakte ho:

✔ Status Code
✔ Response Body
✔ Headers

```java
return new ResponseEntity<>(body, HttpStatus.OK);
```

or shortcut:

```java
return ResponseEntity.ok(body);
```

---

# 🟢 2XX — SUCCESS RESPONSES

Request sahi thi + server ne process kar li.

---

## ✅ **200 OK**

**Meaning:** Request successful, response body return ho rahi hai.

**Mostly Used:** GET, PUT, POST (idempotent)

```java
@GetMapping("/users/{id}")
public ResponseEntity<User> getUser(@PathVariable Long id){
    User user = service.findById(id);
    return ResponseEntity.ok(user);
}
```

📌 Use when: Data mil gaya successfully.

---

## 🆕 **201 Created**

**Meaning:** New resource create hua.

**Mostly Used:** POST

```java
@PostMapping("/users")
public ResponseEntity<User> createUser(@RequestBody User user){
    User saved = service.save(user);
    return ResponseEntity.status(HttpStatus.CREATED).body(saved);
}
```

📌 Example: Naya user account create hua.

---

## ⏳ **202 Accepted**

**Meaning:** Request accept ho gayi, but processing baad me hogi.

📌 Use case: Background jobs, Export, Import

```java
@PostMapping("/export")
public ResponseEntity<String> exportData(){
    asyncService.startExport();
    return ResponseEntity.accepted().body("Export started");
}
```

---

## 🚫 **204 No Content**

**Meaning:** Success, but body return nahi hogi.

**Mostly Used:** DELETE, Update with no return data

```java
@DeleteMapping("/users/{id}")
public ResponseEntity<Void> deleteUser(@PathVariable Long id){
    service.delete(id);
    return ResponseEntity.noContent().build();
}
```

---

## 📦 **206 Partial Content**

**Meaning:** Partial success (bulk operations)

Example: 100 users add kiye, 95 pass, 5 fail

```java
return ResponseEntity.status(HttpStatus.PARTIAL_CONTENT)
        .body("95 users created, 5 failed");
```

---

# 🔁 3XX — REDIRECTION

---

## 🔀 **301 Moved Permanently**

Old API → New API migrate

```java
return ResponseEntity.status(HttpStatus.MOVED_PERMANENTLY)
        .header("Location", "/v2/users")
        .build();
```

---

## 🔁 **308 Permanent Redirect**

Same as 301, but HTTP method change allowed nahi.

POST → POST hi rahega.

---

## 🗂 **304 Not Modified**

Caching ke liye.

Client bolta hai: "Last time jo data mila tha, kya update hua?"

Agar nahi hua:

```java
return ResponseEntity.status(HttpStatus.NOT_MODIFIED).build();
```

⚠️ PATCH ke liye use mat karo.

---

# 🔴 4XX — CLIENT ERRORS

Client ne galti ki.

---

## ❌ **400 Bad Request**

Missing / invalid data

```java
if(user.getEmail() == null){
   return ResponseEntity.badRequest().body("Email required");
}
```

---

## 🔐 **401 Unauthorized**

Auth token nahi diya.

---

## 🚫 **403 Forbidden**

Login hai, par permission nahi.

Admin only API example.

---

## 🔍 **404 Not Found**

```java
User user = repo.findById(id).orElseThrow(
    () -> new ResponseStatusException(HttpStatus.NOT_FOUND, "User not found")
);
```

---

## ⛔ **405 Method Not Allowed**

GET API pe POST maar diya.

Spring automatically handle karta hai.

---

## 📉 **422 Unprocessable Entity**

Business rule fail.

Example: France users allowed nahi

```java
if(user.getCountry().equals("France")){
   return ResponseEntity.unprocessableEntity()
           .body("Country not supported");
}
```

---

## 🚦 **429 Too Many Requests**

Rate limit exceed.

```java
return ResponseEntity.status(429).body("Rate limit exceeded");
```

---

# 💥 5XX — SERVER ERRORS

Server side problem.

---

## 💣 **500 Internal Server Error**

Generic error.

```java
@ExceptionHandler(Exception.class)
public ResponseEntity<String> handle(Exception ex){
    return ResponseEntity.status(500).body("Something went wrong");
}
```

---

## 🧱 **501 Not Implemented**

Feature ready nahi.

---

## 🌉 **502 Bad Gateway**

Proxy (Nginx) upstream server se baat nahi kar pa raha.

---

# ℹ️ 1XX — INFORMATIONAL

---

## ⏩ **100 Continue**

Client puchta hai: "Body bheju kya?"

Server: "Haan bhej"

Mostly large file uploads.

---

# 🧠 Interview Gold Summary

| Situation          | Status Code |
| ------------------ | ----------- |
| Data fetched       | 200         |
| Resource created   | 201         |
| Async process      | 202         |
| Delete success     | 204         |
| Old API → new      | 301/308     |
| Cache unchanged    | 304         |
| Invalid input      | 400         |
| Not logged in      | 401         |
| No permission      | 403         |
| Resource missing   | 404         |
| Business rule fail | 422         |
| Rate limit         | 429         |
| Server crash       | 500         |

---
