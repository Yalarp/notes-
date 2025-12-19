# Repository Pattern in ASP.NET Core

## Table of Contents
1. [Introduction](#introduction)
2. [What is Repository Pattern?](#what-is-repository-pattern)
3. [Benefits of Repository Pattern](#benefits-of-repository-pattern)
4. [Creating Repository Interface](#creating-repository-interface)
5. [Implementing Repository](#implementing-repository)
6. [Generic Repository](#generic-repository)
7. [Unit of Work Pattern](#unit-of-work-pattern)
8. [Dependency Injection Registration](#dependency-injection-registration)
9. [Code Examples](#code-examples)
10. [Best Practices](#best-practices)
11. [Interview Questions](#interview-questions)

---

## Introduction

### Real-World Analogy
Repository Pattern is like a **librarian**:

```
Without Repository:
    You → Go to every shelf → Search books yourself
    ❌ Need to know library layout
    ❌ Hard to find books
    ❌ Different process for different libraries

With Repository:
    You → Ask librarian → Get books
    ✅ Don't need to know internal layout
    ✅ Librarian knows where everything is
    ✅ Same interface at every library
```

---

## What is Repository Pattern?

Repository Pattern is a design pattern that creates an abstraction layer between the data access logic and the business logic. It encapsulates the logic required to access data sources.

### Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                    REPOSITORY PATTERN ARCHITECTURE               │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                        Controllers                               │
│                    (Business Logic)                              │
│         BookController, ProductController, etc.                  │
└───────────────────────────┬─────────────────────────────────────┘
                            │
                            │ Depends on Interface
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Repository Interfaces                         │
│                      (Abstraction)                               │
│           IBookRepository, IProductRepository, etc.              │
└───────────────────────────┬─────────────────────────────────────┘
                            │
                            │ Implements
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                 Repository Implementations                       │
│                    (Data Access)                                 │
│         SqlBookRepository, SqlProductRepository, etc.            │
└───────────────────────────┬─────────────────────────────────────┘
                            │
                            │ Uses
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                      DbContext / ORM                             │
│                    (Entity Framework Core)                       │
└───────────────────────────┬─────────────────────────────────────┘
                            │
                            ▼
                     ┌─────────────┐
                     │  Database   │
                     └─────────────┘
```

---

## Benefits of Repository Pattern

| Benefit | Description |
|---------|-------------|
| **Separation of Concerns** | Data access logic separated from business logic |
| **Testability** | Easy to mock repositories for unit testing |
| **Maintainability** | Changes to data access don't affect business logic |
| **Abstraction** | Business logic doesn't need to know about EF Core |
| **Centralized Queries** | All data access logic in one place |
| **Swappable Data Sources** | Can switch databases without changing business logic |

### With vs Without Repository

```csharp
// ═══════════════════════════════════════════════════════════════
// WITHOUT REPOSITORY (Controller directly uses DbContext)
// ═══════════════════════════════════════════════════════════════

public class BookController : ControllerBase
{
    private readonly AppDbContext _context;  // Tight coupling!
    
    public BookController(AppDbContext context)
    {
        _context = context;
    }
    
    [HttpGet]
    public async Task<IActionResult> GetAll()
    {
        // EF Core code mixed with controller logic
        var books = await _context.Books
            .Include(b => b.Author)
            .Where(b => b.IsActive)
            .OrderBy(b => b.Title)
            .ToListAsync();
        
        return Ok(books);
    }
}

// ═══════════════════════════════════════════════════════════════
// WITH REPOSITORY (Controller uses interface)
// ═══════════════════════════════════════════════════════════════

public class BookController : ControllerBase
{
    private readonly IBookRepository _repository;  // Loose coupling!
    
    public BookController(IBookRepository repository)
    {
        _repository = repository;
    }
    
    [HttpGet]
    public async Task<IActionResult> GetAll()
    {
        // Clean, simple call
        var books = await _repository.GetAllActiveAsync();
        return Ok(books);
    }
}
```

---

## Creating Repository Interface

### Entity-Specific Interface

```csharp
// File: Repositories/IBookRepository.cs

public interface IBookRepository
{
    // ═══════════════════════════════════════════════════════════
    // CRUD OPERATIONS
    // ═══════════════════════════════════════════════════════════
    
    Task<Book> AddAsync(Book book);
    Task<Book?> GetByIdAsync(int id);
    Task<IEnumerable<Book>> GetAllAsync();
    Task<Book?> UpdateAsync(int id, Book book);
    Task<Book?> DeleteAsync(int id);
    
    // ═══════════════════════════════════════════════════════════
    // CUSTOM QUERIES
    // ═══════════════════════════════════════════════════════════
    
    Task<IEnumerable<Book>> GetByAuthorAsync(int authorId);
    Task<IEnumerable<Book>> GetByGenreAsync(string genre);
    Task<IEnumerable<Book>> SearchAsync(string searchTerm);
    Task<bool> ExistsAsync(int id);
}
```

---

## Implementing Repository

### SQL Repository Implementation

```csharp
// File: Repositories/SqlBookRepository.cs

public class SqlBookRepository : IBookRepository
{
    private readonly AppdbContext _context;
    private readonly ILogger<SqlBookRepository> _logger;

    public SqlBookRepository(
        AppdbContext context,
        ILogger<SqlBookRepository> logger)
    {
        _context = context;
        _logger = logger;
    }

    // ═══════════════════════════════════════════════════════════
    // ADD
    // ═══════════════════════════════════════════════════════════
    
    public async Task<Book> AddAsync(Book book)
    {
        _logger.LogInformation("Adding book: {Title}", book.Title);
        
        await _context.Books.AddAsync(book);
        await _context.SaveChangesAsync();
        
        return book;
    }

    // ═══════════════════════════════════════════════════════════
    // GET BY ID
    // ═══════════════════════════════════════════════════════════
    
    public async Task<Book?> GetByIdAsync(int id)
    {
        return await _context.Books
            .Include(b => b.Author)
            .FirstOrDefaultAsync(b => b.BookId == id);
    }

    // ═══════════════════════════════════════════════════════════
    // GET ALL
    // ═══════════════════════════════════════════════════════════
    
    public async Task<IEnumerable<Book>> GetAllAsync()
    {
        return await _context.Books
            .Include(b => b.Author)
            .OrderBy(b => b.Title)
            .ToListAsync();
    }

    // ═══════════════════════════════════════════════════════════
    // UPDATE
    // ═══════════════════════════════════════════════════════════
    
    public async Task<Book?> UpdateAsync(int id, Book book)
    {
        var existingBook = await _context.Books.FindAsync(id);
        
        if (existingBook == null)
        {
            _logger.LogWarning("Book {Id} not found for update", id);
            return null;
        }

        existingBook.Title = book.Title;
        existingBook.ISBN = book.ISBN;
        existingBook.Price = book.Price;
        existingBook.AuthorId = book.AuthorId;

        await _context.SaveChangesAsync();
        
        _logger.LogInformation("Updated book: {Id}", id);
        return existingBook;
    }

    // ═══════════════════════════════════════════════════════════
    // DELETE
    // ═══════════════════════════════════════════════════════════
    
    public async Task<Book?> DeleteAsync(int id)
    {
        var book = await _context.Books.FindAsync(id);
        
        if (book == null)
            return null;

        _context.Books.Remove(book);
        await _context.SaveChangesAsync();
        
        _logger.LogInformation("Deleted book: {Id}", id);
        return book;
    }

    // ═══════════════════════════════════════════════════════════
    // CUSTOM QUERIES
    // ═══════════════════════════════════════════════════════════
    
    public async Task<IEnumerable<Book>> GetByAuthorAsync(int authorId)
    {
        return await _context.Books
            .Where(b => b.AuthorId == authorId)
            .ToListAsync();
    }

    public async Task<IEnumerable<Book>> SearchAsync(string searchTerm)
    {
        return await _context.Books
            .Where(b => b.Title.Contains(searchTerm) || 
                        b.Author.Name.Contains(searchTerm))
            .ToListAsync();
    }

    public async Task<bool> ExistsAsync(int id)
    {
        return await _context.Books.AnyAsync(b => b.BookId == id);
    }
}
```

---

## Generic Repository

### Generic Interface

```csharp
// File: Repositories/IRepository.cs

public interface IRepository<T> where T : class
{
    Task<T> AddAsync(T entity);
    Task<T?> GetByIdAsync(int id);
    Task<IEnumerable<T>> GetAllAsync();
    Task<T?> UpdateAsync(T entity);
    Task<bool> DeleteAsync(int id);
    Task<bool> ExistsAsync(int id);
    
    // Advanced queries
    Task<IEnumerable<T>> FindAsync(Expression<Func<T, bool>> predicate);
    Task<T?> FirstOrDefaultAsync(Expression<Func<T, bool>> predicate);
}
```

### Generic Implementation

```csharp
// File: Repositories/Repository.cs

public class Repository<T> : IRepository<T> where T : class
{
    protected readonly AppDbContext _context;
    protected readonly DbSet<T> _dbSet;

    public Repository(AppDbContext context)
    {
        _context = context;
        _dbSet = context.Set<T>();
    }

    public virtual async Task<T> AddAsync(T entity)
    {
        await _dbSet.AddAsync(entity);
        await _context.SaveChangesAsync();
        return entity;
    }

    public virtual async Task<T?> GetByIdAsync(int id)
    {
        return await _dbSet.FindAsync(id);
    }

    public virtual async Task<IEnumerable<T>> GetAllAsync()
    {
        return await _dbSet.ToListAsync();
    }

    public virtual async Task<T?> UpdateAsync(T entity)
    {
        _dbSet.Update(entity);
        await _context.SaveChangesAsync();
        return entity;
    }

    public virtual async Task<bool> DeleteAsync(int id)
    {
        var entity = await _dbSet.FindAsync(id);
        if (entity == null) return false;
        
        _dbSet.Remove(entity);
        await _context.SaveChangesAsync();
        return true;
    }

    public virtual async Task<bool> ExistsAsync(int id)
    {
        var entity = await _dbSet.FindAsync(id);
        return entity != null;
    }

    public virtual async Task<IEnumerable<T>> FindAsync(
        Expression<Func<T, bool>> predicate)
    {
        return await _dbSet.Where(predicate).ToListAsync();
    }

    public virtual async Task<T?> FirstOrDefaultAsync(
        Expression<Func<T, bool>> predicate)
    {
        return await _dbSet.FirstOrDefaultAsync(predicate);
    }
}
```

### Extending Generic Repository

```csharp
// File: Repositories/BookRepository.cs

// Extend generic repository with entity-specific methods
public interface IBookRepository : IRepository<Book>
{
    Task<IEnumerable<Book>> GetByAuthorAsync(int authorId);
    Task<IEnumerable<Book>> GetRecentBooksAsync(int count);
}

public class BookRepository : Repository<Book>, IBookRepository
{
    public BookRepository(AppDbContext context) : base(context)
    {
    }

    // Override to include related data
    public override async Task<Book?> GetByIdAsync(int id)
    {
        return await _dbSet
            .Include(b => b.Author)
            .FirstOrDefaultAsync(b => b.Id == id);
    }

    // Entity-specific method
    public async Task<IEnumerable<Book>> GetByAuthorAsync(int authorId)
    {
        return await _dbSet
            .Where(b => b.AuthorId == authorId)
            .Include(b => b.Author)
            .ToListAsync();
    }

    public async Task<IEnumerable<Book>> GetRecentBooksAsync(int count)
    {
        return await _dbSet
            .OrderByDescending(b => b.PublishedDate)
            .Take(count)
            .ToListAsync();
    }
}
```

---

## Unit of Work Pattern

### What is Unit of Work?

Unit of Work maintains a list of objects affected by a business transaction and coordinates the writing out of changes.

```csharp
// File: Repositories/IUnitOfWork.cs

public interface IUnitOfWork : IDisposable
{
    IBookRepository Books { get; }
    IAuthorRepository Authors { get; }
    IOrderRepository Orders { get; }
    
    Task<int> SaveChangesAsync();
    Task BeginTransactionAsync();
    Task CommitTransactionAsync();
    Task RollbackTransactionAsync();
}
```

```csharp
// File: Repositories/UnitOfWork.cs

public class UnitOfWork : IUnitOfWork
{
    private readonly AppDbContext _context;
    private IDbContextTransaction _transaction;
    
    private IBookRepository _books;
    private IAuthorRepository _authors;
    private IOrderRepository _orders;

    public UnitOfWork(AppDbContext context)
    {
        _context = context;
    }

    // Lazy loading of repositories
    public IBookRepository Books => 
        _books ??= new BookRepository(_context);
    
    public IAuthorRepository Authors => 
        _authors ??= new AuthorRepository(_context);
    
    public IOrderRepository Orders => 
        _orders ??= new OrderRepository(_context);

    public async Task<int> SaveChangesAsync()
    {
        return await _context.SaveChangesAsync();
    }

    public async Task BeginTransactionAsync()
    {
        _transaction = await _context.Database.BeginTransactionAsync();
    }

    public async Task CommitTransactionAsync()
    {
        await _transaction.CommitAsync();
    }

    public async Task RollbackTransactionAsync()
    {
        await _transaction.RollbackAsync();
    }

    public void Dispose()
    {
        _transaction?.Dispose();
        _context.Dispose();
    }
}
```

### Using Unit of Work

```csharp
// File: Services/OrderService.cs

public class OrderService : IOrderService
{
    private readonly IUnitOfWork _unitOfWork;

    public OrderService(IUnitOfWork unitOfWork)
    {
        _unitOfWork = unitOfWork;
    }

    public async Task<Order> PlaceOrderAsync(OrderDto orderDto)
    {
        await _unitOfWork.BeginTransactionAsync();
        
        try
        {
            // Create order
            var order = new Order { /* ... */ };
            await _unitOfWork.Orders.AddAsync(order);
            
            // Update book stock
            foreach (var item in orderDto.Items)
            {
                var book = await _unitOfWork.Books.GetByIdAsync(item.BookId);
                book.Stock -= item.Quantity;
            }
            
            // Save all changes
            await _unitOfWork.SaveChangesAsync();
            await _unitOfWork.CommitTransactionAsync();
            
            return order;
        }
        catch
        {
            await _unitOfWork.RollbackTransactionAsync();
            throw;
        }
    }
}
```

---

## Dependency Injection Registration

```csharp
// File: Program.cs

var builder = WebApplication.CreateBuilder(args);

// Register DbContext
builder.Services.AddDbContextPool<AppDbContext>(options =>
    options.UseSqlServer(
        builder.Configuration.GetConnectionString("DefaultConnection")));

// ═══════════════════════════════════════════════════════════════
// REGISTER REPOSITORIES
// ═══════════════════════════════════════════════════════════════

// Option 1: Individual registration
builder.Services.AddScoped<IBookRepository, SqlBookRepository>();
builder.Services.AddScoped<IAuthorRepository, SqlAuthorRepository>();

// Option 2: Generic repository
builder.Services.AddScoped(typeof(IRepository<>), typeof(Repository<>));

// Option 3: Unit of Work
builder.Services.AddScoped<IUnitOfWork, UnitOfWork>();

var app = builder.Build();
```

---

## Code Examples

### Controller Using Repository

```csharp
// File: Controllers/BookController.cs

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

    [HttpGet]
    public async Task<ActionResult<IEnumerable<Book>>> GetAll()
    {
        var books = await _bookRepository.GetAllAsync();
        return Ok(books);
    }

    [HttpGet("{id}")]
    public async Task<ActionResult<Book>> GetById(int id)
    {
        var book = await _bookRepository.GetByIdAsync(id);
        
        if (book == null)
            return NotFound();
            
        return Ok(book);
    }

    [HttpPost]
    public async Task<ActionResult<Book>> Create(Book book)
    {
        var createdBook = await _bookRepository.AddAsync(book);
        
        return CreatedAtAction(
            nameof(GetById), 
            new { id = createdBook.BookId }, 
            createdBook);
    }

    [HttpPut("{id}")]
    public async Task<IActionResult> Update(int id, Book book)
    {
        var updatedBook = await _bookRepository.UpdateAsync(id, book);
        
        if (updatedBook == null)
            return NotFound();
            
        return Ok(updatedBook);
    }

    [HttpDelete("{id}")]
    public async Task<IActionResult> Delete(int id)
    {
        var deletedBook = await _bookRepository.DeleteAsync(id);
        
        if (deletedBook == null)
            return NotFound();
            
        return NoContent();
    }
}
```

---

## Best Practices

### 1. Interface Segregation

```csharp
// ✅ GOOD: Specific interface
public interface IBookRepository
{
    Task<Book> GetByIdAsync(int id);
    Task<IEnumerable<Book>> GetByAuthorAsync(int authorId);
}

// ❌ BAD: God interface with everything
public interface IMegaRepository
{
    Task<Book> GetBookByIdAsync(int id);
    Task<Author> GetAuthorByIdAsync(int id);
    Task<Order> GetOrderByIdAsync(int id);
    // ... too many responsibilities
}
```

### 2. Keep Repositories Focused

```csharp
// ✅ GOOD: Repository returns entities
public async Task<Book?> GetByIdAsync(int id)
{
    return await _context.Books.FindAsync(id);
}

// ❌ BAD: Repository returns HTTP responses
public async Task<IActionResult> GetByIdAsync(int id)
{
    var book = await _context.Books.FindAsync(id);
    return new OkObjectResult(book);  // Not repository's job!
}
```

### 3. Use Async Methods

```csharp
// ✅ GOOD: Async for I/O operations
public async Task<Book> AddAsync(Book book)
{
    await _context.Books.AddAsync(book);
    await _context.SaveChangesAsync();
    return book;
}

// ❌ BAD: Blocking synchronous calls
public Book Add(Book book)
{
    _context.Books.Add(book);
    _context.SaveChanges();  // Blocks thread
    return book;
}
```

---

## Interview Questions

### Q1: What is the Repository Pattern?
**Answer:** Repository Pattern is a design pattern that creates an abstraction layer between the data access logic and the business logic. It encapsulates database operations behind an interface, making the code more testable and maintainable.

### Q2: What are the benefits of using Repository Pattern?
**Answer:**
- Separation of concerns
- Easier unit testing (can mock repositories)
- Centralized data access logic
- Ability to switch data sources without changing business logic
- Cleaner controllers

### Q3: What is the difference between Repository and Unit of Work?
**Answer:**
- **Repository**: Encapsulates data access for a single entity type
- **Unit of Work**: Coordinates multiple repositories and manages transactions across them

### Q4: Should you use Generic Repository?
**Answer:** It depends:
- **Yes**: For simple CRUD operations across many entities
- **No**: When entities have very different query requirements
- **Hybrid approach**: Generic base with entity-specific extensions

### Q5: Where should SaveChanges be called?
**Answer:** 
- In individual repository methods for simple operations
- In Unit of Work for complex operations involving multiple entities
- Never in the controller

### Q6: How do you test code using Repository Pattern?
**Answer:** Create mock implementations of repository interfaces using frameworks like Moq:
```csharp
var mockRepo = new Mock<IBookRepository>();
mockRepo.Setup(r => r.GetByIdAsync(1))
    .ReturnsAsync(new Book { Id = 1, Title = "Test" });
```

---

## Quick Reference

### Basic Repository Interface

```csharp
public interface IRepository<T>
{
    Task<T> AddAsync(T entity);
    Task<T?> GetByIdAsync(int id);
    Task<IEnumerable<T>> GetAllAsync();
    Task<T?> UpdateAsync(T entity);
    Task<bool> DeleteAsync(int id);
}
```

### Registration

```csharp
builder.Services.AddScoped<IBookRepository, SqlBookRepository>();
builder.Services.AddScoped<IUnitOfWork, UnitOfWork>();
```

### Controller Usage

```csharp
public class BookController : ControllerBase
{
    private readonly IBookRepository _repository;
    
    public BookController(IBookRepository repository)
    {
        _repository = repository;
    }
}
```
