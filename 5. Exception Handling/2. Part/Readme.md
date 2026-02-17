
---

# ✅ Final Project Structure

```
exception/
   ├── ResourceNotFoundException.java
   ├── ApiError.java
   ├── ApiResponse.java
   └── GlobalExceptionHandler.java
```

---

# 1️⃣ Custom Exception

`ResourceNotFoundException.java`

```java
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.ResponseStatus;

@ResponseStatus(HttpStatus.NOT_FOUND)
public class ResourceNotFoundException extends RuntimeException {

    public ResourceNotFoundException(String message) {
        super(message);
    }
}
```

### ✔ Why `@ResponseStatus` here?

Even if global handler na bhi ho, Spring automatically 404 return karega.

---

# 2️⃣ ApiError (Error Structure)

`ApiError.java`

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

---

# 3️⃣ ApiResponse (Wrapper for ALL responses)

👉 Ye tumne jo diya hai, usko yaha place karna hai:

`ApiResponse.java`

```java
import com.fasterxml.jackson.annotation.JsonFormat;
import lombok.Data;

import java.time.LocalDateTime;

@Data
public class ApiResponse<T> {

    @JsonFormat(pattern = "hh:mm:ss dd-MM-yyyy")
    private LocalDateTime timeStamp;

    private T data;
    private ApiError error;

    public ApiResponse() {
        this.timeStamp = LocalDateTime.now();
    }

    public ApiResponse(T data) {
        this();
        this.data = data;
    }

    public ApiResponse(ApiError error) {
        this();
        this.error = error;
    }
}
```

---

# 🔥 Why ApiResponse is Important?

Ab tumhara har API response structure same hoga:

### ✅ Success Response

```json
{
  "timeStamp": "10:45:12 13-02-2026",
  "data": {
    "id": 1,
    "name": "Anubhav"
  },
  "error": null
}
```

### ❌ Error Response

```json
{
  "timeStamp": "10:46:02 13-02-2026",
  "data": null,
  "error": {
    "status": "NOT_FOUND",
    "message": "User not found with id: 5",
    "subErrors": null
  }
}
```

🔥 This is enterprise-grade API structure.

---

# 4️⃣ Global Exception Handler (Final Version)

`GlobalExceptionHandler.java`

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

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ApiResponse<?>> handleValidationErrors(MethodArgumentNotValidException exception) {

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

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ApiResponse<?>> handleGenericException(Exception exception) {

        ApiError apiError = ApiError.builder()
                .status(HttpStatus.INTERNAL_SERVER_ERROR)
                .message(exception.getMessage())
                .build();

        return buildErrorResponseEntity(apiError);
    }

    private ResponseEntity<ApiResponse<?>> buildErrorResponseEntity(ApiError apiError) {
        return new ResponseEntity<>(new ApiResponse<>(apiError), apiError.getStatus());
    }
}
```

---

# 🔥 How Everything Works Together

```
Controller
   ↓
Service
   ↓
Exception thrown
   ↓
@RestControllerAdvice
   ↓
Matching @ExceptionHandler
   ↓
ApiError created
   ↓
ApiResponse wrapper created
   ↓
ResponseEntity returned
```

---

# 🚀 Example Controller Using This Structure

```java
@GetMapping("/{id}")
public ResponseEntity<ApiResponse<User>> getUser(@PathVariable Long id) {

    User user = userService.getUserById(id);

    return ResponseEntity.ok(new ApiResponse<>(user));
}
```

---

# 🎯 Why This Is Interview-Level Strong?

✔ Centralized exception handling
✔ Custom exceptions
✔ Structured error response
✔ Validation handling
✔ Generic fallback
✔ Response wrapper consistency
✔ Timestamp included
✔ Production ready

---

# 💎 Final Verdict

Bhai 🔥 ab ye implementation:

* Startup level ✔
* Enterprise level ✔
* Microservices friendly ✔
* Interview safe ✔

Agar tum ye confidently explain kar do, interviewer bolega:

> “This guy has worked on real backend systems.”

---
