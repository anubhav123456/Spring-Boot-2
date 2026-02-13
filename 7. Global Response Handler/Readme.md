Bhai ye code **global response wrapper** banane ke liye use ho raha hai in Spring Framework (specially in Spring Boot).

Simple words me 👉 **Har controller ka response automatically ek common format me wrap ho jayega.**

---

## 🔥 1️⃣ `@RestControllerAdvice`

```java
@RestControllerAdvice
public class GlobalResponseHandler
```

Ye annotation ka matlab:

* Ye class **globally** saare controllers ko monitor karegi
* Ye responses ko modify kar sakti hai before sending to client
* Ye combination hai:

  * `@ControllerAdvice`
  * `@ResponseBody`

Matlab: Har API ka response yaha se pass hoga.

---

## 🔥 2️⃣ `implements ResponseBodyAdvice<Object>`

```java
public class GlobalResponseHandler implements ResponseBodyAdvice<Object>
```

Ye interface allow karta hai:

👉 Controller ka response client ko bhejne se pehle modify karne ke liye.

---

## 🔥 3️⃣ `supports()` method

```java
@Override
public boolean supports(MethodParameter returnType,
        Class<? extends HttpMessageConverter<?>> converterType) {
    return true;
}
```

Ye method decide karta hai:

> Kya ye advice apply karni chahiye ya nahi?

Yaha `return true` ka matlab:

✅ Har controller response pe apply karo
(Chahe String ho, Object ho, List ho — sab pe)

---

## 🔥 4️⃣ `beforeBodyWrite()` method (Main Logic 🔥)

```java
@Override
public Object beforeBodyWrite(Object body, ...)
```

Ye method tab call hoti hai jab:

👉 Controller ne response return kar diya ho
👉 Lekin abhi client ko bheja nahi gaya

---

### ⚡ Logic samjho:

```java
if(body instanceof ApiResponse<?>) {
    return body;
}
```

Agar response already `ApiResponse` type ka hai
👉 Toh dubara wrap mat karo
👉 Direct return kar do

---

```java
return new ApiResponse<>(body);
```

Agar normal object return hua hai like:

```java
return "User created successfully";
```

Toh automatically convert ho jayega:

```json
{
   "data": "User created successfully"
}
```

---

## 🎯 Real Example

### Without GlobalResponseHandler

Controller:

```java
@GetMapping("/hello")
public String hello() {
    return "Hello Bhai";
}
```

Response:

```
Hello Bhai
```

---

### With GlobalResponseHandler

Response ban jayega:

```json
{
   "data": "Hello Bhai"
}
```

---

## 🎯 Why This is Useful?

Production APIs me common response format chahiye hota hai like:

```json
{
   "success": true,
   "message": "User created",
   "data": {...}
}
```

Toh tum har controller me manually wrap karne ki jagah
👉 Ek hi jagah se globally handle kar sakte ho.

---

## 🚀 Complete Code

```java
import org.springframework.core.MethodParameter;
import org.springframework.http.MediaType;
import org.springframework.http.converter.HttpMessageConverter;
import org.springframework.http.server.ServerHttpRequest;
import org.springframework.http.server.ServerHttpResponse;
import org.springframework.web.bind.annotation.RestControllerAdvice;
import org.springframework.web.servlet.mvc.method.annotation.ResponseBodyAdvice;

@RestControllerAdvice
public class GlobalResponseHandler implements ResponseBodyAdvice<Object> {

    @Override
    public boolean supports(MethodParameter returnType, Class<? extends HttpMessageConverter<?>> converterType) {
        return true;
    }

    @Override
    public Object beforeBodyWrite(Object body, MethodParameter returnType, MediaType selectedContentType, Class<? extends HttpMessageConverter<?>> selectedConverterType, ServerHttpRequest request, ServerHttpResponse response) {
        if(body instanceof ApiResponse<?>) {
            return body;
        }

        return new ApiResponse<>(body);
    }
}
```
---

## 🚀 Interview Point

Agar interviewer puche:

> How do you implement global response wrapping in Spring Boot?

Tum bol sakte ho:

* Implement `ResponseBodyAdvice`
* Override `supports()` and `beforeBodyWrite()`
* Use `@RestControllerAdvice` for global interception

---

