# Web API in ASP.NET Core

## Table of Contents
1. [Introduction](#introduction)
2. [What is Web API?](#what-is-web-api)
3. [REST Principles](#rest-principles)
4. [Creating Web API](#creating-web-api)
5. [HTTP Methods](#http-methods)
6. [API Controller](#api-controller)
7. [Routing in API](#routing-in-api)
8. [Return Types](#return-types)
9. [Model Binding](#model-binding)
10. [Swagger Documentation](#swagger-documentation)
11. [Code Examples](#code-examples)
12. [Best Practices](#best-practices)
13. [Interview Questions](#interview-questions)

---

## Introduction

### Real-World Analogy
Web API is like a **restaurant waiter**:

```
Client (Customer)     → Makes requests (orders food)
Web API (Waiter)      → Receives and processes requests
Database (Kitchen)    → Prepares the data (food)
Response (Meal)       → Returns data in standard format (JSON)

Menu                  → API Documentation (Swagger)
Order format          → HTTP Methods (GET, POST, PUT, DELETE)
Table number          → URL endpoints
```

---

## What is Web API?

Web API is a framework for building HTTP services that can be consumed by any client including browsers, mobile apps, and other servers.

### API vs MVC

```
┌──────────────────────────────────────────────────────────────────┐
│                    MVC vs WEB API                                 │
├────────────────────────────┬─────────────────────────────────────┤
│         MVC                │           WEB API                   │
├────────────────────────────┼─────────────────────────────────────┤
│ Returns Views (HTML)       │ Returns Data (JSON/XML)             │
│ Controller : Controller    │ Controller : ControllerBase         │
│ Browser clients            │ Any client (mobile, web, IoT)       │
│ Uses Razor Views           │ No views, data only                 │
│ Form submissions           │ HTTP verbs with JSON body           │
│ Session/Cookie state       │ Stateless (typically)               │
└────────────────────────────┴─────────────────────────────────────┘
```

### Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    WEB API ARCHITECTURE                          │
└─────────────────────────────────────────────────────────────────┘

         ┌─────────────┐
         │   Client    │
         │ (React, Vue,│
         │  Mobile App)│
         └──────┬──────┘
                │ HTTP Request
                │ (GET, POST, PUT, DELETE)
                ▼
         ┌─────────────┐
         │ API Gateway │ (optional)
         └──────┬──────┘
                │
                ▼
┌───────────────────────────────────────────────────────────────┐
│                    ASP.NET CORE WEB API                        │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │                   Middleware Pipeline                    │  │
│  │  UseHttpsRedirection → UseAuthentication → UseAuthorize │  │
│  └────────────────────────────┬────────────────────────────┘  │
│                               │                                │
│  ┌────────────────────────────▼────────────────────────────┐  │
│  │                    API Controllers                       │  │
│  │   BookController, UserController, OrderController        │  │
│  └────────────────────────────┬────────────────────────────┘  │
│                               │                                │
│  ┌────────────────────────────▼────────────────────────────┐  │
│  │                    Services/Repositories                 │  │
│  │         IBookRepository, IUserService                    │  │
│  └────────────────────────────┬────────────────────────────┘  │
│                               │                                │
│  ┌────────────────────────────▼────────────────────────────┐  │
│  │                    Database (EF Core)                    │  │
│  └─────────────────────────────────────────────────────────┘  │
└───────────────────────────────────────────────────────────────┘
                │
                ▼
         JSON Response
```

---

## REST Principles

### What is REST?

REST (Representational State Transfer) is an architectural style for designing networked applications.

### 6 REST Principles

| Principle | Description |
|-----------|-------------|
| **Stateless** | Each request contains all information needed |
| **Client-Server** | Separation of concerns |
| **Uniform Interface** | Consistent URL structure |
| **Cacheable** | Responses can be cached |
| **Layered System** | Components cannot see beyond immediate layer |
| **Code on Demand** | Optional - server can send executable code |

### RESTful URL Design

```
RESOURCE: Books

┌───────────┬────────────────────┬──────────────────────────────┐
│ HTTP Verb │ URL                │ Action                       │
├───────────┼────────────────────┼──────────────────────────────┤
│ GET       │ /api/books         │ Get all books                │
│ GET       │ /api/books/5       │ Get book with id 5           │
│ POST      │ /api/books         │ Create new book              │
│ PUT       │ /api/books/5       │ Update book with id 5        │
│ DELETE    │ /api/books/5       │ Delete book with id 5        │
│ PATCH     │ /api/books/5       │ Partial update book 5        │
└───────────┴────────────────────┴──────────────────────────────┘

NESTED RESOURCES:

GET    /api/authors/3/books      → Get all books by author 3
POST   /api/authors/3/books      → Create book for author 3
```

---

## Creating Web API

### Project Setup

```csharp
// File: Program.cs

using AsynctwotableAPI.Repository;
using AsynctwotableAPI.Services;
using Microsoft.EntityFrameworkCore;

namespace AsynctwotableAPI
{
    public class Program
    {
        public static void Main(string[] args)
        {
            var builder = WebApplication.CreateBuilder(args);

            // ═══════════════════════════════════════════════════════
            // STEP 1: Add Services
            // ═══════════════════════════════════════════════════════
            
            // Line 1: Add API controllers
            builder.Services.AddControllers();
            
            // Line 2: Configure DbContext
            var connectionString = builder.Configuration
                .GetConnectionString("BookDatabase");
            builder.Services.AddDbContextPool<AppdbContext>(
                options => options.UseSqlServer(connectionString));
            
            // Line 3: Register repository
            builder.Services.AddTransient<IBookRepository, SqlBookRepository>();
            
            // Line 4: Add Swagger for API documentation
            builder.Services.AddEndpointsApiExplorer();
            builder.Services.AddSwaggerGen();

            var app = builder.Build();

            // ═══════════════════════════════════════════════════════
            // STEP 2: Configure Middleware Pipeline
            // ═══════════════════════════════════════════════════════
            
            // Line 5: Enable Swagger in development
            if (app.Environment.IsDevelopment())
            {
                app.UseSwagger();
                app.UseSwaggerUI();
            }

            app.UseHttpsRedirection();
            app.UseAuthorization();
            
            // Line 6: Map API controllers
            app.MapControllers();

            app.Run();
        }
    }
}
```

---

## HTTP Methods

### CRUD Operations Mapping

```
┌─────────────────────────────────────────────────────────────────┐
│                    HTTP METHODS (CRUD)                           │
└─────────────────────────────────────────────────────────────────┘

HTTP Method    CRUD Operation    SQL Equivalent    Description
──────────────────────────────────────────────────────────────────
GET            Read              SELECT            Retrieve resource
POST           Create            INSERT            Create new resource
PUT            Update (Full)     UPDATE            Replace entire resource
PATCH          Update (Partial)  UPDATE            Modify specific fields
DELETE         Delete            DELETE            Remove resource
──────────────────────────────────────────────────────────────────

Safe Methods: GET, HEAD, OPTIONS (no side effects)
Idempotent: GET, PUT, DELETE (same result on multiple calls)
```

### HTTP Method Attributes

```csharp
// File: Controllers/BookController.cs

[ApiController]
[Route("api/[controller]")]
public class BookController : ControllerBase
{
    private readonly IBookRepository _bookRepository;

    public BookController(IBookRepository bookRepository)
    {
        _bookRepository = bookRepository;
    }

    // ═══════════════════════════════════════════════════════════
    // GET: Retrieve data
    // ═══════════════════════════════════════════════════════════
    
    // GET: api/book
    [HttpGet]
    public async Task<ActionResult<IEnumerable<Book>>> GetAllBooks()
    {
        var books = await _bookRepository.GetAllAsync();
        return Ok(books);  // 200 OK
    }

    // GET: api/book/5
    [HttpGet("{id}")]
    public async Task<ActionResult<Book>> GetBookById(int id)
    {
        var book = await _bookRepository.GetByIdAsync(id);
        if (book == null)
            return NotFound();  // 404 Not Found

        return Ok(book);  // 200 OK
    }

    // ═══════════════════════════════════════════════════════════
    // POST: Create new resource
    // ═══════════════════════════════════════════════════════════
    
    // POST: api/book
    [HttpPost]
    public async Task<ActionResult<Book>> AddBook(Book book)
    {
        var createdBook = await _bookRepository.AddAsync(book);
        
        // 201 Created with location header
        return CreatedAtAction(
            nameof(GetBookById), 
            new { id = createdBook.BookId }, 
            createdBook);
    }

    // ═══════════════════════════════════════════════════════════
    // PUT: Update entire resource
    // ═══════════════════════════════════════════════════════════
    
    // PUT: api/book/5
    [HttpPut("{id}")]
    public async Task<ActionResult<Book>> UpdateBook(int id, Book book)
    {
        var updatedBook = await _bookRepository.UpdateAsync(id, book);
        if (updatedBook == null)
            return NotFound();  // 404 Not Found

        return Ok(updatedBook);  // 200 OK
    }

    // ═══════════════════════════════════════════════════════════
    // DELETE: Remove resource
    // ═══════════════════════════════════════════════════════════
    
    // DELETE: api/book/5
    [HttpDelete("{id}")]
    public async Task<ActionResult<Book>> DeleteBook(int id)
    {
        var deletedBook = await _bookRepository.DeleteAsync(id);
        if (deletedBook == null)
            return NotFound();  // 404 Not Found

        return Ok(deletedBook);  // 200 OK (or NoContent 204)
    }
}
```

---

## API Controller

### ControllerBase vs Controller

```csharp
// For Web API - Use ControllerBase
[ApiController]
public class BooksController : ControllerBase
{
    // Returns data (JSON/XML)
    // No view support
}

// For MVC - Use Controller
public class BooksController : Controller
{
    // Returns views (HTML)
    // Has ViewData, ViewBag, TempData
}
```

### [ApiController] Attribute Benefits

```csharp
[ApiController]  // Add this attribute
[Route("api/[controller]")]
public class BookController : ControllerBase
{
    // Benefits of [ApiController]:
    // 1. Automatic model validation (no need for ModelState.IsValid)
    // 2. Automatic 400 response for invalid model
    // 3. Binding source inference (body, route, query)
    // 4. Problem details for error responses
    
    [HttpPost]
    public IActionResult Create(Book book)
    {
        // No need for:
        // if (!ModelState.IsValid)
        //     return BadRequest(ModelState);
        
        // [ApiController] automatically validates and returns 400
        
        _repository.Add(book);
        return CreatedAtAction(nameof(Get), new { id = book.Id }, book);
    }
}
```

---

## Routing in API

### Attribute Routing

```csharp
[ApiController]
[Route("api/[controller]")]  // api/book
public class BookController : ControllerBase
{
    // ═══════════════════════════════════════════════════════════
    // ROUTE TEMPLATES
    // ═══════════════════════════════════════════════════════════
    
    // Route: GET api/book
    [HttpGet]
    public IActionResult GetAll() { }
    
    // Route: GET api/book/5
    [HttpGet("{id}")]
    public IActionResult GetById(int id) { }
    
    // Route: GET api/book/search?title=C#
    [HttpGet("search")]
    public IActionResult Search([FromQuery] string title) { }
    
    // Route: GET api/book/author/3
    [HttpGet("author/{authorId}")]
    public IActionResult GetByAuthor(int authorId) { }
    
    // Route: GET api/book/category/fiction/year/2024
    [HttpGet("category/{category}/year/{year}")]
    public IActionResult GetByCategoryAndYear(string category, int year) { }
}
```

### Route Constraints

```csharp
// ═══════════════════════════════════════════════════════════
// ROUTE CONSTRAINTS
// ═══════════════════════════════════════════════════════════

// Integer only
[HttpGet("{id:int}")]
public IActionResult GetById(int id) { }

// GUID only
[HttpGet("{id:guid}")]
public IActionResult GetByGuid(Guid id) { }

// Min/Max length
[HttpGet("search/{term:minlength(3):maxlength(50)}")]
public IActionResult Search(string term) { }

// Range
[HttpGet("page/{page:int:min(1):max(100)}")]
public IActionResult GetPage(int page) { }

// Regex
[HttpGet("isbn/{isbn:regex(^\\d{{13}}$)}")]
public IActionResult GetByIsbn(string isbn) { }
```

---

## Return Types

### Common HTTP Status Codes

```csharp
// ═══════════════════════════════════════════════════════════
// SUCCESS RESPONSES (2xx)
// ═══════════════════════════════════════════════════════════

return Ok(data);                    // 200 OK
return Created(uri, data);          // 201 Created
return CreatedAtAction(...);        // 201 Created with location
return NoContent();                 // 204 No Content
return Accepted();                  // 202 Accepted (async processing)

// ═══════════════════════════════════════════════════════════
// CLIENT ERROR RESPONSES (4xx)
// ═══════════════════════════════════════════════════════════

return BadRequest();                // 400 Bad Request
return BadRequest(ModelState);      // 400 with validation errors
return Unauthorized();              // 401 Unauthorized
return Forbid();                    // 403 Forbidden
return NotFound();                  // 404 Not Found
return Conflict();                  // 409 Conflict

// ═══════════════════════════════════════════════════════════
// SERVER ERROR RESPONSES (5xx)
// ═══════════════════════════════════════════════════════════

return StatusCode(500);                        // 500 Internal Server Error
return StatusCode(500, "Error message");       // 500 with message
```

### ActionResult<T> Return Type

```csharp
// ═══════════════════════════════════════════════════════════
// STRONGLY-TYPED RETURN
// ═══════════════════════════════════════════════════════════

[HttpGet("{id}")]
public async Task<ActionResult<Book>> GetBook(int id)
{
    var book = await _repository.GetByIdAsync(id);
    
    if (book == null)
        return NotFound();  // Returns 404
    
    return book;  // Returns 200 with book data
    // Or: return Ok(book);
}

// ═══════════════════════════════════════════════════════════
// COLLECTION RETURN
// ═══════════════════════════════════════════════════════════

[HttpGet]
public async Task<ActionResult<IEnumerable<Book>>> GetAllBooks()
{
    var books = await _repository.GetAllAsync();
    return Ok(books);  // Returns 200 with array
}
```

---

## Model Binding

### Binding Sources

```csharp
[ApiController]
[Route("api/[controller]")]
public class BookController : ControllerBase
{
    // ═══════════════════════════════════════════════════════════
    // FROM ROUTE
    // ═══════════════════════════════════════════════════════════
    
    [HttpGet("{id}")]
    public IActionResult Get([FromRoute] int id) { }

    // ═══════════════════════════════════════════════════════════
    // FROM QUERY STRING: /api/book?title=C#&author=John
    // ═══════════════════════════════════════════════════════════
    
    [HttpGet]
    public IActionResult Search(
        [FromQuery] string title,
        [FromQuery] string author) { }

    // ═══════════════════════════════════════════════════════════
    // FROM BODY (JSON)
    // ═══════════════════════════════════════════════════════════
    
    [HttpPost]
    public IActionResult Create([FromBody] Book book) { }

    // ═══════════════════════════════════════════════════════════
    // FROM HEADER
    // ═══════════════════════════════════════════════════════════
    
    [HttpGet]
    public IActionResult Get([FromHeader(Name = "X-Api-Key")] string apiKey) { }

    // ═══════════════════════════════════════════════════════════
    // FROM FORM
    // ═══════════════════════════════════════════════════════════
    
    [HttpPost("upload")]
    public IActionResult Upload([FromForm] IFormFile file) { }

    // ═══════════════════════════════════════════════════════════
    // COMBINED
    // ═══════════════════════════════════════════════════════════
    
    [HttpPut("{id}")]
    public IActionResult Update(
        [FromRoute] int id,
        [FromBody] Book book,
        [FromHeader(Name = "Authorization")] string token) { }
}
```

---

## Swagger Documentation

### Enabling Swagger

```csharp
// File: Program.cs

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();

// Line 1: Add Swagger services
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen(options =>
{
    options.SwaggerDoc("v1", new OpenApiInfo
    {
        Title = "Book API",
        Version = "v1",
        Description = "API for managing books"
    });
});

var app = builder.Build();

// Line 2: Enable Swagger UI in development
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI(options =>
    {
        options.SwaggerEndpoint("/swagger/v1/swagger.json", "Book API v1");
        options.RoutePrefix = string.Empty;  // Swagger at root URL
    });
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();
app.Run();
```

### Documenting API Endpoints

```csharp
using Microsoft.AspNetCore.Mvc;
using System.ComponentModel.DataAnnotations;

/// <summary>
/// Controller for managing books
/// </summary>
[ApiController]
[Route("api/[controller]")]
[Produces("application/json")]
public class BookController : ControllerBase
{
    /// <summary>
    /// Gets all books
    /// </summary>
    /// <returns>List of all books</returns>
    /// <response code="200">Returns the list of books</response>
    [HttpGet]
    [ProducesResponseType(StatusCodes.Status200OK)]
    public async Task<ActionResult<IEnumerable<Book>>> GetAll()
    {
        // ...
    }

    /// <summary>
    /// Gets a book by ID
    /// </summary>
    /// <param name="id">The book ID</param>
    /// <returns>The requested book</returns>
    /// <response code="200">Returns the book</response>
    /// <response code="404">Book not found</response>
    [HttpGet("{id}")]
    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<ActionResult<Book>> GetById(int id)
    {
        // ...
    }

    /// <summary>
    /// Creates a new book
    /// </summary>
    /// <param name="book">The book to create</param>
    /// <returns>The created book</returns>
    /// <response code="201">Book created successfully</response>
    /// <response code="400">Invalid input</response>
    [HttpPost]
    [ProducesResponseType(StatusCodes.Status201Created)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    public async Task<ActionResult<Book>> Create(Book book)
    {
        // ...
    }
}
```

---

## Code Examples

### Complete API Controller

```csharp
// File: Controllers/BookController.cs

using AsynctwotableAPI.Models;
using Microsoft.AspNetCore.Mvc;
using AsynctwotableAPI.Services;

namespace AsynctwotableAPI.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class BookController : ControllerBase
    {
        private readonly IBookRepository _bookRepository;
        private readonly ILogger<BookController> _logger;

        public BookController(
            IBookRepository bookRepository,
            ILogger<BookController> logger)
        {
            _bookRepository = bookRepository;
            _logger = logger;
        }

        // GET: api/book
        [HttpGet]
        public async Task<ActionResult<IEnumerable<Book>>> GetAllBooks()
        {
            _logger.LogInformation("Getting all books");
            var books = await _bookRepository.GetAllAsync();
            return Ok(books);
        }

        // GET: api/book/5
        [HttpGet("{id}")]
        public async Task<ActionResult<Book>> GetBookById(int id)
        {
            _logger.LogInformation("Getting book with id {Id}", id);
            var book = await _bookRepository.GetByIdAsync(id);
            
            if (book == null)
            {
                _logger.LogWarning("Book with id {Id} not found", id);
                return NotFound();
            }

            return Ok(book);
        }

        // POST: api/book
        [HttpPost]
        public async Task<ActionResult<Book>> AddBook(Book book)
        {
            _logger.LogInformation("Creating new book: {Title}", book.Title);
            var createdBook = await _bookRepository.AddAsync(book);
            
            return CreatedAtAction(
                nameof(GetBookById), 
                new { id = createdBook.BookId }, 
                createdBook);
        }

        // PUT: api/book/5
        [HttpPut("{id}")]
        public async Task<ActionResult<Book>> UpdateBook(int id, Book book)
        {
            if (id != book.BookId)
            {
                return BadRequest("ID mismatch");
            }

            _logger.LogInformation("Updating book with id {Id}", id);
            var updatedBook = await _bookRepository.UpdateAsync(id, book);
            
            if (updatedBook == null)
            {
                return NotFound();
            }

            return Ok(updatedBook);
        }

        // DELETE: api/book/5
        [HttpDelete("{id}")]
        public async Task<ActionResult<Book>> DeleteBook(int id)
        {
            _logger.LogInformation("Deleting book with id {Id}", id);
            var deletedBook = await _bookRepository.DeleteAsync(id);
            
            if (deletedBook == null)
            {
                return NotFound();
            }

            return Ok(deletedBook);
        }
    }
}
```

### Repository Implementation

```csharp
// File: Services/SqlBookRepository.cs

public class SqlBookRepository : IBookRepository
{
    private readonly AppdbContext _context;

    public SqlBookRepository(AppdbContext context)
    {
        _context = context;
    }

    public async Task<Book> AddAsync(Book book)
    {
        await _context.Books.AddAsync(book);
        await _context.SaveChangesAsync();
        return book;
    }

    public async Task<Book?> GetByIdAsync(int id)
    {
        return await _context.Books.FindAsync(id);
    }

    public async Task<IEnumerable<Book>> GetAllAsync()
    {
        return await _context.Books.ToListAsync();
    }

    public async Task<Book?> UpdateAsync(int id, Book book)
    {
        var existingBook = await _context.Books.FindAsync(id);
        if (existingBook == null) return null;

        existingBook.Title = book.Title;
        existingBook.Author = book.Author;
        existingBook.Price = book.Price;

        await _context.SaveChangesAsync();
        return existingBook;
    }

    public async Task<Book?> DeleteAsync(int id)
    {
        var book = await _context.Books.FindAsync(id);
        if (book == null) return null;

        _context.Books.Remove(book);
        await _context.SaveChangesAsync();
        return book;
    }
}
```

---

## Best Practices

### 1. Use Proper HTTP Status Codes

```csharp
// ✅ GOOD: Appropriate status codes
return NotFound();           // 404 when resource doesn't exist
return CreatedAtAction(...); // 201 when resource created
return NoContent();          // 204 for successful delete

// ❌ BAD: Generic response
return Ok("Not found");      // Should be 404, not 200
```

### 2. Use Async/Await for I/O Operations

```csharp
// ✅ GOOD: Async database operations
public async Task<ActionResult<Book>> Get(int id)
{
    var book = await _repository.GetByIdAsync(id);
    return Ok(book);
}

// ❌ BAD: Blocking synchronous call
public ActionResult<Book> Get(int id)
{
    var book = _repository.GetById(id);  // Blocks thread
    return Ok(book);
}
```

### 3. Validate Input

```csharp
// ✅ GOOD: Validation with data annotations
public class BookDto
{
    [Required]
    [StringLength(200)]
    public string Title { get; set; }
    
    [Range(0.01, 1000)]
    public decimal Price { get; set; }
}
```

### 4. Use DTOs Instead of Entities

```csharp
// ✅ GOOD: Separate DTO for API
public class BookResponseDto
{
    public int Id { get; set; }
    public string Title { get; set; }
    // Don't expose internal properties
}

// ❌ BAD: Exposing entity directly
return Ok(bookEntity);  // May expose sensitive data
```

---

## Interview Questions

### Q1: What is the difference between Controller and ControllerBase?
**Answer:**
- `Controller`: For MVC, includes view support (ViewData, ViewBag, View())
- `ControllerBase`: For Web API, lightweight, data responses only (JSON/XML)

### Q2: What does [ApiController] attribute do?
**Answer:** Provides:
- Automatic model validation (400 for invalid)
- Binding source inference
- Problem details for errors
- Multipart/form-data inference

### Q3: What is the difference between PUT and PATCH?
**Answer:**
- `PUT`: Replaces entire resource
- `PATCH`: Partially updates resource (only changed fields)

### Q4: What is CreatedAtAction?
**Answer:** Returns 201 Created with a Location header pointing to the newly created resource. Takes action name, route values, and the created object.

### Q5: How do you handle versioning in Web API?
**Answer:** Options:
- URL path: `/api/v1/books`
- Query string: `/api/books?version=1`
- Header: `X-API-Version: 1`
- Media type: `Accept: application/json;v=1`

### Q6: What is content negotiation?
**Answer:** The process where client and server agree on response format (JSON, XML) using Accept header. ASP.NET Core defaults to JSON.

---

## Quick Reference

### HTTP Methods

```
GET     → Read (SELECT)
POST    → Create (INSERT)
PUT     → Update full (UPDATE)
PATCH   → Update partial
DELETE  → Delete (DELETE)
```

### Common Status Codes

```
200 OK              → Success
201 Created         → Resource created
204 No Content      → Success, no body
400 Bad Request     → Invalid input
401 Unauthorized    → Not authenticated
403 Forbidden       → Not authorized
404 Not Found       → Resource missing
500 Server Error    → Server failure
```

### Controller Template

```csharp
[ApiController]
[Route("api/[controller]")]
public class ItemsController : ControllerBase
{
    [HttpGet]
    public async Task<ActionResult<IEnumerable<Item>>> GetAll() { }
    
    [HttpGet("{id}")]
    public async Task<ActionResult<Item>> GetById(int id) { }
    
    [HttpPost]
    public async Task<ActionResult<Item>> Create(Item item) { }
    
    [HttpPut("{id}")]
    public async Task<IActionResult> Update(int id, Item item) { }
    
    [HttpDelete("{id}")]
    public async Task<IActionResult> Delete(int id) { }
}
```
