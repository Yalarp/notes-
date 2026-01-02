# Exception Handling in Spring Boot - Complete Guide

## Table of Contents
1. [Introduction](#introduction)
2. [Exception Handling Approaches](#exception-handling-approaches)
3. [@ExceptionHandler - Controller Level](#exceptionhandler---controller-level)
4. [@ControllerAdvice - Global Level](#controlleradvice---global-level)
5. [@RestControllerAdvice - REST APIs](#restcontrolleradvice---rest-apis)
6. [Custom Exception Classes](#custom-exception-classes)
7. [Error Response Structure](#error-response-structure)
8. [Complete Implementation](#complete-implementation)
9. [Best Practices](#best-practices)
10. [Summary](#summary)

---

## Introduction

Exception handling in Spring Boot ensures that errors are handled gracefully and clients receive meaningful error responses instead of stack traces.

---

## Exception Handling Approaches

| Approach | Scope | Use Case |
|----------|-------|----------|
| `try-catch` | Single method | Simple, localized handling |
| `@ExceptionHandler` | Single controller | Controller-specific errors |
| `@ControllerAdvice` | All controllers (views) | Global handling for web apps |
| `@RestControllerAdvice` | All REST controllers | Global handling for APIs |

---

## @ExceptionHandler - Controller Level

### Basic Usage

```java
@Controller
public class BookController {
    
    @GetMapping("/book/{id}")
    public String getBook(@PathVariable Long id, Model model) {
        Book book = bookService.findById(id)
                .orElseThrow(() -> new BookNotFoundException("Book not found: " + id));
        model.addAttribute("book", book);
        return "bookDetail";
    }
    
    // Handles BookNotFoundException in THIS controller only
    @ExceptionHandler(BookNotFoundException.class)
    public String handleBookNotFound(BookNotFoundException ex, Model model) {
        model.addAttribute("error", ex.getMessage());
        return "error";
    }
}
```

### For REST Controller

```java
@RestController
@RequestMapping("/api/books")
public class BookRestController {
    
    @GetMapping("/{id}")
    public Book getBook(@PathVariable Long id) {
        return bookService.findById(id)
                .orElseThrow(() -> new BookNotFoundException("Book not found: " + id));
    }
    
    @ExceptionHandler(BookNotFoundException.class)
    public ResponseEntity<String> handleNotFound(BookNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(ex.getMessage());
    }
}
```

---

## @ControllerAdvice - Global Level

### What is @ControllerAdvice?

`@ControllerAdvice` is a specialization of `@Component` that allows you to handle exceptions across all controllers globally.

### Usage for Web Applications (Views)

```java
@ControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(BookNotFoundException.class)
    public ModelAndView handleBookNotFound(BookNotFoundException ex) {
        ModelAndView mav = new ModelAndView("error");
        mav.addObject("message", ex.getMessage());
        mav.addObject("status", "404 - Not Found");
        return mav;
    }
    
    @ExceptionHandler(ValidationException.class)
    public ModelAndView handleValidation(ValidationException ex) {
        ModelAndView mav = new ModelAndView("error");
        mav.addObject("message", "Validation failed: " + ex.getMessage());
        mav.addObject("status", "400 - Bad Request");
        return mav;
    }
    
    @ExceptionHandler(Exception.class)
    public ModelAndView handleGeneral(Exception ex) {
        ModelAndView mav = new ModelAndView("error");
        mav.addObject("message", "An unexpected error occurred");
        mav.addObject("status", "500 - Internal Server Error");
        return mav;
    }
}
```

---

## @RestControllerAdvice - REST APIs

### What is @RestControllerAdvice?

`@RestControllerAdvice` = `@ControllerAdvice` + `@ResponseBody`

It handles exceptions for REST controllers and returns JSON responses.

### Complete Implementation

```java
@RestControllerAdvice
public class GlobalRestExceptionHandler {
    
    // Handle custom not found exception
    @ExceptionHandler(ResourceNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handleNotFound(ResourceNotFoundException ex) {
        return new ErrorResponse(
            HttpStatus.NOT_FOUND.value(),
            "Resource Not Found",
            ex.getMessage(),
            LocalDateTime.now()
        );
    }
    
    // Handle validation errors
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleValidation(MethodArgumentNotValidException ex) {
        List<String> errors = ex.getBindingResult()
                .getFieldErrors()
                .stream()
                .map(err -> err.getField() + ": " + err.getDefaultMessage())
                .collect(Collectors.toList());
        
        return new ErrorResponse(
            HttpStatus.BAD_REQUEST.value(),
            "Validation Failed",
            String.join(", ", errors),
            LocalDateTime.now()
        );
    }
    
    // Handle illegal arguments
    @ExceptionHandler(IllegalArgumentException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleIllegalArgument(IllegalArgumentException ex) {
        return new ErrorResponse(
            HttpStatus.BAD_REQUEST.value(),
            "Bad Request",
            ex.getMessage(),
            LocalDateTime.now()
        );
    }
    
    // Handle all other exceptions
    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ErrorResponse handleGeneral(Exception ex) {
        return new ErrorResponse(
            HttpStatus.INTERNAL_SERVER_ERROR.value(),
            "Internal Server Error",
            "An unexpected error occurred",
            LocalDateTime.now()
        );
    }
}
```

---

## Custom Exception Classes

### Creating Custom Exceptions

```java
// Generic resource not found
public class ResourceNotFoundException extends RuntimeException {
    
    public ResourceNotFoundException(String message) {
        super(message);
    }
    
    public ResourceNotFoundException(String resource, Long id) {
        super(resource + " not found with id: " + id);
    }
}

// Specific exceptions
public class BookNotFoundException extends ResourceNotFoundException {
    public BookNotFoundException(Long id) {
        super("Book", id);
    }
}

public class UserNotFoundException extends ResourceNotFoundException {
    public UserNotFoundException(Long id) {
        super("User", id);
    }
}

// Business logic exception
public class InsufficientStockException extends RuntimeException {
    public InsufficientStockException(String product, int requested, int available) {
        super(String.format("Insufficient stock for %s. Requested: %d, Available: %d",
                product, requested, available));
    }
}
```

### Using in Controller

```java
@GetMapping("/{id}")
public Book getBook(@PathVariable Long id) {
    return bookRepository.findById(id)
            .orElseThrow(() -> new BookNotFoundException(id));
}
```

---

## Error Response Structure

### Error Response Class

```java
public class ErrorResponse {
    private int status;
    private String error;
    private String message;
    private LocalDateTime timestamp;
    private String path;  // Optional: request path
    private List<FieldError> fieldErrors;  // Optional: validation errors
    
    // Constructors
    public ErrorResponse(int status, String error, String message, LocalDateTime timestamp) {
        this.status = status;
        this.error = error;
        this.message = message;
        this.timestamp = timestamp;
    }
    
    // Getters and setters
}

public class FieldError {
    private String field;
    private String message;
    
    // Constructor, getters
}
```

### Example JSON Response

```json
{
    "status": 404,
    "error": "Resource Not Found",
    "message": "Book not found with id: 123",
    "timestamp": "2024-01-02T10:30:45.123Z",
    "path": "/api/books/123"
}
```

### Validation Error Response

```json
{
    "status": 400,
    "error": "Validation Failed",
    "message": "title: must not be blank, price: must be positive",
    "timestamp": "2024-01-02T10:30:45.123Z",
    "fieldErrors": [
        {"field": "title", "message": "must not be blank"},
        {"field": "price", "message": "must be positive"}
    ]
}
```

---

## Complete Implementation

### Full Exception Handler

```java
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {
    
    // 404 - Resource Not Found
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(
            ResourceNotFoundException ex, 
            WebRequest request) {
        
        log.error("Resource not found: {}", ex.getMessage());
        
        ErrorResponse error = ErrorResponse.builder()
                .status(HttpStatus.NOT_FOUND.value())
                .error("Not Found")
                .message(ex.getMessage())
                .timestamp(LocalDateTime.now())
                .path(((ServletWebRequest) request).getRequest().getRequestURI())
                .build();
        
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }
    
    // 400 - Validation Failed
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidation(
            MethodArgumentNotValidException ex) {
        
        List<FieldError> fieldErrors = ex.getBindingResult()
                .getFieldErrors()
                .stream()
                .map(err -> new FieldError(err.getField(), err.getDefaultMessage()))
                .collect(Collectors.toList());
        
        ErrorResponse error = ErrorResponse.builder()
                .status(HttpStatus.BAD_REQUEST.value())
                .error("Validation Failed")
                .message("Input validation failed")
                .timestamp(LocalDateTime.now())
                .fieldErrors(fieldErrors)
                .build();
        
        return ResponseEntity.badRequest().body(error);
    }
    
    // 400 - Bad Request
    @ExceptionHandler({IllegalArgumentException.class, ConstraintViolationException.class})
    public ResponseEntity<ErrorResponse> handleBadRequest(RuntimeException ex) {
        
        ErrorResponse error = ErrorResponse.builder()
                .status(HttpStatus.BAD_REQUEST.value())
                .error("Bad Request")
                .message(ex.getMessage())
                .timestamp(LocalDateTime.now())
                .build();
        
        return ResponseEntity.badRequest().body(error);
    }
    
    // 500 - Internal Server Error
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneral(Exception ex) {
        
        log.error("Unexpected error", ex);
        
        ErrorResponse error = ErrorResponse.builder()
                .status(HttpStatus.INTERNAL_SERVER_ERROR.value())
                .error("Internal Server Error")
                .message("An unexpected error occurred. Please try again later.")
                .timestamp(LocalDateTime.now())
                .build();
        
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(error);
    }
}
```

---

## Best Practices

### 1. Use Custom Exceptions

```java
// Good: Specific exception
throw new BookNotFoundException(id);

// Bad: Generic exception
throw new RuntimeException("Book not found");
```

### 2. Don't Expose Internal Details

```java
// Good: User-friendly message
return new ErrorResponse("An error occurred");

// Bad: Stack trace exposed
return new ErrorResponse(ex.getStackTrace().toString());
```

### 3. Log All Exceptions

```java
@ExceptionHandler(Exception.class)
public ErrorResponse handleAll(Exception ex) {
    log.error("Exception occurred", ex);  // Log full stack trace
    return new ErrorResponse("An error occurred");  // Return safe message
}
```

### 4. Use Appropriate Status Codes

| Situation | Status Code |
|-----------|-------------|
| Resource not found | 404 |
| Validation error | 400 |
| Authentication required | 401 |
| Access denied | 403 |
| Server error | 500 |

---

## Summary

### Key Points

1. **@ExceptionHandler** - Controller-specific handling
2. **@ControllerAdvice** - Global handling (views)
3. **@RestControllerAdvice** - Global handling (REST APIs)
4. Create **custom exceptions** for clarity
5. Return **structured error responses**
6. Always **log exceptions** server-side

### Quick Reference

| Annotation | Returns | Scope |
|------------|---------|-------|
| `@ExceptionHandler` | View or JSON | Single controller |
| `@ControllerAdvice` | View (ModelAndView) | Global |
| `@RestControllerAdvice` | JSON | Global REST |

---

## Practice Questions

1. What is the difference between @ControllerAdvice and @RestControllerAdvice?
2. How do you handle validation errors globally?
3. What is the purpose of @ExceptionHandler?
4. How do you create a custom exception class?
5. What should an error response contain?
6. Why should you log exceptions even when handling them?

---

**End of Note 18: Exception Handling**

*Previous: [17_Executable_JAR.md](file:///c:/Users/2706p/Desktop/mcq/notes/17_Executable_JAR.md)*  
*Next: [19_Pagination_Sorting.md](file:///c:/Users/2706p/Desktop/mcq/notes/19_Pagination_Sorting.md)*
