# Spring Boot Interview Questions - Complete Guide

## Table of Contents
1. [Spring Boot Basics](#spring-boot-basics)
2. [Spring MVC Questions](#spring-mvc-questions)
3. [Spring Data JPA Questions](#spring-data-jpa-questions)
4. [REST API Questions](#rest-api-questions)
5. [AOP Questions](#aop-questions)
6. [Advanced Questions](#advanced-questions)
7. [Quick Revision Points](#quick-revision-points)
8. [Complete Notes Index](#complete-notes-index)

---

## Spring Boot Basics

### Q1: What is Spring Boot?
**Answer**: Spring Boot is an extension of the Spring Framework that simplifies application development by providing:
- **Auto-configuration**: Automatically configures beans based on classpath
- **Embedded servers**: Tomcat, Jetty built-in (no external deployment needed)
- **Starter dependencies**: Pre-configured dependency bundles
- **No XML configuration**: Convention over configuration
- **Production-ready features**: Health checks, metrics, externalized configuration

### Q2: What is @SpringBootApplication?
**Answer**: It's a convenience annotation combining three annotations:
```java
@SpringBootConfiguration  // Configuration class for bean definitions
@EnableAutoConfiguration  // Auto-configure beans based on classpath
@ComponentScan           // Scan for Spring components in current package
```

### Q3: What are Starter Dependencies?
**Answer**: Pre-configured dependency bundles that include all necessary dependencies:
- `spring-boot-starter-web`: Web apps (Tomcat, Spring MVC, Jackson)
- `spring-boot-starter-data-jpa`: JPA (Hibernate, connection pool)
- `spring-boot-starter-security`: Security (authentication, authorization)
- `spring-boot-starter-validation`: Bean validation
- `spring-boot-starter-thymeleaf`: Thymeleaf template engine

### Q4: What is auto-configuration?
**Answer**: Spring Boot automatically configures beans based on:
- Dependencies on classpath (e.g., H2 → in-memory DB configured)
- `@Conditional` annotations
- Properties in `application.properties`
- Default sensible configurations

### Q5: How to change the default port?
**Answer**: In `application.properties`:
```properties
server.port=9090
```

### Q6: What is the purpose of application.properties?
**Answer**: Externalized configuration file for:
- Server settings (port, context path)
- Database configuration
- Logging levels
- Custom application properties
- Profile-specific settings

---

## Spring MVC Questions

### Q7: What is DispatcherServlet?
**Answer**: The **Front Controller** in Spring MVC that:
- Receives all incoming HTTP requests
- Delegates to appropriate handler (controller)
- Coordinates with view resolver
- Manages the complete request-response cycle

### Q8: Difference between @Controller and @RestController?
**Answer**:

| @Controller | @RestController |
|-------------|-----------------|
| Returns **view name** | Returns **data** (JSON/XML) |
| For web pages (Thymeleaf, JSP) | For REST APIs |
| Needs `@ResponseBody` for JSON | `@ResponseBody` is implicit |

```java
// @Controller returns view
@Controller
public class BookController {
    @GetMapping("/book")
    public String getBook() {
        return "bookView"; // → Renders bookView.html
    }
}

// @RestController returns JSON
@RestController
public class BookRestController {
    @GetMapping("/api/book")
    public Book getBook() {
        return new Book(); // → Returns JSON
    }
}
```

### Q9: What is @RequestMapping?
**Answer**: Maps HTTP requests to handler methods:
```java
@RequestMapping(value = "/books", method = RequestMethod.GET)
```
Specialized annotations: `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`

### Q10: Difference between @RequestParam and @PathVariable?
**Answer**:
- **@RequestParam**: Query parameters (`/books?id=1`)
- **@PathVariable**: URL path segments (`/books/1`)

```java
@GetMapping("/books")
public Book getByQuery(@RequestParam Long id) { }  // /books?id=1

@GetMapping("/books/{id}")
public Book getByPath(@PathVariable Long id) { }   // /books/1
```

### Q11: What is @ModelAttribute?
**Answer**: Binds form data to a Java object:
```java
@PostMapping("/book")
public String save(@ModelAttribute Book book) {
    // Form fields automatically mapped to Book properties
}
```

### Q12: How does @Valid work with BindingResult?
**Answer**: 
- `@Valid` triggers validation annotations on the object
- `BindingResult` holds validation errors
- **Must be immediately after @Valid parameter**

```java
@PostMapping("/book")
public String save(@Valid @ModelAttribute Book book, 
                   BindingResult result) {
    if (result.hasErrors()) {
        return "bookForm"; // Return to form with errors
    }
    return "success";
}
```

---

## Spring Data JPA Questions

### Q13: What is JPA?
**Answer**: **Java Persistence API** - a specification for ORM in Java. It defines:
- Annotations for mapping (`@Entity`, `@Id`, `@Column`)
- EntityManager for persistence operations
- JPQL for querying

**Hibernate** is the most popular JPA implementation.

### Q14: What is JpaRepository?
**Answer**: Interface providing CRUD operations without implementation:
- `save()`, `findById()`, `findAll()`, `deleteById()`
- Derived query methods from method names
- Custom `@Query` support

```java
@Repository
public interface BookRepository extends JpaRepository<Book, Long> {
    // No implementation needed!
}
```

### Q15: Explain derived query methods.
**Answer**: Query generation from method names:
```java
// Method name → SQL query
List<Book> findByAuthor(String author);
// → SELECT * FROM book WHERE author = ?

List<Book> findByPriceGreaterThan(double price);
// → SELECT * FROM book WHERE price > ?

List<Book> findByTitleContaining(String keyword);
// → SELECT * FROM book WHERE title LIKE '%keyword%'
```

### Q16: What is @Query annotation?
**Answer**: For custom JPQL or native SQL:
```java
@Query("SELECT b FROM Book b WHERE b.price > :price")
List<Book> findExpensiveBooks(@Param("price") double price);

@Query(value = "SELECT * FROM books", nativeQuery = true)
List<Book> findAllNative();
```

### Q17: Difference between EAGER and LAZY loading?
**Answer**:
- **EAGER**: Related entities loaded immediately with parent
- **LAZY**: Related entities loaded on first access

```java
@OneToMany(fetch = FetchType.LAZY)  // Load books when accessed
private List<Book> books;

@ManyToOne(fetch = FetchType.EAGER) // Load author with book
private Author author;
```

### Q18: Why are @Repository and @Transactional needed?
**Answer**:
- **@Repository**: Marks as Spring Data repository, enables exception translation
- **@Transactional**: Ensures database operations run in a transaction with rollback on errors

---

## REST API Questions

### Q19: What is REST?
**Answer**: **Representational State Transfer** - architectural style:
- Uses HTTP methods (GET, POST, PUT, DELETE)
- Stateless communication
- JSON/XML data formats
- Resource-based URLs

### Q20: HTTP Methods and their purposes?

| Method | Purpose | Idempotent | Safe |
|--------|---------|------------|------|
| GET | Read | ✅ Yes | ✅ Yes |
| POST | Create | ❌ No | ❌ No |
| PUT | Update (full) | ✅ Yes | ❌ No |
| PATCH | Update (partial) | ✅ Yes | ❌ No |
| DELETE | Delete | ✅ Yes | ❌ No |

### Q21: What is ResponseEntity?
**Answer**: Full control over HTTP response:
```java
@GetMapping("/{id}")
public ResponseEntity<Book> getBook(@PathVariable Long id) {
    return bookRepository.findById(id)
            .map(ResponseEntity::ok)                    // 200 OK
            .orElse(ResponseEntity.notFound().build()); // 404 Not Found
}
```

### Q22: What are @RequestBody and @ResponseBody?
**Answer**:
- **@RequestBody**: Converts JSON request → Java object
- **@ResponseBody**: Converts Java object → JSON response (implicit in @RestController)

### Q23: What is CORS?
**Answer**: **Cross-Origin Resource Sharing** - controls which domains can access your API:
```java
@CrossOrigin("http://localhost:3000")
@RestController
public class BookController { }
```

### Q24: What is idempotency?
**Answer**: An operation is **idempotent** if calling it multiple times produces the same result as calling it once.
- **Idempotent**: GET, PUT, DELETE
- **Non-idempotent**: POST (creates new resource each time)

---

## AOP Questions

### Q25: What is AOP?
**Answer**: **Aspect-Oriented Programming** separates cross-cutting concerns:
- Logging, Security, Transactions, Caching
- Without modifying business code

### Q26: Key AOP concepts?
**Answer**:
- **Aspect**: Module with cross-cutting logic (`@Aspect`)
- **Advice**: Action at joinpoint (`@Before`, `@After`, `@Around`)
- **Pointcut**: Where to apply (`execution(* Controller.*(..))`)
- **JoinPoint**: Execution point (method call)

### Q27: Types of Advice?
**Answer**:
- `@Before`: Before method execution
- `@After`: After method (always, success or failure)
- `@AfterReturning`: After successful return
- `@AfterThrowing`: After exception
- `@Around`: Before and after (most powerful)

### Q28: Sample AOP code?
**Answer**:
```java
@Aspect
@Component
public class LoggingAspect {
    
    @Pointcut("execution(* com.example.controller.*.*(..))")
    public void controllerMethods() {}
    
    @Before("controllerMethods()")
    public void logBefore(JoinPoint jp) {
        System.out.println("Entering: " + jp.getSignature().getName());
    }
}
```

---

## Advanced Questions

### Q29: How do you handle exceptions globally?
**Answer**: Use `@RestControllerAdvice`:
```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ResourceNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handleNotFound(ResourceNotFoundException ex) {
        return new ErrorResponse(404, ex.getMessage());
    }
}
```

### Q30: Difference between Page and Slice?
**Answer**:
- **Page**: Includes total count (extra COUNT query)
- **Slice**: No total count (faster for infinite scroll)

---

## Quick Revision Points

### Spring Boot
- ✅ `@SpringBootApplication` = @Configuration + @EnableAutoConfiguration + @ComponentScan
- ✅ Embedded server (no external deployment)
- ✅ `application.properties` for configuration
- ✅ Starter dependencies bundle related dependencies

### Spring MVC
- ✅ `DispatcherServlet` is the Front Controller
- ✅ `@Controller` → views, `@RestController` → JSON
- ✅ `@RequestMapping` / `@GetMapping` / `@PostMapping`
- ✅ `@ModelAttribute` for form binding
- ✅ `@Valid` + `BindingResult` for validation

### Spring Data JPA
- ✅ `JpaRepository` provides CRUD
- ✅ Derived query methods from method names
- ✅ `@Query` for custom JPQL
- ✅ `@OneToMany`, `@ManyToOne` for relationships
- ✅ `LAZY` fetch for collections

### REST APIs
- ✅ Use proper HTTP methods (GET/POST/PUT/DELETE)
- ✅ Use appropriate status codes
- ✅ `@RestController` for REST endpoints
- ✅ `ResponseEntity` for full response control
- ✅ `@CrossOrigin` for CORS

### AOP
- ✅ `@Aspect` + `@Component` for aspects
- ✅ Pointcut expressions: `execution(* Controller.*(..))`
- ✅ Advice types: @Before, @After, @Around
- ✅ Separates cross-cutting concerns

---

## Complete Notes Index

| # | Topic | Link |
|---|-------|------|
| 01 | Spring Boot Introduction | [01_Spring_Boot_Introduction.md](01_Spring_Boot_Introduction.md) |
| 02 | Spring Boot Architecture | [02_Spring_Boot_Architecture.md](02_Spring_Boot_Architecture.md) |
| 03 | MVC Architecture Pattern | [03_MVC_Architecture_Pattern.md](03_MVC_Architecture_Pattern.md) |
| 04 | Controllers & Request Mapping | [04_Controllers_Request_Mapping.md](04_Controllers_Request_Mapping.md) |
| 05 | @ModelAttribute & Form Binding | [05_Model_Attribute_Form_Binding.md](05_Model_Attribute_Form_Binding.md) |
| 06 | Session Management | [06_Session_Management.md](06_Session_Management.md) |
| 07 | Thymeleaf Template Engine | [07_Thymeleaf_Template_Engine.md](07_Thymeleaf_Template_Engine.md) |
| 08 | Server-Side Validation | [08_Server_Side_Validation.md](08_Server_Side_Validation.md) |
| 09 | Spring Data JPA | [09_Spring_Data_JPA.md](09_Spring_Data_JPA.md) |
| 10 | AOP in Spring Boot | [10_AOP_Spring_Boot.md](10_AOP_Spring_Boot.md) |
| 11 | Web Services Introduction | [11_Web_Services_Introduction.md](11_Web_Services_Introduction.md) |
| 12 | REST API Fundamentals | [12_REST_API_Fundamentals.md](12_REST_API_Fundamentals.md) |
| 13 | Building REST APIs | [13_Building_REST_APIs_Spring_Boot.md](13_Building_REST_APIs_Spring_Boot.md) |
| 14 | CORS Configuration | [14_CORS_Configuration.md](14_CORS_Configuration.md) |
| 15 | JPA Relationships | [15_JPA_Relationships.md](15_JPA_Relationships.md) |
| 16 | JPQL & Custom Queries | [16_JPQL_Custom_Queries.md](16_JPQL_Custom_Queries.md) |
| 17 | Executable JAR | [17_Executable_JAR.md](17_Executable_JAR.md) |
| 18 | Exception Handling | [18_Exception_Handling.md](18_Exception_Handling.md) |
| 19 | Pagination & Sorting | [19_Pagination_Sorting.md](19_Pagination_Sorting.md) |
| 20 | Interview Questions | [20_Interview_Questions.md](20_Interview_Questions.md) |

---

**End of Note 20: Interview Questions & Complete Reference**
