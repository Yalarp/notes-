# Middleware in ASP.NET Core

## Table of Contents
1. [Introduction](#introduction)
2. [What is Middleware?](#what-is-middleware)
3. [Built-in Middleware](#built-in-middleware)
4. [Custom Middleware](#custom-middleware)
5. [Middleware Patterns](#middleware-patterns)
6. [Exception Handling Middleware](#exception-handling-middleware)
7. [StatusCodePages Middleware](#statuscodepages-middleware)
8. [Code Examples](#code-examples)
9. [Best Practices](#best-practices)
10. [Interview Questions](#interview-questions)

---

## Introduction

### Real-World Analogy
Middleware is like a **series of filters in a water purification system**:

```
Raw Water (HTTP Request)
        │
        ▼
┌─────────────────┐
│  Sediment Filter│  ← Exception Handler (catches big problems)
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Carbon Filter  │  ← HTTPS Redirection (transforms protocol)
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  UV Sterilizer  │  ← Authentication (validates identity)
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Mineral Add    │  ← Authorization (adds permissions)
└────────┬────────┘
         │
         ▼
   Pure Water (Endpoint)
```

Each filter can:
- **Transform** the water (modify request/response)
- **Block** contaminated water (short-circuit)
- **Pass** to the next filter (call next())

---

## What is Middleware?

Middleware is software that's assembled into an application pipeline to handle requests and responses.

### Core Concepts

| Concept | Description |
|---------|-------------|
| **Request Delegate** | `Func<HttpContext, Task>` - function that handles a request |
| **next()** | Invokes the next middleware in the pipeline |
| **Short-circuit** | Stopping the pipeline without calling next() |
| **Terminal** | Middleware that doesn't call next() |

### How Middleware Works

```csharp
// Each middleware receives a RequestDelegate representing the next middleware
public class MyMiddleware
{
    private readonly RequestDelegate _next;
    
    public MyMiddleware(RequestDelegate next)
    {
        _next = next;
    }
    
    public async Task InvokeAsync(HttpContext context)
    {
        // BEFORE: Code runs when request comes in
        Console.WriteLine("Before handling request");
        
        await _next(context);  // Call next middleware
        
        // AFTER: Code runs when response goes out
        Console.WriteLine("After handling request");
    }
}
```

### Middleware Execution Flow

```
               REQUEST
                  │
    ┌─────────────▼─────────────┐
    │      Middleware 1         │
    │  ┌─────────────────────┐  │
    │  │    Before code      │  │
    │  └──────────┬──────────┘  │
    │             │ await next()│
    │  ┌──────────▼──────────┐  │
    │  │     Middleware 2    │  │
    │  │  ┌───────────────┐  │  │
    │  │  │ Before code   │  │  │
    │  │  └───────┬───────┘  │  │
    │  │          │          │  │
    │  │  ┌───────▼───────┐  │  │
    │  │  │   Endpoint    │  │  │
    │  │  └───────┬───────┘  │  │
    │  │          │          │  │
    │  │  ┌───────▼───────┐  │  │
    │  │  │ After code    │  │  │
    │  │  └───────────────┘  │  │
    │  └──────────┬──────────┘  │
    │             │             │
    │  ┌──────────▼──────────┐  │
    │  │    After code       │  │
    │  └─────────────────────┘  │
    └─────────────┬─────────────┘
                  │
               RESPONSE
```

---

## Built-in Middleware

### Essential Built-in Middleware

| Middleware | Method | Purpose |
|------------|--------|---------|
| **Exception Handler** | `UseExceptionHandler()` | Catches unhandled exceptions |
| **Developer Exception** | `UseDeveloperExceptionPage()` | Detailed dev error pages |
| **HSTS** | `UseHsts()` | HTTP Strict Transport Security |
| **HTTPS Redirection** | `UseHttpsRedirection()` | Redirects HTTP to HTTPS |
| **Static Files** | `UseStaticFiles()` | Serves static files |
| **Routing** | `UseRouting()` | URL route matching |
| **Authentication** | `UseAuthentication()` | Validates identity |
| **Authorization** | `UseAuthorization()` | Checks permissions |
| **Session** | `UseSession()` | Session state |
| **CORS** | `UseCors()` | Cross-origin requests |
| **Response Caching** | `UseResponseCaching()` | Cache responses |
| **Response Compression** | `UseResponseCompression()` | Compress responses |

### Middleware Registration Examples

```csharp
// File: Program.cs

var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// ═══════════════════════════════════════════════════════════════
// EXCEPTION HANDLING (Must be first)
// ═══════════════════════════════════════════════════════════════

// Line 1: Development - detailed error pages
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}
else
{
    // Line 2: Production - generic error page
    app.UseExceptionHandler("/Error");
    
    // Line 3: HTTP Strict Transport Security
    app.UseHsts();
}

// ═══════════════════════════════════════════════════════════════
// SECURITY MIDDLEWARE
// ═══════════════════════════════════════════════════════════════

// Line 4: Redirect HTTP to HTTPS
app.UseHttpsRedirection();

// ═══════════════════════════════════════════════════════════════
// STATIC FILES
// ═══════════════════════════════════════════════════════════════

// Line 5: Serve files from wwwroot
app.UseStaticFiles();

// Line 6: Also serve files from a custom folder
app.UseStaticFiles(new StaticFileOptions
{
    FileProvider = new PhysicalFileProvider(
        Path.Combine(builder.Environment.ContentRootPath, "MyFiles")),
    RequestPath = "/files"
});

// ═══════════════════════════════════════════════════════════════
// ROUTING
// ═══════════════════════════════════════════════════════════════

// Line 7: Enable routing
app.UseRouting();

// ═══════════════════════════════════════════════════════════════
// AUTHENTICATION & AUTHORIZATION
// ═══════════════════════════════════════════════════════════════

// Line 8: Authentication - "Who are you?"
app.UseAuthentication();

// Line 9: Authorization - "Are you allowed?"
app.UseAuthorization();

// ═══════════════════════════════════════════════════════════════
// ENDPOINTS
// ═══════════════════════════════════════════════════════════════

app.MapControllers();

app.Run();
```

---

## Custom Middleware

### Method 1: Inline Middleware (Use/Run)

```csharp
// File: Program.cs

var app = builder.Build();

// ─────────────────────────────────────────────────────────────
// Use() - Middleware that can call next
// ─────────────────────────────────────────────────────────────

app.Use(async (context, next) =>
{
    // Line 1: Log request start
    Console.WriteLine($"Request: {context.Request.Path}");
    
    // Line 2: Start timer
    var stopwatch = System.Diagnostics.Stopwatch.StartNew();
    
    // Line 3: Call next middleware
    await next();
    
    // Line 4: Log response time
    stopwatch.Stop();
    Console.WriteLine($"Response time: {stopwatch.ElapsedMilliseconds}ms");
});

// ─────────────────────────────────────────────────────────────
// Run() - Terminal middleware (doesn't call next)
// ─────────────────────────────────────────────────────────────

app.Run(async context =>
{
    // This is the end of the pipeline
    await context.Response.WriteAsync("Hello from terminal middleware!");
});
```

### Method 2: Class-Based Middleware (Recommended)

```csharp
// File: Middleware/RequestTimingMiddleware.cs

namespace MyApp.Middleware
{
    // Line 1: Custom middleware class
    public class RequestTimingMiddleware
    {
        // Line 2: Store reference to next middleware
        private readonly RequestDelegate _next;
        
        // Line 3: Store logger (injected)
        private readonly ILogger<RequestTimingMiddleware> _logger;

        // Line 4: Constructor - receives next middleware and services
        public RequestTimingMiddleware(
            RequestDelegate next,
            ILogger<RequestTimingMiddleware> logger)
        {
            _next = next;
            _logger = logger;
        }

        // Line 5: InvokeAsync - called for every request
        // MUST be named "Invoke" or "InvokeAsync"
        public async Task InvokeAsync(HttpContext context)
        {
            // ═══════════════════════════════════════════════════
            // BEFORE: Request processing
            // ═══════════════════════════════════════════════════
            
            // Line 6: Log request start
            var requestPath = context.Request.Path;
            var requestMethod = context.Request.Method;
            _logger.LogInformation(
                "Request started: {Method} {Path}", 
                requestMethod, requestPath);
            
            // Line 7: Start timer
            var stopwatch = System.Diagnostics.Stopwatch.StartNew();
            
            // ═══════════════════════════════════════════════════
            // CALL NEXT MIDDLEWARE
            // ═══════════════════════════════════════════════════
            
            // Line 8: Pass to next middleware
            await _next(context);
            
            // ═══════════════════════════════════════════════════
            // AFTER: Response processing
            // ═══════════════════════════════════════════════════
            
            // Line 9: Stop timer and log
            stopwatch.Stop();
            _logger.LogInformation(
                "Request completed: {Method} {Path} - {StatusCode} in {Ms}ms",
                requestMethod, 
                requestPath, 
                context.Response.StatusCode,
                stopwatch.ElapsedMilliseconds);
        }
    }
}
```

```csharp
// File: Program.cs - Using custom middleware

var app = builder.Build();

// Register custom middleware
app.UseMiddleware<RequestTimingMiddleware>();

app.MapControllers();
app.Run();
```

### Method 3: Extension Method Pattern (Best Practice)

```csharp
// File: Middleware/MiddlewareExtensions.cs

namespace MyApp.Middleware
{
    // Line 1: Static class for extension methods
    public static class MiddlewareExtensions
    {
        // Line 2: Extension method for easy registration
        public static IApplicationBuilder UseRequestTiming(
            this IApplicationBuilder builder)
        {
            return builder.UseMiddleware<RequestTimingMiddleware>();
        }
        
        // Line 3: Extension with options
        public static IApplicationBuilder UseCustomLogging(
            this IApplicationBuilder builder,
            CustomLoggingOptions options)
        {
            return builder.UseMiddleware<CustomLoggingMiddleware>(options);
        }
    }
}
```

```csharp
// File: Program.cs - Clean registration

var app = builder.Build();

// Now you can use: (much cleaner!)
app.UseRequestTiming();

// Instead of:
// app.UseMiddleware<RequestTimingMiddleware>();
```

---

## Middleware Patterns

### Pattern 1: Run, Use, Map

```csharp
// RUN - Terminal middleware (ends pipeline)
app.Run(async context =>
{
    await context.Response.WriteAsync("Terminal - No next()");
});

// USE - Passes to next middleware
app.Use(async (context, next) =>
{
    // Do something
    await next();  // Continue pipeline
    // Do something after
});

// MAP - Branch pipeline based on path
app.Map("/api", apiApp =>
{
    apiApp.Run(async context =>
    {
        await context.Response.WriteAsync("API Branch");
    });
});
```

### Pattern 2: Conditional Middleware (UseWhen/MapWhen)

```csharp
// UseWhen - Conditional without branching
// Continue to main pipeline after condition branch
app.UseWhen(
    context => context.Request.Path.StartsWithSegments("/api"),
    apiApp =>
    {
        // Only runs for /api/* requests
        apiApp.UseMiddleware<ApiLoggingMiddleware>();
    }
);
// Main pipeline continues for all requests

// MapWhen - Conditional with branching
// Creates separate branch, doesn't return to main pipeline
app.MapWhen(
    context => context.Request.Query.ContainsKey("debug"),
    debugApp =>
    {
        debugApp.Run(async context =>
        {
            await context.Response.WriteAsync("Debug mode enabled");
        });
    }
);
```

### Pattern 3: Injecting Services in Middleware

```csharp
// Method 1: Constructor injection (singleton-scoped services only)
public class MyMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger _logger;  // Singleton service
    
    public MyMiddleware(RequestDelegate next, ILogger<MyMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }
}

// Method 2: Invoke parameter injection (scoped/transient services)
public class MyMiddleware
{
    private readonly RequestDelegate _next;
    
    public MyMiddleware(RequestDelegate next)
    {
        _next = next;
    }
    
    // Scoped services injected here
    public async Task InvokeAsync(HttpContext context, IDbContext dbContext)
    {
        // dbContext is scoped to this request
        await _next(context);
    }
}
```

---

## Exception Handling Middleware

### Complete Exception Handling Middleware

```csharp
// File: Middleware/ExceptionHandlingMiddleware.cs

using Newtonsoft.Json;
using System.Net;

namespace MVCEmpDept.Models
{
    // Line 1: Exception handling middleware class
    public class ExceptionHandlingMiddleware
    {
        // Line 2: Store next middleware delegate
        private readonly RequestDelegate _next;
        
        // Line 3: Store logger for error logging
        private readonly ILogger<ExceptionHandlingMiddleware> _logger;

        // Line 4: Constructor - receives dependencies via DI
        public ExceptionHandlingMiddleware(
            RequestDelegate next, 
            ILogger<ExceptionHandlingMiddleware> logger)
        {
            _next = next;
            _logger = logger;
        }

        // Line 5: Main method called for each request
        public async Task InvokeAsync(HttpContext httpContext)
        {
            try
            {
                // Line 6: Try to execute the rest of the pipeline
                await _next(httpContext);
            }
            catch (Exception ex)
            {
                // Line 7: If any exception occurs, handle it
                await HandleExceptionAsync(httpContext, ex);
            }
        }

        // Line 8: Handle exception and return proper response
        private async Task HandleExceptionAsync(
            HttpContext context, 
            Exception exception)
        {
            // Line 9: Log the exception
            _logger.LogError(exception, "An unhandled exception occurred");
            
            // Line 10: Set status code to 500 (Internal Server Error)
            var code = HttpStatusCode.InternalServerError;

            // Line 11: Create error response object
            var result = new BaseResponseDTO()
            {
                ErrorCode = (int)HttpStatusCode.InternalServerError,
                ErrorMessage = exception.Message,
                Succeed = false,
            };
            
            // Line 12: Serialize to JSON
            var jsonResult = JsonConvert.SerializeObject(result);
            
            // Line 13: Set response content type
            context.Response.ContentType = "application/json";
            
            // Line 14: Set response status code
            context.Response.StatusCode = (int)code;
            
            // Line 15: Write response body
            await context.Response.WriteAsync(jsonResult);
        }
    }
}
```

### Usage in Program.cs

```csharp
// File: Program.cs

var app = builder.Build();

// Exception middleware should be early in the pipeline
// but after UseExceptionHandler for production
if (app.Environment.IsDevelopment())
{
    // Development: Let our middleware handle exceptions
    app.UseMiddleware<ExceptionHandlingMiddleware>();
}
else
{
    // Production: Use built-in handler first, then ours
    app.UseExceptionHandler("/Error");
    app.UseMiddleware<ExceptionHandlingMiddleware>();
}

app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();

app.MapControllers();
app.Run();
```

### BaseResponseDTO Class

```csharp
// File: Models/BaseResponseDTO.cs

namespace MVCEmpDept.Models
{
    // Line 1: DTO for API responses
    public class BaseResponseDTO
    {
        // Line 2: Error code (HTTP status code)
        public int ErrorCode { get; set; }
        
        // Line 3: Error message description
        public string ErrorMessage { get; set; } = string.Empty;
        
        // Line 4: Whether operation succeeded
        public bool Succeed { get; set; }
        
        // Line 5: Data payload (for successful responses)
        public object? Data { get; set; }
    }
}
```

---

## StatusCodePages Middleware

### Different StatusCodePages Options

```csharp
// File: Program.cs

var app = builder.Build();

// ═══════════════════════════════════════════════════════════════
// OPTION 1: Simple text response
// ═══════════════════════════════════════════════════════════════

app.UseStatusCodePages();
// Returns: "Status Code: 404; Not Found"

// ═══════════════════════════════════════════════════════════════
// OPTION 2: Custom text format
// ═══════════════════════════════════════════════════════════════

app.UseStatusCodePages("text/plain", "Error - Status code: {0}");
// Returns: "Error - Status code: 404"

// ═══════════════════════════════════════════════════════════════
// OPTION 3: Redirect to error page
// ═══════════════════════════════════════════════════════════════

// Redirects browser (changes URL)
app.UseStatusCodePagesWithRedirects("/Error/{0}");
// Redirects to: /Error/404

// ═══════════════════════════════════════════════════════════════
// OPTION 4: Re-execute pipeline (URL stays same)
// ═══════════════════════════════════════════════════════════════

// Re-executes request internally (URL doesn't change)
app.UseStatusCodePagesWithReExecute("/Error/{0}");
// Internally executes: /Error/404 but URL shows original

// ═══════════════════════════════════════════════════════════════
// OPTION 5: Custom handler lambda
// ═══════════════════════════════════════════════════════════════

app.UseStatusCodePages(async context =>
{
    var statusCode = context.HttpContext.Response.StatusCode;
    
    context.HttpContext.Response.ContentType = "text/html";
    
    await context.HttpContext.Response.WriteAsync(
        $"<h1>Error {statusCode}</h1>" +
        $"<p>Sorry, something went wrong.</p>");
});
```

### Error Controller for StatusCodePages

```csharp
// File: Controllers/ErrorController.cs

public class ErrorController : Controller
{
    // Line 1: Handle different status codes
    [Route("Error/{statusCode}")]
    public IActionResult HttpStatusCodeHandler(int statusCode)
    {
        // Line 2: Build view model with error details
        var errorViewModel = new ErrorViewModel
        {
            StatusCode = statusCode,
            Message = statusCode switch
            {
                404 => "Page not found",
                500 => "Internal server error",
                401 => "Unauthorized",
                403 => "Forbidden",
                _ => "An error occurred"
            }
        };
        
        // Line 3: Return appropriate view
        return statusCode switch
        {
            404 => View("NotFound", errorViewModel),
            _ => View("Error", errorViewModel)
        };
    }
}
```

---

## Code Examples

### Logging Middleware

```csharp
// File: Middleware/LoggingMiddleware.cs

public class LoggingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<LoggingMiddleware> _logger;

    public LoggingMiddleware(RequestDelegate next, ILogger<LoggingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        // Line 1: Log request details
        _logger.LogInformation(
            "REQUEST: {Method} {Path} {QueryString}",
            context.Request.Method,
            context.Request.Path,
            context.Request.QueryString);
        
        // Line 2: Capture original response body stream
        var originalBodyStream = context.Response.Body;
        
        using var responseBody = new MemoryStream();
        context.Response.Body = responseBody;
        
        // Line 3: Call next middleware
        await _next(context);
        
        // Line 4: Log response details
        _logger.LogInformation(
            "RESPONSE: {StatusCode}",
            context.Response.StatusCode);
        
        // Line 5: Copy response to original stream
        await responseBody.CopyToAsync(originalBodyStream);
    }
}
```

### Authentication Check Middleware

```csharp
// File: Middleware/ApiKeyMiddleware.cs

public class ApiKeyMiddleware
{
    private readonly RequestDelegate _next;
    private const string API_KEY_HEADER = "X-Api-Key";

    public ApiKeyMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(HttpContext context, IConfiguration config)
    {
        // Line 1: Check if API key header exists
        if (!context.Request.Headers.TryGetValue(API_KEY_HEADER, out var extractedApiKey))
        {
            // Line 2: No API key - return 401
            context.Response.StatusCode = 401;
            await context.Response.WriteAsync("API Key missing");
            return;  // Short-circuit
        }

        // Line 3: Get valid API key from configuration
        var validApiKey = config["ApiKey"];
        
        // Line 4: Validate API key
        if (!validApiKey.Equals(extractedApiKey))
        {
            // Line 5: Invalid key - return 401
            context.Response.StatusCode = 401;
            await context.Response.WriteAsync("Invalid API Key");
            return;  // Short-circuit
        }

        // Line 6: Valid key - continue pipeline
        await _next(context);
    }
}
```

---

## Best Practices

### 1. Keep Middleware Single-Purpose

```csharp
// ✅ GOOD: Single responsibility
public class TimingMiddleware { /* Only timing */ }
public class LoggingMiddleware { /* Only logging */ }

// ❌ BAD: Multiple responsibilities
public class DoEverythingMiddleware { /* Timing + Logging + Auth + ... */ }
```

### 2. Use Extension Methods for Clean Registration

```csharp
// ✅ GOOD: Clean and readable
app.UseExceptionHandling();
app.UseRequestTiming();
app.UseApiAuthentication();

// ❌ BAD: Verbose registration
app.UseMiddleware<ExceptionHandlingMiddleware>();
app.UseMiddleware<RequestTimingMiddleware>();
app.UseMiddleware<ApiAuthenticationMiddleware>();
```

### 3. Avoid Blocking Calls

```csharp
// ✅ GOOD: Async all the way
public async Task InvokeAsync(HttpContext context)
{
    await _next(context);
}

// ❌ BAD: Blocking call
public Task InvokeAsync(HttpContext context)
{
    _next(context).Wait();  // NEVER do this!
    return Task.CompletedTask;
}
```

### 4. Handle Exceptions Properly

```csharp
// ✅ GOOD: Exception handling middleware at start
app.UseExceptionHandler();
app.UseMiddleware<CustomExceptionHandler>();

// ❌ BAD: Other middleware before exception handler
app.UseStaticFiles();
app.UseExceptionHandler();  // Won't catch errors from UseStaticFiles!
```

---

## Interview Questions

### Q1: What is middleware in ASP.NET Core?
**Answer:** Middleware is software assembled into an application pipeline to handle requests and responses. Each middleware component can:
- Choose to pass the request to the next component
- Perform work before and after the next component
- Short-circuit the pipeline by not calling next()

### Q2: What is the difference between Use() and Run()?
**Answer:**
- `Use()`: Adds middleware that can call `next()` to pass to the next middleware
- `Run()`: Adds terminal middleware that doesn't call `next()` - ends the pipeline

### Q3: How do you create custom middleware?
**Answer:** Create a class with:
1. Constructor that accepts `RequestDelegate` (and optionally other services)
2. An `InvokeAsync(HttpContext context)` or `Invoke(HttpContext context)` method
3. Register using `app.UseMiddleware<T>()` or an extension method

### Q4: What is short-circuiting in middleware?
**Answer:** Short-circuiting occurs when middleware doesn't call `next()` and returns a response directly. This stops the request from traveling further down the pipeline. Common scenarios include authentication failures or cached responses.

### Q5: Why is middleware order important?
**Answer:** Order matters because:
- Each middleware runs in registration order
- Some middleware depends on others (Authorization needs Routing)
- Exception handlers must be first to catch all exceptions
- Static files should be before authentication for performance

---

## Quick Reference

### Middleware Method Signatures

```csharp
// Inline middleware
app.Use(async (HttpContext context, RequestDelegate next) => { });

// Terminal middleware
app.Run(async (HttpContext context) => { });

// Class-based middleware
public class MyMiddleware
{
    public MyMiddleware(RequestDelegate next) { }
    public async Task InvokeAsync(HttpContext context) { }
}
```

### Registration Patterns

```csharp
// Built-in middleware
app.UseStaticFiles();
app.UseRouting();
app.UseAuthentication();
app.UseAuthorization();

// Custom middleware
app.UseMiddleware<MyMiddleware>();
app.UseMiddleware<MyMiddleware>(options);  // With options

// Extension method (recommended)
app.UseMyMiddleware();
```

### Correct Order Cheat Sheet

```
1. Exception/Error handling
2. HSTS
3. HTTPS Redirection
4. Static Files
5. Routing
6. CORS
7. Authentication
8. Authorization
9. Custom middleware
10. Endpoints
```
