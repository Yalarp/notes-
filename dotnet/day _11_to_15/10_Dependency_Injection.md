# Dependency Injection in ASP.NET Core

## Table of Contents
1. [Introduction](#introduction)
2. [What is Dependency Injection?](#what-is-dependency-injection)
3. [Benefits of DI](#benefits-of-di)
4. [Service Lifetimes](#service-lifetimes)
5. [Registering Services](#registering-services)
6. [Constructor Injection](#constructor-injection)
7. [Multiple Implementations](#multiple-implementations)
8. [Service Provider](#service-provider)
9. [Code Examples](#code-examples)
10. [Best Practices](#best-practices)
11. [Interview Questions](#interview-questions)

---

## Introduction

### Real-World Analogy
Dependency Injection is like **ordering food delivery**:

```
Without DI (Tight Coupling):
    Restaurant → Own kitchen (must cook everything)
    ❌ Can't change suppliers easily
    ❌ Hard to test
    ❌ Inflexible

With DI (Loose Coupling):
    Restaurant → Ingredient Suppliers (injected)
    ✅ Easy to swap suppliers
    ✅ Easy to test with mock suppliers
    ✅ Flexible and maintainable
```

---

## What is Dependency Injection?

Dependency Injection (DI) is a design pattern where dependencies are provided to a class from outside rather than created inside the class.

### Without DI (Tight Coupling)

```csharp
// ❌ BAD: Class creates its own dependencies
public class EmployeeController : Controller
{
    private readonly EmployeeService _service;
    
    public EmployeeController()
    {
        // Hard-coded dependency - tight coupling!
        _service = new EmployeeService();
    }
    
    public IActionResult Index()
    {
        var employees = _service.GetAll();
        return View(employees);
    }
}

// Problems:
// 1. Hard to test (can't mock EmployeeService)
// 2. Hard to change implementation
// 3. Violates Dependency Inversion Principle
```

### With DI (Loose Coupling)

```csharp
// ✅ GOOD: Dependencies injected via constructor
public class EmployeeController : Controller
{
    private readonly IEmployeeService _service;
    
    // Dependency injected by DI container
    public EmployeeController(IEmployeeService service)
    {
        _service = service;
    }
    
    public IActionResult Index()
    {
        var employees = _service.GetAll();
        return View(employees);
    }
}

// Benefits:
// 1. Easy to test (can inject mock service)
// 2. Easy to swap implementations
// 3. Follows SOLID principles
```

### DI Flow Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                  DEPENDENCY INJECTION FLOW                       │
└─────────────────────────────────────────────────────────────────┘

1. Register Services (Program.cs)
   ────────────────────────────────────────────────────────────
   builder.Services.AddScoped<IEmployeeService, SqlEmployeeService>();
                                    │
                                    ▼
                         DI Container stores mapping
                         IEmployeeService → SqlEmployeeService

2. Request Comes In
   ────────────────────────────────────────────────────────────
   GET /Employee/Index
                │
                ▼
   Controller needed: EmployeeController
                │
                ▼
   Constructor needs: IEmployeeService

3. DI Container Resolves Dependency
   ────────────────────────────────────────────────────────────
   DI Container:
   - Looks up IEmployeeService
   - Finds mapping to SqlEmployeeService
   - Creates instance: new SqlEmployeeService(dependencies...)
   - Injects into EmployeeController constructor
                │
                ▼
   EmployeeController instance created with dependencies

4. Action Executes
   ────────────────────────────────────────────────────────────
   Controller uses injected service
   Returns view with data
```

---

## Benefits of DI

### Key Advantages

| Benefit | Description |
|---------|-------------|
| **Loose Coupling** | Classes depend on abstractions, not concrete types |
| **Testability** | Easy to inject mock dependencies for unit testing |
| **Maintainability** | Changes to implementations don't affect consumers |
| **Flexibility** | Easy to swap implementations without code changes |
| **Lifecycle Management** | Framework manages object creation and disposal |

### SOLID Principles

```
DI supports SOLID principles:

S - Single Responsibility
    Each class has one reason to change

O - Open/Closed
    Open for extension, closed for modification

L - Liskov Substitution
    Derived classes can replace base classes

D - Dependency Inversion ✅ (DI's main focus)
    Depend on abstractions, not concretions

I - Interface Segregation
    Many specific interfaces > one general interface
```

---

## Service Lifetimes

ASP.NET Core provides three service lifetimes:

### 1. Transient

**Created each time they're requested**

```csharp
// Line 1: Register as Transient
builder.Services.AddTransient<IEmailService, EmailService>();

// Behavior:
// Request 1: new EmailService() created
// Request 2: new EmailService() created (different instance)
// Request 3: new EmailService() created (different instance)
```

```
┌─────────────────────────────────────────────────────────────┐
│                    TRANSIENT LIFETIME                        │
└─────────────────────────────────────────────────────────────┘

HTTP Request arrives
    │
    ├──→ Controller needs IEmailService → New EmailService()
    ├──→ View needs IEmailService → New EmailService()
    └──→ Middleware needs IEmailService → New EmailService()
    
    Each request gets a NEW instance
```

**Use for**: Lightweight, stateless services

### 2. Scoped

**Created once per HTTP request**

```csharp
// Line 1: Register as Scoped
builder.Services.AddScoped<IEmployeeService, SqlEmployeeService>();

// Behavior:
// Request 1: One instance for entire request
// Request 2: New instance for this request
// Request 3: New instance for this request
```

```
┌─────────────────────────────────────────────────────────────┐
│                     SCOPED LIFETIME                          │
└─────────────────────────────────────────────────────────────┘

HTTP Request 1
    │
    ├──→ Controller needs IEmployeeService → Instance A
    ├──→ View needs IEmployeeService → Instance A (same)
    └──→ Repository needs IEmployeeService → Instance A (same)
    
    ONE instance shared within the request

HTTP Request 2
    │
    └──→ New instance created for this request
```

**Use for**: Database contexts, request-specific services

### 3. Singleton

**Created once for the application lifetime**

```csharp
// Line 1: Register as Singleton
builder.Services.AddSingleton<ICacheService, MemoryCacheService>();

// Behavior:
// App starts: One instance created
// Request 1: Uses same instance
// Request 2: Uses same instance
// Request 3: Uses same instance
// App lifetime: Same instance always
```

```
┌─────────────────────────────────────────────────────────────┐
│                   SINGLETON LIFETIME                         │
└─────────────────────────────────────────────────────────────┘

Application Startup
    │
    └──→ ONE instance created
            │
            ├──→ Request 1 uses it
            ├──→ Request 2 uses it
            ├──→ Request 3 uses it
            └──→ All requests share SAME instance
```

**Use for**: Configuration, logging, caching services

### Lifetime Comparison Table

| Lifetime | Created | Shared | Disposed | Use Case |
|----------|---------|--------|----------|----------|
| **Transient** | Every request | No | After use | Lightweight, stateless |
| **Scoped** | Per HTTP request | Within request | End of request | DbContext, repositories |
| **Singleton** | Once at startup | App-wide | App shutdown | Cache, config, logging |

---

## Registering Services

### Basic Registration

```csharp
// File: Program.cs

var builder = WebApplication.CreateBuilder(args);

// ═══════════════════════════════════════════════════════════════
// ADDING SERVICES TO DI CONTAINER
// ═══════════════════════════════════════════════════════════════

// Line 1: Register MVC services
builder.Services.AddControllersWithViews();

// Line 2: Register DbContext (Scoped by default)
builder.Services.AddDbContextPool<AppdbContextRepository>(options =>
    options.UseSqlServer(
        builder.Configuration.GetConnectionString("EmployeeDBConnection")));

// Line 3: Register application services

// Scoped - new instance per request
builder.Services.AddScoped<IEmployeeService, SqlEmployeeService>();
builder.Services.AddScoped<IDepartmentService, SqlDepartmentService>();

// Transient - new instance every time
builder.Services.AddTransient<IEmailService, EmailService>();
builder.Services.AddTransient<ILogger, FileLogger>();

// Singleton - one instance for app lifetime
builder.Services.AddSingleton<IConfiguration>(builder.Configuration);
builder.Services.AddSingleton<ICacheService, MemoryCacheService>();

var app = builder.Build();

// Configure middleware pipeline
app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Employee}/{action=Index}/{id?}");

app.Run();
```

### Registration Patterns

```csharp
// ═══════════════════════════════════════════════════════════════
// PATTERN 1: Interface to Implementation
// ═══════════════════════════════════════════════════════════════
builder.Services.AddScoped<IEmployeeService, SqlEmployeeService>();
// IEmployeeService requested → SqlEmployeeService provided

// ═══════════════════════════════════════════════════════════════
// PATTERN 2: Concrete Type Only
// ═══════════════════════════════════════════════════════════════
builder.Services.AddScoped<EmployeeService>();
// EmployeeService requested → EmployeeService provided

// ═══════════════════════════════════════════════════════════════
// PATTERN 3: Factory Method
// ═══════════════════════════════════════════════════════════════
builder.Services.AddScoped<IEmployeeService>(serviceProvider =>
{
    var config = serviceProvider.GetRequiredService<IConfiguration>();
    var connectionString = config.GetConnectionString("EmployeeDB");
    return new SqlEmployeeService(connectionString);
});

// ═══════════════════════════════════════════════════════════════
// PATTERN 4: Instance Registration (Singleton only)
// ═══════════════════════════════════════════════════════════════
var cacheService = new MemoryCacheService();
builder.Services.AddSingleton<ICacheService>(cacheService);

// ═══════════════════════════════════════════════════════════════
// PATTERN 5: Generic Registration
// ═══════════════════════════════════════════════════════════════
builder.Services.AddScoped(typeof(IRepository<>), typeof(Repository<>));
// IRepository<Employee> → Repository<Employee>
// IRepository<Department> → Repository<Department>
```

---

## Constructor Injection

### Basic Constructor Injection

```csharp
// File: Controllers/EmployeeController.cs

public class EmployeeController : Controller
{
    // Line 1: Declare dependencies as readonly fields
    private readonly IEmployeeService _employeeService;
    private readonly ILogger<EmployeeController> _logger;
    
    // Line 2: Constructor with dependencies
    // DI container automatically injects these
    public EmployeeController(
        IEmployeeService employeeService,
        ILogger<EmployeeController> logger)
    {
        // Line 3: Assign to fields
        _employeeService = employeeService;
        _logger = logger;
    }
    
    // Line 4: Use injected dependencies
    public IActionResult Index()
    {
        _logger.LogInformation("Fetching all employees");
        
        var employees = _employeeService.GetAllEmployee();
        
        return View(employees);
    }
}
```

### Multiple Dependencies

```csharp
// File: Services/SqlEmployeeService.cs

public class SqlEmployeeService : IEmployeeService
{
    // Line 1: Service can have its own dependencies
    private readonly AppdbContextRepository _context;
    private readonly ILogger<SqlEmployeeService> _logger;
    private readonly IConfiguration _configuration;
    
    // Line 2: All dependencies injected by DI container
    public SqlEmployeeService(
        AppdbContextRepository context,
        ILogger<SqlEmployeeService> logger,
        IConfiguration configuration)
    {
        _context = context;
        _logger = logger;
        _configuration = configuration;
    }
    
    public List<Employee> GetAllEmployee()
    {
        _logger.LogInformation("Getting all employees from database");
        return _context.Employees.Include(e => e.Department).ToList();
    }
    
    public void Add(Employee employee)
    {
        _context.Employees.Add(employee);
        _context.SaveChanges();
        _logger.LogInformation($"Employee {employee.Name} added");
    }
}
```

### Dependency Chain

```
┌─────────────────────────────────────────────────────────────────┐
│                    DEPENDENCY RESOLUTION CHAIN                   │
└─────────────────────────────────────────────────────────────────┘

Request: /Employee/Index

1. Need: EmployeeController
    │
    ├─→ Requires: IEmployeeService
    │       │
    │       ├─→ Requires: AppdbContextRepository
    │       │       │
    │       │       └─→ Requires: DbContextOptions
    │       │
    │       ├─→ Requires: ILogger<SqlEmployeeService>
    │       │
    │       └─→ Requires: IConfiguration
    │
    └─→ Requires: ILogger<EmployeeController>

DI Container resolves ALL dependencies automatically!
```

---

## Multiple Implementations

### Registering Multiple Implementations

```csharp
// File: Program.cs

// Line 1: Register multiple implementations
builder.Services.AddScoped<INotificationService, EmailNotificationService>();
builder.Services.AddScoped<INotificationService, SmsNotificationService>();
builder.Services.AddScoped<INotificationService, PushNotificationService>();

// When resolving IEnumerable<INotificationService>, gets ALL implementations
```

### Using Multiple Implementations

```csharp
// File: Services/NotificationManager.cs

public class NotificationManager
{
    private readonly IEnumerable<INotificationService> _notificationServices;
    
    // Line 1: Inject collection of all implementations
    public NotificationManager(IEnumerable<INotificationService> notificationServices)
    {
        _notificationServices = notificationServices;
    }
    
    // Line 2: Use all implementations
    public async Task NotifyAll(string message)
    {
        foreach (var service in _notificationServices)
        {
            await service.SendAsync(message);
        }
        // Sends via Email, SMS, and Push notifications
    }
}
```

### Named Services Pattern

```csharp
// File: Program.cs

// Register with factory selecting implementation
builder.Services.AddScoped<IEmployeeService>(serviceProvider =>
{
    var config = serviceProvider.GetRequiredService<IConfiguration>();
    var dataSource = config["DataSource"]; // "Sql" or "InMemory"
    
    if (dataSource == "Sql")
        return new SqlEmployeeService(serviceProvider);
    else
        return new InMemoryEmployeeService();
});
```

---

## Service Provider

### Manual Service Resolution

```csharp
// File: Startup or Program.cs

var app = builder.Build();

// Line 1: Get service provider
var serviceProvider = app.Services;

// Line 2: Resolve service manually
var employeeService = serviceProvider.GetRequiredService<IEmployeeService>();

// Line 3: Resolve optional service
var cacheService = serviceProvider.GetService<ICacheService>();
if (cacheService != null)
{
    // Use cache service
}
```

### IServiceProvider in Controllers

```csharp
// File: Controllers/EmployeeController.cs

public class EmployeeController : Controller
{
    private readonly IServiceProvider _serviceProvider;
    
    // Line 1: Inject IServiceProvider (not recommended for regular use)
    public EmployeeController(IServiceProvider serviceProvider)
    {
        _serviceProvider = serviceProvider;
    }
    
    public IActionResult Index()
    {
        // Line 2: Resolve service at runtime (Service Locator anti-pattern)
        var service = _serviceProvider.GetRequiredService<IEmployeeService>();
        
        var employees = service.GetAllEmployee();
        return View(employees);
    }
}

// ⚠️ NOTE: Prefer constructor injection over service locator pattern
```

---

## Code Examples

### Complete DI Setup

```csharp
// File: Program.cs

using Microsoft.EntityFrameworkCore;
using MVCEmpDept.Models;
using MVCEmpDept.Service;

var builder = WebApplication.CreateBuilder(args);

// ═══════════════════════════════════════════════════════════════
// REGISTER SERVICES
// ═══════════════════════════════════════════════════════════════

// Line 1: Add framework services
builder.Services.AddControllersWithViews();

// Line 2: Register DbContext with connection pooling
builder.Services.AddDbContextPool<AppdbContextRepository>(options =>
    options.UseSqlServer(
        builder.Configuration.GetConnectionString("EmployeeDBConnection")));

// Line 3: Register application services

// Scoped services (per request)
builder.Services.AddScoped<IEmployeeService, SqlEmployeeService>();
builder.Services.AddScoped<IDepartmentService, SqlDepartmentService>();

// Transient services (per dependency)
builder.Services.AddTransient<IEmailService, EmailService>();
builder.Services.AddTransient<IPdfGenerator, PdfGenerator>();

// Singleton services (app lifetime)
builder.Services.AddSingleton<ICacheService, MemoryCacheService>();

// Line 4: Add logging
builder.Services.AddLogging(config =>
{
    config.AddConsole();
    config.AddDebug();
});

var app = builder.Build();

// ═══════════════════════════════════════════════════════════════
// CONFIGURE MIDDLEWARE PIPELINE
// ═══════════════════════════════════════════════════════════════

if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Home/Error");
    app.UseHsts();
}
else
{
    app.UseStatusCodePagesWithRedirects("/Error/{0}");
}

app.UseMiddleware<ExceptionHandlingMiddleware>();
app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Employee}/{action=Index}/{id?}");

app.Run();
```

### Service Interface and Implementation

```csharp
// File: Service/IEmployeeService.cs

public interface IEmployeeService
{
    List<Employee> GetAllEmployee();
    Employee GetEmployee(int id);
    Employee Add(Employee employee);
    Employee Update(Employee employee);
    void Delete(int id);
    List<Department> GetAllDepartment();
}
```

```csharp
// File: Service/SqlEmployeeService.cs

public class SqlEmployeeService : IEmployeeService
{
    private readonly AppdbContextRepository _context;
    private readonly ILogger<SqlEmployeeService> _logger;
    
    // Constructor injection
    public SqlEmployeeService(
        AppdbContextRepository context,
        ILogger<SqlEmployeeService> logger)
    {
        _context = context;
        _logger = logger;
    }
    
    public List<Employee> GetAllEmployee()
    {
        _logger.LogInformation("Fetching all employees");
        return _context.Employees
            .Include(e => e.Department)
            .ToList();
    }
    
    public Employee GetEmployee(int id)
    {
        return _context.Employees
            .Include(e => e.Department)
            .FirstOrDefault(e => e.Id == id);
    }
    
    public Employee Add(Employee employee)
    {
        _context.Employees.Add(employee);
        _context.SaveChanges();
        _logger.LogInformation($"Employee {employee.Name} added with ID {employee.Id}");
        return employee;
    }
    
    public Employee Update(Employee employee)
    {
        _context.Employees.Update(employee);
        _context.SaveChanges();
        _logger.LogInformation($"Employee {employee.Id} updated");
        return employee;
    }
    
    public void Delete(int id)
    {
        var employee = _context.Employees.Find(id);
        if (employee != null)
        {
            _context.Employees.Remove(employee);
            _context.SaveChanges();
            _logger.LogInformation($"Employee {id} deleted");
        }
    }
    
    public List<Department> GetAllDepartment()
    {
        return _context.Departments.ToList();
    }
}
```

---

## Best Practices

### 1. Depend on Abstractions

```csharp
// ✅ GOOD: Depend on interface
private readonly IEmployeeService _service;

public EmployeeController(IEmployeeService service)
{
    _service = service;
}

// ❌ BAD: Depend on concrete type
private readonly SqlEmployeeService _service;

public EmployeeController(SqlEmployeeService service)
{
    _service = service;
}
```

### 2. Use Appropriate Lifetimes

```csharp
// ✅ GOOD: DbContext is Scoped
builder.Services.AddScoped<AppdbContext>();

// ❌ BAD: DbContext as Singleton (NOT thread-safe!)
builder.Services.AddSingleton<AppdbContext>();

// ✅ GOOD: Stateless services can be Singleton
builder.Services.AddSingleton<ICacheService, MemoryCacheService>();

// ❌ BAD: Stateful service as Singleton
builder.Services.AddSingleton<IShoppingCart, ShoppingCart>();
```

### 3. Avoid Service Locator Pattern

```csharp
// ✅ GOOD: Constructor injection
public EmployeeController(IEmployeeService service)
{
    _service = service;
}

// ❌ BAD: Service locator anti-pattern
public EmployeeController(IServiceProvider serviceProvider)
{
    _service = serviceProvider.GetService<IEmployeeService>();
}
```

---

## Interview Questions

### Q1: What is Dependency Injection?
**Answer:** Dependency Injection is a design pattern where dependencies are provided to a class from outside rather than created inside the class. It promotes loose coupling and makes code more testable and maintainable.

### Q2: What are the three service lifetimes in ASP.NET Core?
**Answer:**
1. **Transient**: Created each time requested
2. **Scoped**: Created once per HTTP request
3. **Singleton**: Created once for application lifetime

### Q3: When should you use Scoped lifetime?
**Answer:** Use Scoped for services that should be created once per HTTP request but shared within that request. Common example: DbContext in Entity Framework Core.

### Q4: What's the difference between AddScoped and AddTransient?
**Answer:**
- `AddScoped`: One instance per HTTP request (shared within request)
- `AddTransient`: New instance every time the service is requested

### Q5: Can a Singleton service depend on a Scoped service?
**Answer:** No, this creates a "captive dependency" problem. Since Singleton lives for the app lifetime, it would hold onto a Scoped service longer than intended, causing issues.

### Q6: How do you register multiple implementations of the same interface?
**Answer:** Register all implementations, then inject `IEnumerable<IInterface>` to get all of them:
```csharp
builder.Services.AddScoped<INotification, EmailNotification>();
builder.Services.AddScoped<INotification, SmsNotification>();

// Constructor: IEnumerable<INotification> notifications
```

---

## Quick Reference

### Registration Methods

```csharp
// Transient
builder.Services.AddTransient<IService, ServiceImpl>();

// Scoped
builder.Services.AddScoped<IService, ServiceImpl>();

// Singleton
builder.Services.AddSingleton<IService, ServiceImpl>();

// With factory
builder.Services.AddScoped<IService>(sp => new ServiceImpl(config));
```

### Constructor Injection

```csharp
public class MyController : Controller
{
    private readonly IService _service;
    
    public MyController(IService service)
    {
        _service = service;
    }
}
```

### Lifetime Guidelines

```
Transient  → Lightweight, stateless services
Scoped     → DbContext, per-request state
Singleton  → Configuration, caching, logging
```
