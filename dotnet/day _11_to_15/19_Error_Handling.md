# Error Handling in ASP.NET Core MVC

## Table of Contents
1. [Introduction](#introduction)
2. [Types of Errors](#types-of-errors)
3. [Developer Exception Page](#developer-exception-page)
4. [Exception Handler Middleware](#exception-handler-middleware)
5. [Status Code Pages](#status-code-pages)
6. [Custom Exception Middleware](#custom-exception-middleware)
7. [Error Views](#error-views)
8. [Logging Errors](#logging-errors)
9. [Code Examples](#code-examples)
10. [Best Practices](#best-practices)
11. [Interview Questions](#interview-questions)

---

## Introduction

### Real-World Analogy
Error handling is like **hospital emergency protocols**:

```
Minor Issue (400 Bad Request)
    → Reception handles it (Status Code Page)
    
Serious Issue (500 Internal Error)
    → Emergency Room (Exception Handler)
    
Development Hospital (Dev Environment)
    → Show all details (Developer Exception Page)
    
Public Hospital (Production)
    → Show user-friendly message (Custom Error Page)
```

---

## Types of Errors

### Error Categories

| Error Type | Status Code | Description | Example |
|------------|-------------|-------------|---------|
| **Client Errors** | 400-499 | User's fault | 404 Not Found, 401 Unauthorized |
| **Server Errors** | 500-599 | Server's fault | 500 Internal Server Error |
| **Exceptions** | Any | Unhandled code errors | NullReferenceException, DbException |

### Common HTTP Status Codes

```
┌─────────────────────────────────────────────────────────────────┐
│                   HTTP STATUS CODES                              │
├────────┬────────────────────┬──────────────────────────────────┤
│ Code   │ Name               │ Meaning                          │
├────────┼────────────────────┼──────────────────────────────────┤
│ 200    │ OK                 │ Request succeeded                │
│ 201    │ Created            │ Resource created                 │
│ 204    │ No Content         │ Success, no content to return    │
│ ────── │ ────────────────── │ ──────────────────────────────── │
│ 400    │ Bad Request        │ Invalid request format           │
│ 401    │ Unauthorized       │ Authentication required          │
│ 403    │ Forbidden          │ No permission                    │
│ 404    │ Not Found          │ Resource doesn't exist           │
│ ────── │ ────────────────── │ ──────────────────────────────── │
│ 500    │ Internal Error     │ Server-side error                │
│ 503    │ Service Unavailable│ Server temporarily down          │
└────────┴────────────────────┴──────────────────────────────────┘
```

---

## Developer Exception Page

### What is Developer Exception Page?

Shows **detailed error information** during development (stack trace, request details, variables).

```csharp
// File: Program.cs

var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllersWithViews();

var app = builder.Build();

// ═══════════════════════════════════════════════════════════════
// DEVELOPER EXCEPTION PAGE (Development Only)
// ═══════════════════════════════════════════════════════════════

if (app.Environment.IsDevelopment())
{
    // Line 1: Show detailed error page in development
    app.UseDeveloperExceptionPage();
    // Shows:
    // - Full stack trace
    // - Request headers
    // - Cookies
    // - Query strings
    // - Route data
}
else
{
    // Production error handling
    app.UseExceptionHandler("/Home/Error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();
```

### Developer Exception Page Features

```
When exception occurs in Development:
────────────────────────────────────────────────────────────────
┌──────────────────────────────────────────────────────────────┐
│  DEVELOPER EXCEPTION PAGE                                     │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Exception: NullReferenceException                           │
│  Message: Object reference not set to an instance...         │
│                                                              │
│  Stack Trace:                                                │
│    at EmployeeController.Details() line 45                   │
│    at Microsoft.AspNetCore.Mvc.Infrastructure...             │
│                                                              │
│  Query   │ Cookies  │ Headers  │ Routing  │ Environment     │
│                                                              │
│  Request Path: /Employee/Details/5                           │
│  Request Method: GET                                         │
│                                                              │
└──────────────────────────────────────────────────────────────┘

⚠️ NEVER enable in production! Exposes sensitive information!
```

---

## Exception Handler Middleware

### Using UseExceptionHandler

```csharp
// File: Program.cs

var app = builder.Build();

// ═══════════════════════════════════════════════════════════════
// EXCEPTION HANDLER FOR PRODUCTION
// ═══════════════════════════════════════════════════════════════

if (!app.Environment.IsDevelopment())
{
    // Line 1: Redirect to error page on exception
    app.UseExceptionHandler("/Home/Error");
    
    // Line 2: Enable HSTS (HTTP Strict Transport Security)
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();
```

### Error Controller

```csharp
// File: Controllers/HomeController.cs

using Microsoft.AspNetCore.Diagnostics;
using Microsoft.AspNetCore.Mvc;

public class HomeController : Controller
{
    private readonly ILogger<HomeController> _logger;
    
    public HomeController(ILogger<HomeController> logger)
    {
        _logger = logger;
    }
    
    // ═══════════════════════════════════════════════════════════
    // ERROR ACTION
    // ═══════════════════════════════════════════════════════════
    
    [Route("Home/Error")]
    public IActionResult Error()
    {
        // Line 1: Get exception details
        var exceptionFeature = HttpContext.Features.Get<IExceptionHandlerPathFeature>();
        
        if (exceptionFeature != null)
        {
            // Line 2: Log the error
            _logger.LogError(
                "Error at {Path}: {Message}",
                exceptionFeature.Path,
                exceptionFeature.Error.Message);
            
            // Line 3: Create error model
            var errorModel = new ErrorViewModel
            {
                RequestId = Activity.Current?.Id ?? HttpContext.TraceIdentifier,
                Path = exceptionFeature.Path,
                ErrorMessage = app.Environment.IsDevelopment() 
                    ? exceptionFeature.Error.Message 
                    : "An error occurred while processing your request."
            };
            
            return View(errorModel);
        }
        
        return View(new ErrorViewModel
        {
            RequestId = Activity.Current?.Id ?? HttpContext.TraceIdentifier
        });
    }
}
```

### Error ViewModel

```csharp
// File: Models/ErrorViewModel.cs

public class ErrorViewModel
{
    public string? RequestId { get; set; }
    public string? Path { get; set; }
    public string? ErrorMessage { get; set; }
    
    public bool ShowRequestId => !string.IsNullOrEmpty(RequestId);
}
```

---

## Status Code Pages

### Basic Status Code Pages

```csharp
// File: Program.cs

var app = builder.Build();

// ═══════════════════════════════════════════════════════════════
// STATUS CODE PAGES
// ═══════════════════════════════════════════════════════════════

if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
    
    // Line 1: Show status code and message
    app.UseStatusCodePages();
    // Simple text: "Status Code: 404; Not Found"
}

app.UseHttpsRedirection();
// ... rest of pipeline
```

### Status Code Pages with Custom Format

```csharp
// ═══════════════════════════════════════════════════════════════
// CUSTOM FORMAT
// ═══════════════════════════════════════════════════════════════

app.UseStatusCodePages("text/html", 
    "<h1>Status Code: {0}</h1><p>An error occurred.</p>");
```

### Status Code Pages with Redirect

```csharp
// File: Program.cs

// ═══════════════════════════════════════════════════════════════
// REDIRECT TO ERROR PAGE WITH STATUS CODE
// ═══════════════════════════════════════════════════════════════

if (app.Environment.IsDevelopment())
{
    // Line 1: Redirect to error page with status code in URL
    app.UseStatusCodePagesWithRedirects("/Error/{0}");
    // 404 → /Error/404
    // 500 → /Error/500
}
```

### Error Controller for Status Codes

```csharp
// File: Controllers/ErrorController.cs

public class ErrorController : Controller
{
    // ═══════════════════════════════════════════════════════════
    // HANDLE DIFFERENT STATUS CODES
    // ═══════════════════════════════════════════════════════════
    
    [Route("Error/{statusCode}")]
    public IActionResult HttpStatusCodeHandler(int statusCode)
    {
        switch (statusCode)
        {
            case 404:
                ViewBag.ErrorMessage = "The page you requested could not be found.";
                ViewBag.ErrorTitle = "404 - Page Not Found";
                break;
                
            case 401:
                ViewBag.ErrorMessage = "You are not authorized to view this page.";
                ViewBag.ErrorTitle = "401 - Unauthorized";
                break;
                
            case 403:
                ViewBag.ErrorMessage = "You don't have permission to access this resource.";
                ViewBag.ErrorTitle = "403 - Forbidden";
                break;
                
            case 500:
                ViewBag.ErrorMessage = "An internal server error occurred.";
                ViewBag.ErrorTitle = "500 - Internal Server Error";
                break;
                
            default:
                ViewBag.ErrorMessage = "An error occurred while processing your request.";
                ViewBag.ErrorTitle = $"{statusCode} - Error";
                break;
        }
        
        return View("Error");
    }
}
```

---

## Custom Exception Middleware

### Creating Custom Middleware

```csharp
// File: Models/ExceptionHandlingMiddleware.cs

using Newtonsoft.Json;
using System.Net;

namespace MVCEmpDept.Models
{
    // Line 1: Custom exception handling middleware
    public class ExceptionHandlingMiddleware
    {
        private readonly RequestDelegate _next;
        private readonly ILogger<ExceptionHandlingMiddleware> _logger;
        
        // Line 2: Constructor with dependencies
        public ExceptionHandlingMiddleware(
            RequestDelegate next,
            ILogger<ExceptionHandlingMiddleware> logger)
        {
            _next = next;
            _logger = logger;
        }
        
        // Line 3: Invoke method - wraps request in try-catch
        public async Task InvokeAsync(HttpContext httpContext)
        {
            try
            {
                // Line 4: Call next middleware in pipeline
                await _next(httpContext);
            }
            catch (Exception ex)
            {
                // Line 5: Handle any exception
                await HandleExceptionAsync(httpContext, ex);
            }
        }
        
        // Line 6: Exception handling logic
        private async Task HandleExceptionAsync(
            HttpContext context, 
            Exception exception)
        {
            // Line 7: Log the exception
            _logger.LogError(exception, 
                "An unhandled exception occurred: {Message}", 
                exception.Message);
            
            // Line 8: Set status code
            var code = HttpStatusCode.InternalServerError; // 500
            
            // Line 9: Create error response
            var result = new BaseResponseDTO()
            {
                ErrorCode = (int)HttpStatusCode.InternalServerError,
                ErrorMessage = exception.Message,
                Succeed = false,
            };
            
            // Line 10: Serialize to JSON
            var jsonResult = JsonConvert.SerializeObject(result);
            
            // Line 11: Set response headers and body
            context.Response.ContentType = "application/json";
            context.Response.StatusCode = (int)code;
            await context.Response.WriteAsync(jsonResult);
        }
    }
}
```

### Response DTO

```csharp
// File: Models/BaseResponseDTO.cs

public class BaseResponseDTO
{
    public int ErrorCode { get; set; }
    public string ErrorMessage { get; set; }
    public bool Succeed { get; set; }
}
```

### Registering Custom Middleware

```csharp
// File: Program.cs

var app = builder.Build();

// ═══════════════════════════════════════════════════════════════
// REGISTER CUSTOM EXCEPTION MIDDLEWARE
// ═══════════════════════════════════════════════════════════════

// Line 1: Add custom middleware to pipeline
app.UseMiddleware<ExceptionHandlingMiddleware>();

// Important: Place early in pipeline to catch all exceptions
app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Employee}/{action=Index}/{id?}");

app.Run();
```

### Middleware Execution Flow

```
┌─────────────────────────────────────────────────────────────────┐
│            CUSTOM EXCEPTION MIDDLEWARE FLOW                      │
└─────────────────────────────────────────────────────────────────┘

Request arrives
     │
     ▼
┌──────────────────────────────────────────────────────────────┐
│  ExceptionHandlingMiddleware                                  │
│                                                              │
│  try {                                                       │
│      await _next(httpContext);  ──┐                         │
│  }                                │                          │
│  catch (Exception ex) {           │                          │
│      HandleException(ex);         │                          │
│  }                                │                          │
└───────────────────────────────────┼──────────────────────────┘
                                    │
                                    ▼
                    ┌───────────────────────────┐
                    │  StaticFiles Middleware   │
                    └────────────┬──────────────┘
                                 ▼
                    ┌───────────────────────────┐
                    │  Routing Middleware       │
                    └────────────┬──────────────┘
                                 ▼
                    ┌───────────────────────────┐
                    │  Controller Action        │
                    │                           │
                    │  EXCEPTION OCCURS! ────────┐
                    └───────────────────────────┘│
                                                 │
                    ┌────────────────────────────┘
                    │ Exception bubbles back up
                    ▼
┌──────────────────────────────────────────────────────────────┐
│  catch (Exception ex)                                         │
│  {                                                            │
│      Log error                                               │
│      Create error response                                   │
│      Return JSON with 500 status                             │
│  }                                                            │
└──────────────────────────────────────────────────────────────┘
```

---

## Error Views

### Error View

```html
<!-- File: Views/Shared/Error.cshtml -->

@model ErrorViewModel

@{
    ViewData["Title"] = "Error";
}

<div class="container mt-5">
    <div class="row">
        <div class="col-md-8 offset-md-2">
            <div class="card border-danger">
                <div class="card-header bg-danger text-white">
                    <h2>
                        <i class="bi bi-exclamation-triangle"></i>
                        @ViewBag.ErrorTitle ?? "An Error Occurred"
                    </h2>
                </div>
                <div class="card-body">
                    <!-- Error Message -->
                    <p class="lead">
                        @ViewBag.ErrorMessage ?? "We're sorry, but something went wrong."
                    </p>
                    
                    <!-- Request ID (for development/support) -->
                    @if (Model.ShowRequestId)
                    {
                        <p class="text-muted">
                            <strong>Request ID:</strong> @Model.RequestId
                        </p>
                    }
                    
                    <!-- Development Info -->
                    @if (!string.IsNullOrEmpty(Model.ErrorMessage))
                    {
                        <div class="alert alert-warning">
                            <strong>Details:</strong> @Model.ErrorMessage
                        </div>
                    }
                    
                    <!-- Actions -->
                    <div class="mt-4">
                        <a asp-controller="Home" asp-action="Index" class="btn btn-primary">
                            <i class="bi bi-house"></i> Go to Home
                        </a>
                        <button onclick="history.back()" class="btn btn-secondary">
                            <i class="bi bi-arrow-left"></i> Go Back
                        </button>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>
```

### 404 Not Found View

```html
<!-- File: Views/Error/404.cshtml -->

@{
    ViewData["Title"] = "404 - Page Not Found";
}

<div class="container mt-5 text-center">
    <h1 class="display-1">404</h1>
    <h2>Page Not Found</h2>
    <p class="lead">
        The page you're looking for doesn't exist or has been moved.
    </p>
    
    <div class="mt-4">
        <a asp-controller="Home" asp-action="Index" class="btn btn-lg btn-primary">
            Return Home
        </a>
    </div>
    
    <!-- Helpful suggestions -->
    <div class="mt-5">
        <h4>You might want to:</h4>
        <ul class="list-unstyled">
            <li><a asp-controller="Employee" asp-action="Index">Browse Employees</a></li>
            <li><a asp-controller="Department" asp-action="Index">View Departments</a></li>
            <li><a asp-controller="Home" asp-action="Contact">Contact Support</a></li>
        </ul>
    </div>
</div>
```

---

## Logging Errors

### Using ILogger

```csharp
// File: Controllers/EmployeeController.cs

public class EmployeeController : Controller
{
    private readonly IEmployeeService _service;
    private readonly ILogger<EmployeeController> _logger;
    
    public EmployeeController(
        IEmployeeService service,
        ILogger<EmployeeController> logger)
    {
        _service = service;
        _logger = logger;
    }
    
    public IActionResult Index()
    {
        try
        {
            // Line 1: Log information
            _logger.LogInformation("Fetching all employees");
            
            var employees = _service.GetAllEmployee();
            
            return View(employees);
        }
        catch (Exception ex)
        {
            // Line 2: Log error with exception
            _logger.LogError(ex, "Error fetching employees");
            
            // Line 3: Redirect to error page
            TempData["ErrorMessage"] = "Unable to load employees.";
            return RedirectToAction("Error", "Home");
        }
    }
}
```

### Log Levels

```csharp
// Different log levels for different scenarios

_logger.LogTrace("Very detailed logs (disabled by default)");
_logger.LogDebug("Debug information");
_logger.LogInformation("General information");
_logger.LogWarning("Warning messages");
_logger.LogError(exception, "Error messages");
_logger.LogCritical(exception, "Critical failures");
```

---

## Code Examples

### Complete Error Handling Setup

```csharp
// File: Program.cs

using Microsoft.EntityFrameworkCore;
using MVCEmpDept.Models;
using MVCEmpDept.Service;

var builder = WebApplication.CreateBuilder(args);

// Add services
builder.Services.AddControllersWithViews();
builder.Services.AddDbContextPool<AppdbContextRepository>(options =>
    options.UseSqlServer(
        builder.Configuration.GetConnectionString("EmployeeDBConnection")));
builder.Services.AddScoped<IEmployeeService, SqlEmployeeService>();

var app = builder.Build();

// ═══════════════════════════════════════════════════════════════
// ERROR HANDLING CONFIGURATION
// ═══════════════════════════════════════════════════════════════

if (!app.Environment.IsDevelopment())
{
    // Production: User-friendly error page
    app.UseExceptionHandler("/Home/Error");
    app.UseHsts();
}
else
{
    // Development: Detailed error page
    app.UseDeveloperExceptionPage();
    
    // Status code pages with redirect
    app.UseStatusCodePagesWithRedirects("/Error/{0}");
}

// Custom exception middleware
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

---

## Best Practices

### 1. Different Handling for Dev vs Production

```csharp
// ✅ GOOD: Show details in dev, hide in production
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}
else
{
    app.UseExceptionHandler("/Home/Error");
}

// ❌ BAD: Developer exception page in production
app.UseDeveloperExceptionPage(); // Exposes sensitive info!
```

### 2. Log All Errors

```csharp
// ✅ GOOD: Always log exceptions
catch (Exception ex)
{
    _logger.LogError(ex, "Error details");
    return View("Error");
}

// ❌ BAD: Silent failures
catch (Exception ex)
{
    // No logging - error disappears!
    return View("Error");
}
```

### 3. User-Friendly Error Messages

```csharp
// ✅ GOOD: Hide technical details from users
errorModel.Message = "We're experiencing technical difficulties.";

// ❌ BAD: Expose internal details
errorModel.Message = ex.Message; // "Connection to DB failed at..."
```

---

## Interview Questions

### Q1: What is the difference between UseExceptionHandler and UseDeveloperExceptionPage?
**Answer:**
- `UseDeveloperExceptionPage`: Shows detailed error information (stack trace, request details) - use only in development
- `UseExceptionHandler`: Redirects to custom error page - use in production for user-friendly errors

### Q2: What are Status Code Pages?
**Answer:** Status Code Pages middleware intercepts responses with status codes (like 404, 500) and allows you to display custom error pages or messages instead of default browser errors.

### Q3: How do you create custom exception middleware?
**Answer:** Create a class with constructor accepting `RequestDelegate` and `InvokeAsync` method. Wrap `await _next(context)` in try-catch to handle exceptions. Register with `app.UseMiddleware<YourMiddleware>()`.

### Q4: What is the purpose of Request ID in error pages?
**Answer:** Request ID uniquely identifies a request, making it easier to trace and debug errors in logs. Users can provide this ID to support teams for faster issue resolution.

### Q5: Where should error handling middleware be placed in the pipeline?
**Answer:** Early in the pipeline, typically right after environment-specific configuration. This ensures it can catch exceptions from all subsequent middleware and the application code.

### Q6: What are the different ILogger log levels?
**Answer:** Trace, Debug, Information, Warning, Error, Critical - in increasing order of severity.

---

## Quick Reference

### Error Handling Setup

```csharp
// Development
app.UseDeveloperExceptionPage();
app.UseStatusCodePages();

// Production
app.UseExceptionHandler("/Home/Error");
app.UseHsts();

// Custom middleware
app.UseMiddleware<ExceptionHandlingMiddleware>();
```

### Logging

```csharp
_logger.LogInformation("Info message");
_logger.LogWarning("Warning message");
_logger.LogError(ex, "Error message");
```

### Status Codes

```
404 - Not Found
401 - Unauthorized
403 - Forbidden
500 - Internal Server Error
503 - Service Unavailable
```
