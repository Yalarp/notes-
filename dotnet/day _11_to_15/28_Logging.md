# Logging in ASP.NET Core

## Table of Contents
1. [Introduction](#introduction)
2. [Built-in Logging](#built-in-logging)
3. [Log Levels](#log-levels)
4. [Logging Configuration](#logging-configuration)
5. [Structured Logging](#structured-logging)
6. [Logging Providers](#logging-providers)
7. [Using ILogger](#using-ilogger)
8. [Best Practices](#best-practices)
9. [Interview Questions](#interview-questions)

---

## Introduction

### Real-World Analogy
Logging is like a **flight recorder (black box)**:

```
Without Logging:
    Plane crashes → No idea what happened
    ❌ Can't investigate issues

With Logging:
    Every action recorded → Review recordings after incident
    ✅ Full visibility into what happened
    ✅ Debug issues, track performance
```

---

## Built-in Logging

ASP.NET Core has built-in logging infrastructure through `ILogger<T>`.

### Basic Usage

```csharp
// File: Controllers/ProductController.cs

[ApiController]
[Route("api/[controller]")]
public class ProductController : ControllerBase
{
    // Inject ILogger<T> through constructor
    private readonly ILogger<ProductController> _logger;

    public ProductController(ILogger<ProductController> logger)
    {
        _logger = logger;
    }

    [HttpGet("{id}")]
    public IActionResult GetProduct(int id)
    {
        // Log at different levels
        _logger.LogInformation("Getting product with ID: {Id}", id);
        
        var product = _repository.GetById(id);
        
        if (product == null)
        {
            _logger.LogWarning("Product {Id} not found", id);
            return NotFound();
        }
        
        return Ok(product);
    }
}
```

---

## Log Levels

### Hierarchy (Lowest to Highest)

```
┌─────────────────────────────────────────────────────────────────┐
│                    LOG LEVELS (Severity)                         │
└─────────────────────────────────────────────────────────────────┘

Level          | Value | Description
─────────────────────────────────────────────────────────────────
Trace          |  0    | Most detailed, for debugging
Debug          |  1    | Development debugging information
Information    |  2    | General operational events
Warning        |  3    | Abnormal or unexpected events
Error          |  4    | Errors and exceptions
Critical       |  5    | System failures requiring attention
None           |  6    | Disable all logging
```

### Using Log Levels

```csharp
public class OrderService
{
    private readonly ILogger<OrderService> _logger;

    public async Task<Order> ProcessOrderAsync(Order order)
    {
        // Trace: Very detailed diagnostic information
        _logger.LogTrace("Entering ProcessOrderAsync with order {OrderId}", order.Id);
        
        // Debug: Development-time debugging
        _logger.LogDebug("Order items count: {Count}", order.Items.Count);
        
        // Information: General operational messages
        _logger.LogInformation("Processing order {OrderId} for user {UserId}", 
            order.Id, order.UserId);
        
        // Warning: Something unexpected but not an error
        if (order.Total > 10000)
        {
            _logger.LogWarning("High value order {OrderId}: ${Total}", 
                order.Id, order.Total);
        }
        
        try
        {
            await _paymentService.ProcessPaymentAsync(order);
        }
        catch (PaymentException ex)
        {
            // Error: An error occurred but app can continue
            _logger.LogError(ex, "Payment failed for order {OrderId}", order.Id);
            throw;
        }
        catch (Exception ex)
        {
            // Critical: System-wide failure
            _logger.LogCritical(ex, "Critical failure processing order {OrderId}", 
                order.Id);
            throw;
        }
        
        return order;
    }
}
```

---

## Logging Configuration

### appsettings.json

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information",
      "Microsoft.AspNetCore": "Warning",
      "Microsoft.EntityFrameworkCore": "Warning",
      "MyApp.Controllers": "Debug",
      "MyApp.Services": "Information"
    },
    "Console": {
      "LogLevel": {
        "Default": "Information"
      },
      "FormatterName": "simple",
      "FormatterOptions": {
        "SingleLine": true,
        "TimestampFormat": "HH:mm:ss "
      }
    },
    "Debug": {
      "LogLevel": {
        "Default": "Debug"
      }
    }
  }
}
```

### Environment-Specific Configuration

```json
// appsettings.Development.json
{
  "Logging": {
    "LogLevel": {
      "Default": "Debug",
      "Microsoft.EntityFrameworkCore.Database.Command": "Information"
    }
  }
}

// appsettings.Production.json
{
  "Logging": {
    "LogLevel": {
      "Default": "Warning",
      "Microsoft": "Warning"
    }
  }
}
```

---

## Structured Logging

### Use Message Templates

```csharp
// ✅ GOOD: Structured logging with message templates
_logger.LogInformation("Processing order {OrderId} for user {UserId}", 
    orderId, userId);

// Properties can be queried/filtered:
// OrderId = 12345
// UserId = 67890

// ❌ BAD: String concatenation - loses structure
_logger.LogInformation("Processing order " + orderId + " for user " + userId);

// ❌ BAD: String interpolation - loses structure
_logger.LogInformation($"Processing order {orderId} for user {userId}");
```

### Destructuring Objects

```csharp
// @ prefix destructures object
_logger.LogInformation("Order created: {@Order}", order);

// Output includes all properties:
// Order = { Id = 1, Total = 100.00, Items = [...] }

// Without @ uses ToString()
_logger.LogInformation("Order created: {Order}", order);
// Order = "MyApp.Models.Order"
```

---

## Logging Providers

### Built-in Providers

```csharp
// File: Program.cs

var builder = WebApplication.CreateBuilder(args);

// Clear default providers and add specific ones
builder.Logging.ClearProviders();
builder.Logging.AddConsole();        // Output to console
builder.Logging.AddDebug();          // Output to Debug window
builder.Logging.AddEventSourceLogger(); // ETW (Windows)
builder.Logging.AddEventLog();       // Windows Event Log

var app = builder.Build();
```

### Adding Serilog (Popular Third-Party)

```bash
dotnet add package Serilog.AspNetCore
dotnet add package Serilog.Sinks.Console
dotnet add package Serilog.Sinks.File
```

```csharp
// Program.cs
using Serilog;

Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Information()
    .WriteTo.Console()
    .WriteTo.File("logs/log-.txt", rollingInterval: RollingInterval.Day)
    .CreateLogger();

var builder = WebApplication.CreateBuilder(args);
builder.Host.UseSerilog();  // Replace built-in logging

var app = builder.Build();

app.UseSerilogRequestLogging();  // Log HTTP requests

app.Run();
```

---

## Using ILogger

### In Controllers

```csharp
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    private readonly ILogger<UsersController> _logger;
    private readonly IUserService _userService;

    public UsersController(
        ILogger<UsersController> logger,
        IUserService userService)
    {
        _logger = logger;
        _userService = userService;
    }

    [HttpGet("{id}")]
    public async Task<IActionResult> GetUser(int id)
    {
        _logger.LogInformation("Retrieving user {UserId}", id);
        
        var user = await _userService.GetByIdAsync(id);
        
        if (user == null)
        {
            _logger.LogWarning("User {UserId} not found", id);
            return NotFound();
        }
        
        return Ok(user);
    }
}
```

### In Services

```csharp
public class EmailService : IEmailService
{
    private readonly ILogger<EmailService> _logger;

    public EmailService(ILogger<EmailService> logger)
    {
        _logger = logger;
    }

    public async Task SendEmailAsync(string to, string subject, string body)
    {
        _logger.LogInformation("Sending email to {Recipient}", to);
        
        try
        {
            // Send email logic
            _logger.LogDebug("Email sent successfully to {Recipient}", to);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to send email to {Recipient}", to);
            throw;
        }
    }
}
```

### Using Log Scopes

```csharp
public async Task<Order> ProcessOrderAsync(int orderId)
{
    // Create a scope - all logs within include OrderId
    using (_logger.BeginScope("OrderId: {OrderId}", orderId))
    {
        _logger.LogInformation("Starting order processing");
        // Log output: OrderId: 123 - Starting order processing
        
        await ValidateOrderAsync(orderId);
        // Log output: OrderId: 123 - Validation completed
        
        await ProcessPaymentAsync(orderId);
        // Log output: OrderId: 123 - Payment processed
    }
}
```

---

## Best Practices

### 1. Use Appropriate Log Levels

```csharp
// ✅ GOOD: Meaningful levels
_logger.LogInformation("User {UserId} logged in", userId);
_logger.LogWarning("Failed login attempt for {Username}", username);
_logger.LogError(ex, "Payment processing failed");

// ❌ BAD: Everything as Information
_logger.LogInformation("Exception: " + ex.Message);  // Should be Error
```

### 2. Include Context

```csharp
// ✅ GOOD: Include relevant context
_logger.LogError(ex, 
    "Order {OrderId} failed for user {UserId} with items {ItemCount}", 
    orderId, userId, items.Count);

// ❌ BAD: Generic message
_logger.LogError(ex, "An error occurred");
```

### 3. Don't Log Sensitive Data

```csharp
// ❌ BAD: Logging sensitive information
_logger.LogInformation("User logged in with password {Password}", password);
_logger.LogDebug("Credit card: {CardNumber}", cardNumber);

// ✅ GOOD: Mask or omit sensitive data
_logger.LogInformation("User {UserId} authenticated", user.Id);
_logger.LogDebug("Payment with card ending in {LastFour}", cardNumber[^4..]);
```

### 4. Use Conditional Logging for Expensive Operations

```csharp
// ✅ GOOD: Check if level is enabled before expensive operations
if (_logger.IsEnabled(LogLevel.Debug))
{
    var expensiveData = ComputeExpensiveDebugInfo();
    _logger.LogDebug("Debug info: {@Data}", expensiveData);
}

// ❌ BAD: Always compute even if not logging
_logger.LogDebug("Debug info: {@Data}", ComputeExpensiveDebugInfo());
```

---

## Interview Questions

### Q1: What are the different log levels in ASP.NET Core?
**Answer:** Trace (0), Debug (1), Information (2), Warning (3), Error (4), Critical (5), None (6). Each level includes all higher severity levels.

### Q2: What is structured logging?
**Answer:** Logging with message templates that preserve data structure, allowing logs to be queried by property values. Example: `_logger.LogInformation("User {UserId} logged in", userId)`.

### Q3: What is the difference between LogError and LogCritical?
**Answer:**
- **LogError**: Application errors that can be handled, app continues running
- **LogCritical**: System failures requiring immediate attention, may cause app to stop

### Q4: What are log scopes?
**Answer:** Scopes add contextual information to all log messages within a block. Created with `_logger.BeginScope()`, useful for tracing requests through multiple operations.

### Q5: How do you configure logging per namespace?
**Answer:** In appsettings.json, specify namespace patterns under LogLevel:
```json
"LogLevel": {
  "Default": "Information",
  "MyApp.Services": "Debug",
  "Microsoft.EntityFrameworkCore": "Warning"
}
```

---

## Quick Reference

### Basic Usage

```csharp
// Inject
private readonly ILogger<MyClass> _logger;

// Use
_logger.LogInformation("Message with {Parameter}", value);
_logger.LogError(exception, "Error message with {Context}", context);
```

### Log Levels

```csharp
_logger.LogTrace("Most detailed");
_logger.LogDebug("Development info");
_logger.LogInformation("Operational events");
_logger.LogWarning("Unexpected events");
_logger.LogError(ex, "Errors");
_logger.LogCritical(ex, "System failures");
```

### Configuration

```json
"Logging": {
  "LogLevel": {
    "Default": "Information",
    "Microsoft": "Warning"
  }
}
```
