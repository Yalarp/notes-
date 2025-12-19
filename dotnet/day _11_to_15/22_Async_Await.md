# Async and Await in ASP.NET Core

## Table of Contents
1. [Introduction](#introduction)
2. [Why Async Programming?](#why-async-programming)
3. [Async/Await Basics](#asyncawait-basics)
4. [Task and Task<T>](#task-and-taskt)
5. [Async in Controllers](#async-in-controllers)
6. [Async Repository Pattern](#async-repository-pattern)
7. [Common Patterns](#common-patterns)
8. [Error Handling](#error-handling)
9. [Code Examples](#code-examples)
10. [Best Practices](#best-practices)
11. [Interview Questions](#interview-questions)

---

## Introduction

### Real-World Analogy
Async programming is like a **restaurant kitchen**:

```
Synchronous (Blocking):
    Chef 1 takes order → Cooks dish → Waits → Serves
    Chef 2 idle until Chef 1 finishes
    ❌ One chef at a time, slow service

Asynchronous (Non-Blocking):
    Chef 1 takes order → Starts cooking → While waiting for oven...
    Chef 2 takes another order → Starts cooking
    Chef 1 finishes → Serves → Takes new order
    ✅ Multiple dishes in progress, fast service
```

---

## Why Async Programming?

### The Problem with Synchronous Code

```csharp
// Synchronous - BLOCKS the thread
public Book GetBook(int id)
{
    // Thread waits here doing NOTHING while database responds
    var book = _context.Books.Find(id);  // Blocking I/O
    return book;
}
```

```
┌─────────────────────────────────────────────────────────────────┐
│                 SYNCHRONOUS REQUEST HANDLING                     │
└─────────────────────────────────────────────────────────────────┘

Thread Pool: [Thread 1] [Thread 2] [Thread 3] [Thread 4]

Request 1 → Thread 1 blocks waiting for DB ████████████████
Request 2 → Thread 2 blocks waiting for DB ████████████████
Request 3 → Thread 3 blocks waiting for DB ████████████████
Request 4 → Thread 4 blocks waiting for DB ████████████████
Request 5 → ❌ NO THREADS AVAILABLE! Request queued...

Result: Thread starvation under load
```

### The Solution: Async

```csharp
// Asynchronous - RELEASES the thread
public async Task<Book> GetBookAsync(int id)
{
    // Thread is released while waiting for database
    var book = await _context.Books.FindAsync(id);  // Non-blocking
    return book;
}
```

```
┌─────────────────────────────────────────────────────────────────┐
│                 ASYNCHRONOUS REQUEST HANDLING                    │
└─────────────────────────────────────────────────────────────────┘

Thread Pool: [Thread 1] [Thread 2] [Thread 3] [Thread 4]

Request 1 → Thread 1 starts DB call → Releases thread
Request 2 → Thread 1 starts DB call → Releases thread
Request 3 → Thread 2 starts DB call → Releases thread
Request 4 → Thread 3 starts DB call → Releases thread
Request 5 → Thread 4 starts DB call → Releases thread

DB Response 1 → Any thread continues processing
DB Response 2 → Any thread continues processing

Result: ✅ High throughput, efficient resource usage
```

### Benefits of Async

| Benefit | Description |
|---------|-------------|
| **Scalability** | Handle more requests with same resources |
| **Responsiveness** | UI remains responsive during I/O |
| **Resource Efficiency** | Threads released during waiting |
| **Throughput** | More operations per second |

---

## Async/Await Basics

### Keywords Explained

```csharp
// ═══════════════════════════════════════════════════════════════
// async - Marks method as asynchronous
// await - Waits for task without blocking
// ═══════════════════════════════════════════════════════════════

public async Task<string> GetDataAsync()
{
    // Line 1: 'async' enables use of 'await' in method
    // Method returns Task<string> (not string)
    
    // Line 2: 'await' pauses execution until task completes
    // BUT releases thread to do other work
    var result = await httpClient.GetStringAsync("https://api.example.com");
    
    // Line 3: Execution continues when result is ready
    return result;
}
```

### Execution Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                    ASYNC/AWAIT EXECUTION FLOW                    │
└─────────────────────────────────────────────────────────────────┘

public async Task<Book> GetBookAsync(int id)
{
    Console.WriteLine("1. Before await");      ← Thread A
    
    var book = await _context.Books.FindAsync(id);
    //         │
    //         ├─→ Thread A released to pool
    //         │
    //         │   (Database working...)
    //         │
    //         └─→ Any thread (maybe A, maybe B) continues
    
    Console.WriteLine("2. After await");       ← Thread A or B
    return book;
}

Timeline:
────────────────────────────────────────────────────────────
Thread A: [Start] → [await] → [Released to pool]
                              
                    ↓ (Database I/O happening)
                              
Thread B: ────────────────────→ [Continues] → [Return]
────────────────────────────────────────────────────────────
```

---

## Task and Task<T>

### Return Types

```csharp
// ═══════════════════════════════════════════════════════════════
// Task - Async method with no return value (like void)
// ═══════════════════════════════════════════════════════════════

public async Task SendEmailAsync(string to, string message)
{
    await emailService.SendAsync(to, message);
    // No return statement needed
}

// ═══════════════════════════════════════════════════════════════
// Task<T> - Async method returning type T
// ═══════════════════════════════════════════════════════════════

public async Task<Book> GetBookAsync(int id)
{
    var book = await _context.Books.FindAsync(id);
    return book;  // Returns Book, wrapped in Task<Book>
}

// ═══════════════════════════════════════════════════════════════
// ValueTask<T> - Lightweight alternative (for hot paths)
// ═══════════════════════════════════════════════════════════════

public async ValueTask<Book> GetCachedBookAsync(int id)
{
    if (_cache.TryGetValue(id, out Book book))
        return book;  // No allocation if cached
    
    return await _context.Books.FindAsync(id);
}
```

### Creating Tasks

```csharp
// ═══════════════════════════════════════════════════════════════
// Task.Run - CPU-bound work (avoid in ASP.NET Core)
// ═══════════════════════════════════════════════════════════════

// ❌ Usually NOT needed in ASP.NET Core
var result = await Task.Run(() => HeavyCpuWork());

// ═══════════════════════════════════════════════════════════════
// Task.FromResult - Synchronously completed task
// ═══════════════════════════════════════════════════════════════

public Task<int> GetDefaultAsync()
{
    return Task.FromResult(42);  // Already completed, no async needed
}

// ═══════════════════════════════════════════════════════════════
// Task.CompletedTask - Void task that's already complete
// ═══════════════════════════════════════════════════════════════

public Task DoNothingAsync()
{
    return Task.CompletedTask;
}

// ═══════════════════════════════════════════════════════════════
// Task.WhenAll - Parallel execution
// ═══════════════════════════════════════════════════════════════

public async Task<(Book, Author)> GetBookAndAuthorAsync(int bookId, int authorId)
{
    var bookTask = _bookRepo.GetByIdAsync(bookId);
    var authorTask = _authorRepo.GetByIdAsync(authorId);
    
    await Task.WhenAll(bookTask, authorTask);  // Run in parallel
    
    return (bookTask.Result, authorTask.Result);
}

// ═══════════════════════════════════════════════════════════════
// Task.WhenAny - First completed task
// ═══════════════════════════════════════════════════════════════

public async Task<string> GetFastestResponseAsync()
{
    var task1 = api1.GetDataAsync();
    var task2 = api2.GetDataAsync();
    
    var firstCompleted = await Task.WhenAny(task1, task2);
    return await firstCompleted;
}
```

---

## Async in Controllers

### Async Controller Actions

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
    // ASYNC GET ALL
    // ═══════════════════════════════════════════════════════════
    
    [HttpGet]
    public async Task<ActionResult<IEnumerable<Book>>> GetAllBooks()
    {
        // Line 1: Async database call
        var books = await _bookRepository.GetAllAsync();
        
        // Line 2: Return result
        return Ok(books);
    }

    // ═══════════════════════════════════════════════════════════
    // ASYNC GET BY ID
    // ═══════════════════════════════════════════════════════════
    
    [HttpGet("{id}")]
    public async Task<ActionResult<Book>> GetBookById(int id)
    {
        var book = await _bookRepository.GetByIdAsync(id);
        
        if (book == null)
            return NotFound();

        return Ok(book);
    }

    // ═══════════════════════════════════════════════════════════
    // ASYNC CREATE
    // ═══════════════════════════════════════════════════════════
    
    [HttpPost]
    public async Task<ActionResult<Book>> AddBook(Book book)
    {
        var createdBook = await _bookRepository.AddAsync(book);
        
        return CreatedAtAction(
            nameof(GetBookById), 
            new { id = createdBook.BookId }, 
            createdBook);
    }

    // ═══════════════════════════════════════════════════════════
    // ASYNC UPDATE
    // ═══════════════════════════════════════════════════════════
    
    [HttpPut("{id}")]
    public async Task<ActionResult<Book>> UpdateBook(int id, Book book)
    {
        var updatedBook = await _bookRepository.UpdateAsync(id, book);
        
        if (updatedBook == null)
            return NotFound();

        return Ok(updatedBook);
    }

    // ═══════════════════════════════════════════════════════════
    // ASYNC DELETE
    // ═══════════════════════════════════════════════════════════
    
    [HttpDelete("{id}")]
    public async Task<ActionResult<Book>> DeleteBook(int id)
    {
        var deletedBook = await _bookRepository.DeleteAsync(id);
        
        if (deletedBook == null)
            return NotFound();

        return Ok(deletedBook);
    }
}
```

---

## Async Repository Pattern

### Interface Definition

```csharp
// File: Services/IBookRepository.cs

public interface IBookRepository
{
    // All methods return Task for async operations
    Task<Book> AddAsync(Book book);
    Task<Book?> GetByIdAsync(int id);
    Task<IEnumerable<Book>> GetAllAsync();
    Task<Book?> UpdateAsync(int id, Book book);
    Task<Book?> DeleteAsync(int id);
}
```

### Implementation

```csharp
// File: Services/SqlBookRepository.cs

public class SqlBookRepository : IBookRepository
{
    private readonly AppdbContext _context;

    public SqlBookRepository(AppdbContext context)
    {
        _context = context;
    }

    // ═══════════════════════════════════════════════════════════
    // ADD ASYNC
    // ═══════════════════════════════════════════════════════════
    
    public async Task<Book> AddAsync(Book book)
    {
        // Line 1: AddAsync is truly async
        await _context.Books.AddAsync(book);
        
        // Line 2: SaveChangesAsync commits to database
        await _context.SaveChangesAsync();
        
        return book;
    }

    // ═══════════════════════════════════════════════════════════
    // GET BY ID ASYNC
    // ═══════════════════════════════════════════════════════════
    
    public async Task<Book?> GetByIdAsync(int id)
    {
        // FindAsync is async version of Find
        return await _context.Books.FindAsync(id);
    }

    // ═══════════════════════════════════════════════════════════
    // GET ALL ASYNC
    // ═══════════════════════════════════════════════════════════
    
    public async Task<IEnumerable<Book>> GetAllAsync()
    {
        // ToListAsync converts query to list asynchronously
        return await _context.Books.ToListAsync();
    }

    // ═══════════════════════════════════════════════════════════
    // UPDATE ASYNC
    // ═══════════════════════════════════════════════════════════
    
    public async Task<Book?> UpdateAsync(int id, Book book)
    {
        var existingBook = await _context.Books.FindAsync(id);
        
        if (existingBook == null)
            return null;

        existingBook.Title = book.Title;
        existingBook.Author = book.Author;
        existingBook.Price = book.Price;

        await _context.SaveChangesAsync();
        return existingBook;
    }

    // ═══════════════════════════════════════════════════════════
    // DELETE ASYNC
    // ═══════════════════════════════════════════════════════════
    
    public async Task<Book?> DeleteAsync(int id)
    {
        var book = await _context.Books.FindAsync(id);
        
        if (book == null)
            return null;

        _context.Books.Remove(book);
        await _context.SaveChangesAsync();
        
        return book;
    }
}
```

---

## Common Patterns

### Parallel Execution

```csharp
// ═══════════════════════════════════════════════════════════════
// PATTERN 1: Parallel independent tasks
// ═══════════════════════════════════════════════════════════════

public async Task<DashboardViewModel> GetDashboardAsync()
{
    // Start all tasks (don't await yet)
    var booksTask = _bookRepo.GetAllAsync();
    var authorsTask = _authorRepo.GetAllAsync();
    var ordersTask = _orderRepo.GetRecentAsync();
    
    // Wait for all to complete in parallel
    await Task.WhenAll(booksTask, authorsTask, ordersTask);
    
    return new DashboardViewModel
    {
        Books = booksTask.Result,
        Authors = authorsTask.Result,
        Orders = ordersTask.Result
    };
}

// Time comparison:
// Sequential: Task1 (2s) + Task2 (2s) + Task3 (2s) = 6 seconds
// Parallel:   Max(Task1, Task2, Task3) = 2 seconds
```

### Sequential with Dependency

```csharp
// ═══════════════════════════════════════════════════════════════
// PATTERN 2: Sequential when tasks depend on each other
// ═══════════════════════════════════════════════════════════════

public async Task<OrderDetails> ProcessOrderAsync(int orderId)
{
    // Step 1: Get order
    var order = await _orderRepo.GetByIdAsync(orderId);
    
    // Step 2: Get customer (depends on order)
    var customer = await _customerRepo.GetByIdAsync(order.CustomerId);
    
    // Step 3: Get products (depends on order)
    var products = await _productRepo.GetByIdsAsync(order.ProductIds);
    
    return new OrderDetails { Order = order, Customer = customer, Products = products };
}
```

### Async Enumerable (C# 8+)

```csharp
// ═══════════════════════════════════════════════════════════════
// PATTERN 3: Streaming results as they come
// ═══════════════════════════════════════════════════════════════

public async IAsyncEnumerable<Book> StreamBooksAsync()
{
    await foreach (var book in _context.Books.AsAsyncEnumerable())
    {
        yield return book;  // Stream each book as it's loaded
    }
}

// Usage:
await foreach (var book in repo.StreamBooksAsync())
{
    Console.WriteLine(book.Title);
}
```

---

## Error Handling

### Try-Catch with Async

```csharp
public async Task<ActionResult<Book>> GetBookAsync(int id)
{
    try
    {
        var book = await _repository.GetByIdAsync(id);
        
        if (book == null)
            return NotFound();
            
        return Ok(book);
    }
    catch (SqlException ex)
    {
        _logger.LogError(ex, "Database error getting book {Id}", id);
        return StatusCode(500, "Database error");
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Error getting book {Id}", id);
        return StatusCode(500, "An error occurred");
    }
}
```

### Handling Multiple Task Exceptions

```csharp
public async Task ProcessMultipleAsync()
{
    var tasks = new[]
    {
        ProcessItem1Async(),
        ProcessItem2Async(),
        ProcessItem3Async()
    };
    
    try
    {
        await Task.WhenAll(tasks);
    }
    catch (Exception)
    {
        // WhenAll throws first exception only
        // Check all tasks for exceptions
        foreach (var task in tasks)
        {
            if (task.IsFaulted)
            {
                _logger.LogError(task.Exception, "Task failed");
            }
        }
    }
}
```

---

## Code Examples

### Complete Async Service

```csharp
// File: Services/BookService.cs

public class BookService : IBookService
{
    private readonly IBookRepository _repository;
    private readonly ILogger<BookService> _logger;
    private readonly IMemoryCache _cache;

    public BookService(
        IBookRepository repository,
        ILogger<BookService> logger,
        IMemoryCache cache)
    {
        _repository = repository;
        _logger = logger;
        _cache = cache;
    }

    public async Task<Book?> GetByIdAsync(int id)
    {
        var cacheKey = $"book_{id}";
        
        // Try cache first
        if (_cache.TryGetValue(cacheKey, out Book cachedBook))
        {
            _logger.LogDebug("Book {Id} retrieved from cache", id);
            return cachedBook;
        }
        
        // Get from database
        var book = await _repository.GetByIdAsync(id);
        
        if (book != null)
        {
            // Store in cache
            var cacheOptions = new MemoryCacheEntryOptions()
                .SetSlidingExpiration(TimeSpan.FromMinutes(5));
            _cache.Set(cacheKey, book, cacheOptions);
        }
        
        return book;
    }

    public async Task<BookWithAuthor> GetBookWithAuthorAsync(int bookId)
    {
        // Parallel fetch - book and author at same time
        var bookTask = _repository.GetByIdAsync(bookId);
        
        var book = await bookTask;
        if (book == null)
            return null;
            
        var authorTask = _authorRepository.GetByIdAsync(book.AuthorId);
        var author = await authorTask;
        
        return new BookWithAuthor { Book = book, Author = author };
    }

    public async Task<IEnumerable<Book>> SearchAsync(string term)
    {
        _logger.LogInformation("Searching books with term: {Term}", term);
        
        return await _repository.SearchAsync(term);
    }
}
```

---

## Best Practices

### 1. Always Use Async All the Way Down

```csharp
// ✅ GOOD: Async all the way
public async Task<Book> GetBookAsync(int id)
{
    return await _repository.GetByIdAsync(id);
}

// ❌ BAD: Blocking on async (deadlock risk!)
public Book GetBook(int id)
{
    return _repository.GetByIdAsync(id).Result;  // BLOCKS!
}

// ❌ BAD: .Wait() is also blocking
public void Process()
{
    _repository.SaveAsync().Wait();  // BLOCKS!
}
```

### 2. Avoid async void

```csharp
// ✅ GOOD: async Task
public async Task ProcessAsync()
{
    await DoWorkAsync();
}

// ❌ BAD: async void (can't catch exceptions!)
public async void ProcessAsync()
{
    await DoWorkAsync();  // Exceptions lost!
}

// Exception: Event handlers can use async void
private async void Button_Click(object sender, EventArgs e)
{
    await ProcessAsync();
}
```

### 3. Use ConfigureAwait When Appropriate

```csharp
// In library code (no UI context needed)
public async Task<byte[]> DownloadAsync(string url)
{
    var response = await httpClient.GetAsync(url)
        .ConfigureAwait(false);  // Don't capture context
    
    return await response.Content.ReadAsByteArrayAsync()
        .ConfigureAwait(false);
}
```

### 4. Don't Use Task.Run for I/O

```csharp
// ❌ BAD: Unnecessary thread hop
public async Task<Book> GetBookAsync(int id)
{
    return await Task.Run(() => _context.Books.Find(id));
}

// ✅ GOOD: Use async I/O directly
public async Task<Book> GetBookAsync(int id)
{
    return await _context.Books.FindAsync(id);
}
```

### 5. Cancel Long Operations

```csharp
public async Task<IEnumerable<Book>> SearchAsync(
    string term, 
    CancellationToken cancellationToken = default)
{
    return await _context.Books
        .Where(b => b.Title.Contains(term))
        .ToListAsync(cancellationToken);  // Pass token
}
```

---

## Interview Questions

### Q1: What is the difference between async/await and Task.Run?
**Answer:**
- `async/await`: For I/O-bound operations (database, HTTP, file). Releases thread during wait.
- `Task.Run`: For CPU-bound operations. Creates new thread pool thread.

### Q2: What is the difference between Task and ValueTask?
**Answer:**
- `Task`: Allocates memory, suitable for most cases
- `ValueTask`: Avoids allocation when result is immediately available (cached results). Use for hot paths.

### Q3: What happens when you don't await an async method?
**Answer:** The method runs in a "fire and forget" manner. Exceptions are lost, and you don't know when it completes. Usually a bug.

### Q4: What is thread starvation?
**Answer:** When all thread pool threads are blocked waiting for synchronous I/O, leaving no threads to handle new requests. Async/await prevents this.

### Q5: Why should you avoid .Result and .Wait()?
**Answer:** They block the calling thread, which can cause:
- Deadlocks (especially in UI/ASP.NET contexts)
- Thread starvation
- Reduced scalability

### Q6: What is ConfigureAwait(false)?
**Answer:** Tells the runtime not to capture and resume on the original synchronization context. Useful in libraries to avoid deadlocks and improve performance.

---

## Quick Reference

### Async Method Signatures

```csharp
// No return value
async Task MethodAsync()

// Return value
async Task<T> MethodAsync()

// Lightweight (for hot paths)
async ValueTask<T> MethodAsync()

// Streaming
async IAsyncEnumerable<T> MethodAsync()
```

### Common Async Methods

```csharp
// EF Core
await _context.SaveChangesAsync();
await _context.Books.FindAsync(id);
await _context.Books.ToListAsync();
await _context.Books.FirstOrDefaultAsync(x => x.Id == id);

// HTTP
await httpClient.GetAsync(url);
await httpClient.GetStringAsync(url);
await httpClient.PostAsJsonAsync(url, data);

// File
await File.ReadAllTextAsync(path);
await File.WriteAllTextAsync(path, content);
```

### Parallel Patterns

```csharp
// Wait for all
await Task.WhenAll(task1, task2, task3);

// Wait for first
await Task.WhenAny(task1, task2);

// Delay
await Task.Delay(1000);
```
