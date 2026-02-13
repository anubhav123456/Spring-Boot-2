
---

# 🔥 Spring Boot Exception Handling

Spring Boot me exception handling ka main goal hota hai:

* Clean API response dena
* Proper HTTP status return karna
* Centralized error handling karna
* Controller me try-catch avoid karna

Spring Boot me 3 important cheezein hoti hain:

1. `@ExceptionHandler`
2. `@RestControllerAdvice`
3. `@ResponseStatus`

---

# 1️⃣ Custom Exception – `ResourceNotFoundException`

```java
public class ResourceNotFoundException extends RuntimeException{
    public ResourceNotFoundException(String message) {
        super(message);
    }
}
```

### ✅ Kya ho raha hai?

* Ye ek **custom unchecked exception** hai
* `RuntimeException` extend kar raha hai
* Jab koi resource (User, Product, etc.) database me na mile → ye throw karte hain

### Example

```java
User user = userRepository.findById(id)
        .orElseThrow(() -> new ResourceNotFoundException("User not found with id: " + id));
```

Agar user nahi mila → exception throw hogi.

---

# 2️⃣ Error Response Object – `ApiError`

```java
import lombok.Builder;
import lombok.Data;
import org.springframework.http.HttpStatus;

import java.util.List;

@Data
@Builder
public class ApiError {

    private HttpStatus status;
    private String message;
    private List<String> subErrors;

}
```

### ✅ Purpose

Ye ek **standard error structure** define karta hai.

Response JSON kuch aisa banega:

```json
{
  "status": "BAD_REQUEST",
  "message": "Input validation failed",
  "subErrors": [
    "Name is required",
    "Email must be valid"
  ]
}
```

Professional APIs me hamesha structured error response hota hai.

---

# 3️⃣ Global Exception Handler – `@RestControllerAdvice`

```java
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

import java.util.List;
import java.util.stream.Collectors;

@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ApiResponse<?>> handleResourceNotFound(ResourceNotFoundException exception) {
        ApiError apiError = ApiError.builder()
                .status(HttpStatus.NOT_FOUND)
                .message(exception.getMessage())
                .build();
        return buildErrorResponseEntity(apiError);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ApiResponse<?>> handleInternalServerError(Exception exception) {
        ApiError apiError = ApiError.builder()
                .status(HttpStatus.INTERNAL_SERVER_ERROR)
                .message(exception.getMessage())
                .build();
        return buildErrorResponseEntity(apiError);
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ApiResponse<?>> handleInputValidationErrors(MethodArgumentNotValidException exception) {

        List<String> errors = exception
                .getBindingResult()
                .getAllErrors()
                .stream()
                .map(error -> error.getDefaultMessage())
                .collect(Collectors.toList());

        ApiError apiError = ApiError.builder()
                .status(HttpStatus.BAD_REQUEST)
                .message("Input validation failed")
                .subErrors(errors)
                .build();

        return buildErrorResponseEntity(apiError);
    }

    private ResponseEntity<ApiResponse<?>> buildErrorResponseEntity(ApiError apiError) {
        return new ResponseEntity<>(new ApiResponse<>(apiError), apiError.getStatus());
    }
}
```

---

# 🔥 Now Concept Samjho Properly

---

# 🧠 1️⃣ `@RestControllerAdvice`

### 🔹 Kya hai?

Ye ek **global exception handler** hai jo pure application ke controllers ko monitor karta hai.

Ye internally combination hai:

```
@ControllerAdvice + @ResponseBody
```

Matlab:

* Sare controllers ke exceptions yaha catch honge
* Automatically JSON response return hoga

### Why use it?

Agar 20 controllers hain, toh har controller me try-catch nahi likhna padega.

Centralized handling ✔
Clean code ✔
Production ready ✔

---

# 🧠 2️⃣ `@ExceptionHandler`

### 🔹 Kya karta hai?

Ye specific exception ko handle karta hai.

Example:

```java
@ExceptionHandler(ResourceNotFoundException.class)
```

Matlab:

> Jab bhi ResourceNotFoundException throw hogi → ye method execute hoga.

---

### Flow Diagram

```
Controller
   ↓
Service
   ↓
Exception thrown
   ↓
@RestControllerAdvice
   ↓
Matching @ExceptionHandler method
   ↓
ResponseEntity return
```

---

# 🧠 3️⃣ `@ResponseStatus`

Ye annotation tab use hota hai jab tum direct exception class pe status define karna chaho.

Example:

```java
@ResponseStatus(HttpStatus.NOT_FOUND)
public class ResourceNotFoundException extends RuntimeException {
}
```

Ab agar ye exception throw hui, Spring automatically 404 return karega.

---

### Difference:

| Approach                | When to use                    |
| ----------------------- | ------------------------------ |
| `@ResponseStatus`       | Simple exception               |
| `@ExceptionHandler`     | Custom response body banana ho |
| `@RestControllerAdvice` | Global handling                |

Tumhara approach best hai for real-world production apps 🔥

---

# 🧠 4️⃣ Validation Exception Handling

```java
@ExceptionHandler(MethodArgumentNotValidException.class)
```

Ye tab trigger hota hai jab:

```java
@PostMapping
public ResponseEntity<?> createUser(@Valid @RequestBody UserDto dto)
```

Aur validation fail ho jaye.

Tum:

* All validation errors collect kar rahe ho
* List<String> me convert kar rahe ho
* Structured response bhej rahe ho

Very clean implementation ✔

---

# 🔥 What Happens Internally

1. Spring MVC request receive karta hai
2. Controller execute hota hai
3. Agar exception aayi →
4. Spring scans for:

   * Local `@ExceptionHandler`
   * Then `@RestControllerAdvice`
5. Matching handler milta hai
6. ResponseEntity return hota hai

---

# 🔥 Interview Ready Answer

Agar interviewer pooche:

> How does Spring Boot handle exceptions?

Tum bolna:

* Spring Boot provides centralized exception handling using `@RestControllerAdvice`
* `@ExceptionHandler` is used to handle specific exceptions
* `@ResponseStatus` is used to associate HTTP status with exceptions
* We create custom exceptions
* We return structured error responses using ResponseEntity

---

# 🚀 Why Your Implementation Is Good

✔ Custom Exception
✔ Structured Error Model
✔ Validation Handling
✔ Generic Exception Fallback
✔ Clean Builder Pattern
✔ Reusable build method

Production level design 🔥

---
