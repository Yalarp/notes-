# 💉 Dependency Injection Fundamentals

## 📚 Table of Contents
- [Overview](#-overview)
- [Tight vs Loose Coupling](#-tight-vs-loose-coupling)
- [DI Patterns](#-di-patterns)
- [.NET Core DI Container](#-net-core-di-container)
- [Service Lifetimes](#-service-lifetimes)
- [Interview Questions](#-interview-questions)

---

## 🎯 Overview

**Dependency Injection (DI)** is a design pattern where dependencies are provided externally rather than created internally.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    WITHOUT DI (Tight Coupling)                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   class OrderService                                                │
│   {                                                                 │
│       private EmailService _email = new EmailService(); ◄── Creates│
│       private Logger _logger = new Logger();             dependency │
│   }                                                                 │
│                                                                     │
│   Problems: Hard to test, hard to change implementations           │
│                                                                     │
├─────────────────────────────────────────────────────────────────────┤
│                    WITH DI (Loose Coupling)                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   class OrderService                                                │
│   {                                                                 │
│       private IEmailService _email;                                 │
│       private ILogger _logger;                                      │
│                                                                     │
│       public OrderService(IEmailService email, ILogger logger)     │
│       {                              ▲                              │
│           _email = email;            │ Injected from outside       │
│           _logger = logger;          │                              │
│       }                                                             │
│   }                                                                 │
│                                                                     │
│   Benefits: Testable, swappable, maintainable                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔷 Tight vs Loose Coupling

### Tight Coupling (Bad)

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// TIGHT COUPLING - HARD TO TEST AND MAINTAIN
// ═══════════════════════════════════════════════════════════════════════════

// ❌ BAD: Directly creates dependencies
public class OrderService
{
    private SqlDatabase _database;  // Concrete class, not interface
    private EmailService _emailer;  // Concrete class
    
    public OrderService()
    {
        _database = new SqlDatabase();  // Hardcoded dependency
        _emailer = new EmailService();  // Cannot change without modifying class
    }
    
    public void PlaceOrder(Order order)
    {
        _database.Save(order);
        _emailer.Send($"Order {order.Id} placed");
    }
}

// Problems:
// 1. Cannot unit test without real database
// 2. Cannot swap SQL for MongoDB without changing this class
// 3. Cannot mock email service for testing
```

### Loose Coupling (Good)

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// LOOSE COUPLING - TESTABLE AND MAINTAINABLE
// ═══════════════════════════════════════════════════════════════════════════

// Define interfaces (contracts)
public interface IDatabase
{
    void Save(Order order);
}

public interface IEmailService
{
    void Send(string message);
}

// ✅ GOOD: Depends on interfaces, receives dependencies via constructor
public class OrderService
{
    private readonly IDatabase _database;
    private readonly IEmailService _emailer;
    
    // Constructor Injection
    public OrderService(IDatabase database, IEmailService emailer)
    // Line 1: Dependencies injected via constructor
    {
        _database = database;
        _emailer = emailer;
    }
    
    public void PlaceOrder(Order order)
    {
        _database.Save(order);
        _emailer.Send($"Order {order.Id} placed");
    }
}

// Implementations
public class SqlDatabase : IDatabase
{
    public void Save(Order order) { /* Save to SQL */ }
}

public class MongoDatabase : IDatabase
{
    public void Save(Order order) { /* Save to MongoDB */ }
}

// Usage - can swap implementations easily
IDatabase db = new SqlDatabase();  // Or MongoDatabase
IEmailService email = new EmailService();
var service = new OrderService(db, email);

// Testing - can use mock
var mockDb = new Mock<IDatabase>();
var mockEmail = new Mock<IEmailService>();
var testService = new OrderService(mockDb.Object, mockEmail.Object);
```

---

## 🔶 DI Patterns

### 1. Constructor Injection (Most Common)

```csharp
public class ProductService
{
    private readonly IRepository _repo;
    private readonly ILogger _logger;
    
    // Dependencies provided via constructor
    public ProductService(IRepository repo, ILogger logger)
    {
        _repo = repo ?? throw new ArgumentNullException(nameof(repo));
        _logger = logger ?? throw new ArgumentNullException(nameof(logger));
    }
}
```

### 2. Property Injection

```csharp
public class ReportGenerator
{
    // Optional dependency via property
    public ILogger Logger { get; set; }
    
    public void Generate()
    {
        Logger?.Log("Generating report...");
        // Generate report
    }
}

// Usage
var generator = new ReportGenerator { Logger = new FileLogger() };
```

### 3. Method Injection

```csharp
public class DataProcessor
{
    // Dependency provided per method call
    public void Process(IFormatter formatter)
    {
        var data = GetData();
        var formatted = formatter.Format(data);
        Save(formatted);
    }
}
```

---

## 🔵 .NET Core DI Container

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// BUILT-IN DI CONTAINER IN .NET CORE
// ═══════════════════════════════════════════════════════════════════════════

using Microsoft.Extensions.DependencyInjection;

// Program.cs in .NET 6+
var builder = WebApplication.CreateBuilder(args);

// ─────────────────────────────────────────────────────────────────────
// REGISTER SERVICES
// ─────────────────────────────────────────────────────────────────────

// Register interface -> implementation
builder.Services.AddTransient<IEmailService, EmailService>();
// Line 1: When IEmailService requested, create EmailService

builder.Services.AddScoped<IOrderRepository, SqlOrderRepository>();
// Line 2: One instance per request/scope

builder.Services.AddSingleton<IConfiguration, ConfigurationService>();
// Line 3: Single instance for entire application

// ─────────────────────────────────────────────────────────────────────
// USING REGISTERED SERVICES
// ─────────────────────────────────────────────────────────────────────

public class OrderController : ControllerBase
{
    private readonly IOrderRepository _repo;
    private readonly IEmailService _email;
    
    // DI container automatically injects registered services
    public OrderController(IOrderRepository repo, IEmailService email)
    {
        _repo = repo;
        _email = email;
    }
    
    [HttpPost]
    public IActionResult Create(Order order)
    {
        _repo.Add(order);
        _email.Send($"Order created: {order.Id}");
        return Ok();
    }
}

// ─────────────────────────────────────────────────────────────────────
// ADVANCED REGISTRATION
// ─────────────────────────────────────────────────────────────────────

// Factory registration
builder.Services.AddTransient<IService>(provider =>
{
    var config = provider.GetRequiredService<IConfiguration>();
    return new Service(config.GetValue<string>("ApiKey"));
});

// Multiple implementations
builder.Services.AddTransient<INotifier, EmailNotifier>();
builder.Services.AddTransient<INotifier, SmsNotifier>();
// Inject IEnumerable<INotifier> to get all

// Try add (only if not already registered)
builder.Services.TryAddTransient<IService, DefaultService>();
```

---

## 🟢 Service Lifetimes

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// SERVICE LIFETIMES
// ═══════════════════════════════════════════════════════════════════════════

// Three lifetimes in .NET DI:

// 1. TRANSIENT - New instance every time
services.AddTransient<ITransientService, TransientService>();
// Use for: Lightweight, stateless services

// 2. SCOPED - One instance per scope/request
services.AddScoped<IScopedService, ScopedService>();
// Use for: DbContext, per-request caching

// 3. SINGLETON - One instance for app lifetime
services.AddSingleton<ISingletonService, SingletonService>();
// Use for: Configuration, caching, logging
```

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SERVICE LIFETIME COMPARISON                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   TRANSIENT                                                         │
│   Request 1: [Instance A] [Instance B] [Instance C]                │
│   Request 2: [Instance D] [Instance E] [Instance F]                │
│   (New instance every injection)                                   │
│                                                                     │
│   SCOPED                                                            │
│   Request 1: [Instance A] [Instance A] [Instance A]                │
│   Request 2: [Instance B] [Instance B] [Instance B]                │
│   (Same instance within request, new per request)                  │
│                                                                     │
│   SINGLETON                                                         │
│   Request 1: [Instance A] [Instance A] [Instance A]                │
│   Request 2: [Instance A] [Instance A] [Instance A]                │
│   (Same instance everywhere, forever)                              │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Lifetime Rules

```csharp
// ❌ DANGER: Scoped service in Singleton
// Singleton lives forever, but Scoped should die with request
services.AddSingleton<MySingleton>();  // Lives forever
services.AddScoped<MyScoped>();        // Dies after request

public class MySingleton
{
    // ❌ BAD: Capturing scoped in singleton
    public MySingleton(MyScoped scoped) { }
}

// ✅ CORRECT: Only inject same or longer lifetime
// Transient can inject: Transient, Scoped, Singleton
// Scoped can inject: Scoped, Singleton
// Singleton can inject: Singleton only
```

---

## 🎤 Interview Questions

**Q1: What is Dependency Injection?**
> Design pattern where dependencies are provided externally rather than created internally. Promotes loose coupling and testability.

**Q2: What are the benefits of DI?**
> Loose coupling, testability (can mock), flexibility (swap implementations), maintainability.

**Q3: Constructor vs Property injection?**
> Constructor for required dependencies (fails fast). Property for optional dependencies.

**Q4: Explain service lifetimes?**
> Transient: new instance each time. Scoped: one per request. Singleton: one for app lifetime.

**Q5: Can Singleton inject Scoped service?**
> No! Singleton outlives Scoped, causing "captive dependency" bug. Only inject same or longer lifetime.

---

## 📝 Summary

| Concept | Description |
|---------|-------------|
| **DI** | Provide dependencies externally |
| **Interface** | Contract for loose coupling |
| **Constructor Injection** | Pass deps via constructor |
| **Transient** | New instance every time |
| **Scoped** | One per request |
| **Singleton** | One for app lifetime |
