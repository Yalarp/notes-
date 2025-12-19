# Request Pipeline in ASP.NET Core

## Table of Contents
1. [Introduction](#introduction)
2. [What is Request Pipeline?](#what-is-request-pipeline)
3. [Pipeline Flow](#pipeline-flow)
4. [Middleware Components](#middleware-components)
5. [Request/Response Cycle](#requestresponse-cycle)
6. [Pipeline Configuration](#pipeline-configuration)
7. [Code Examples](#code-examples)
8. [Best Practices](#best-practices)
9. [Interview Questions](#interview-questions)

---

## Introduction

### Real-World Analogy
The request pipeline is like an **airport security and processing system**:

```
Passenger (HTTP Request) arriving at airport
        │
        ▼
┌─────────────────┐
│  Check-in Desk  │  ← Static Files Middleware (quick exit if just picking up)
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   Security      │  ← Authentication Middleware
│   Screening     │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   Passport      │  ← Authorization Middleware
│   Control       │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   Gate/Flight   │  ← Endpoint (Controller Action)
│   (Destination) │
└─────────────────┘
         │
         ▼
   Response returns through same checkpoints (in reverse)
```

Each checkpoint can:
- **Pass the request** to the next checkpoint
- **Short-circuit** and return early (like if documents are invalid)
- **Modify** the request or response

---

## What is Request Pipeline?

The **Request Pipeline** (also called the HTTP pipeline) is a series of **middleware components** that process HTTP requests and responses in ASP.NET Core.

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Pipeline** | Sequence of middleware processing requests |
| **Middleware** | Component that handles part of request/response |
| **Request Delegate** | Function that processes an HTTP request |
| **Short-circuiting** | Stopping the pipeline early and returning response |

### Pipeline Visualization

```
┌─────────────────────────────────────────────────────────────────────┐
│                        REQUEST PIPELINE                              │
└─────────────────────────────────────────────────────────────────────┘

                         HTTP REQUEST
                              │
     ┌────────────────────────┼────────────────────────┐
     │                        │                        │
     │   ┌────────────────────▼───────────────────┐    │
     │   │        MIDDLEWARE 1 (Exception)        │    │
     │   │   ┌─────────────────────────────────┐  │    │
     │   │   │                                 │  │    │
     │   │   │  ┌──────────────────────────┐   │  │    │
     │   │   │  │   MIDDLEWARE 2 (HTTPS)   │   │  │    │
     │   │   │  │  ┌───────────────────┐   │   │  │    │
     │   │   │  │  │                   │   │   │  │    │
     │   │   │  │  │  ┌────────────┐   │   │   │  │    │
     │   │   │  │  │  │ MIDDLEWARE │   │   │   │  │    │
     │   │   │  │  │  │ 3 (Static) │   │   │   │  │    │
     │   │   │  │  │  │┌──────────┐│   │   │   │  │    │
     │   │   │  │  │  ││ROUTING   ││   │   │   │  │    │
     │   │   │  │  │  ││┌────────┐││   │   │   │  │    │
     │   │   │  │  │  │││ENDPOINT│││   │   │   │  │    │
     │   │   │  │  │  ││└────────┘││   │   │   │  │    │
     │   │   │  │  │  │└──────────┘│   │   │   │  │    │
     │   │   │  │  │  └────────────┘   │   │   │  │    │
     │   │   │  │  └───────────────────┘   │   │  │    │
     │   │   │  └──────────────────────────┘   │  │    │
     │   │   └─────────────────────────────────┘  │    │
     │   └────────────────────────────────────────┘    │
     │                        │                        │
     └────────────────────────┼────────────────────────┘
                              │
                         HTTP RESPONSE
```

---

## Pipeline Flow

### Request Flow (Inward)

```
HTTP Request → Middleware 1 → Middleware 2 → ... → Endpoint
```

### Response Flow (Outward)

```
Endpoint → ... → Middleware 2 → Middleware 1 → HTTP Response
```

### Complete Flow Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                    DETAILED REQUEST FLOW                             │
└─────────────────────────────────────────────────────────────────────┘

CLIENT
  │
  │ 1. HTTP Request (GET /api/products)
  ▼
┌────────────────────────────────────────────────────────────────────┐
│                         WEB SERVER                                  │
│                    (Kestrel / IIS)                                  │
└────────────────────────────────────────────────────────────────────┘
  │
  │ 2. Request enters pipeline
  ▼
┌────────────────────────────────────────────────────────────────────┐
│ MIDDLEWARE 1: Exception Handler                                     │
│ - Wraps entire pipeline in try-catch                               │
│ - Catches unhandled exceptions                                     │
│ - Entry: Log request start                                         │
│ - Exit: Handle any caught exceptions                               │
└───────────────────────────────┬────────────────────────────────────┘
                                │
                                ▼
┌────────────────────────────────────────────────────────────────────┐
│ MIDDLEWARE 2: HTTPS Redirection                                     │
│ - If HTTP, redirect to HTTPS                                       │
│ - Can short-circuit here with 307 redirect                         │
└───────────────────────────────┬────────────────────────────────────┘
                                │
                                ▼
┌────────────────────────────────────────────────────────────────────┐
│ MIDDLEWARE 3: Static Files                                          │
│ - Check if request is for static file (css, js, images)           │
│ - If yes: return file and SHORT-CIRCUIT (skip remaining)          │
│ - If no: pass to next middleware                                   │
└───────────────────────────────┬────────────────────────────────────┘
                                │
                                ▼
┌────────────────────────────────────────────────────────────────────┐
│ MIDDLEWARE 4: Routing                                               │
│ - Match URL to endpoint                                            │
│ - Sets Endpoint on HttpContext                                     │
│ - Does NOT execute endpoint yet                                    │
└───────────────────────────────┬────────────────────────────────────┘
                                │
                                ▼
┌────────────────────────────────────────────────────────────────────┐
│ MIDDLEWARE 5: Authentication                                        │
│ - Validate credentials (JWT, Cookie, etc.)                         │
│ - Sets User identity on HttpContext                                │
└───────────────────────────────┬────────────────────────────────────┘
                                │
                                ▼
┌────────────────────────────────────────────────────────────────────┐
│ MIDDLEWARE 6: Authorization                                         │
│ - Check if user has permission                                     │
│ - Can short-circuit with 401/403                                   │
└───────────────────────────────┬────────────────────────────────────┘
                                │
                                ▼
┌────────────────────────────────────────────────────────────────────┐
│ ENDPOINT EXECUTION                                                  │
│ - Execute matched controller action                                │
│ - Controller.Action() runs                                         │
│ - Generate response                                                │
└───────────────────────────────┬────────────────────────────────────┘
                                │
                                │ 3. Response flows back through middleware
                                ▼
                           HTTP Response
                                │
                                ▼
                             CLIENT
```

---

## Middleware Components

### Built-in Middleware

| Middleware | Purpose | Method |
|------------|---------|--------|
| Exception Handler | Catch unhandled exceptions | `UseExceptionHandler()` |
| HSTS | HTTP Strict Transport Security | `UseHsts()` |
| HTTPS Redirection | Redirect HTTP to HTTPS | `UseHttpsRedirection()` |
| Static Files | Serve static files | `UseStaticFiles()` |
| Routing | URL matching | `UseRouting()` |
| CORS | Cross-Origin Resource Sharing | `UseCors()` |
| Authentication | Identity verification | `UseAuthentication()` |
| Authorization | Permission checking | `UseAuthorization()` |
| Session | Session state | `UseSession()` |
| Endpoints | Execute endpoint | `MapControllers()` |

### Middleware Order (CRITICAL)

```csharp
// File: Program.cs
// CORRECT ORDER - Follow this sequence!

var app = builder.Build();

// 1. Exception handling (MUST be first)
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}
else
{
    app.UseExceptionHandler("/Error");
    app.UseHsts();  // 2. HSTS (production only)
}

// 3. HTTPS redirection
app.UseHttpsRedirection();

// 4. Static files (before routing for performance)
app.UseStaticFiles();

// 5. Cookie Policy
app.UseCookiePolicy();

// 6. Routing (determines which endpoint matches)
app.UseRouting();

// 7. CORS (after routing, before auth)
app.UseCors();

// 8. Authentication (who are you?)
app.UseAuthentication();

// 9. Authorization (are you allowed?)
app.UseAuthorization();

// 10. Session (if using sessions)
app.UseSession();

// 11. Custom middleware (your middleware)
app.UseMiddleware<CustomMiddleware>();

// 12. Endpoint execution (MUST be last)
app.MapControllers();             // Web API
app.MapControllerRoute(...);      // MVC

app.Run();
```

### Why Order Matters

```
❌ WRONG ORDER:

app.UseAuthorization();  // Authorization runs BEFORE...
app.UseRouting();        // ...routing determines the endpoint!
                         // Authorization doesn't know what to protect!

✅ CORRECT ORDER:

app.UseRouting();        // First, determine the endpoint
app.UseAuthorization();  // Then, check if user can access it
```

---

## Request/Response Cycle

### HttpContext

`HttpContext` encapsulates all HTTP-specific information about a request/response:

```csharp
public class HttpContext
{
    // Request information
    public HttpRequest Request { get; }      // URL, headers, body, cookies
    
    // Response information  
    public HttpResponse Response { get; }    // Status code, headers, body
    
    // User information
    public ClaimsPrincipal User { get; }     // Authenticated user
    
    // Connection information
    public ConnectionInfo Connection { get; } // IP, port
    
    // Service provider
    public IServiceProvider RequestServices { get; }
    
    // Items dictionary (share data between middleware)
    public IDictionary<object, object?> Items { get; }
}
```

### Request Processing Stages

```
┌─────────────────────────────────────────────────────────────────────┐
│                     REQUEST PROCESSING STAGES                        │
└─────────────────────────────────────────────────────────────────────┘

STAGE 1: Connection Management
─────────────────────────────────
• TCP connection established
• TLS handshake (if HTTPS)
• Connection kept alive for reuse

STAGE 2: Request Parsing
─────────────────────────────────
• HTTP method (GET, POST, etc.)
• URL and query string parsed
• Headers parsed
• Body buffered (if present)

STAGE 3: Middleware Processing
─────────────────────────────────
• Each middleware processes request
• Can modify request/response
• Can short-circuit pipeline

STAGE 4: Endpoint Execution
─────────────────────────────────
• Controller action executes
• Model binding occurs
• Action result created

STAGE 5: Response Generation
─────────────────────────────────
• Action result executed
• Response body written
• Response headers finalized

STAGE 6: Response Transmission
─────────────────────────────────
• Response sent to client
• Connection may be reused
• Logging completed
```

---

## Pipeline Configuration

### Complete Program.cs with Pipeline Configuration

```csharp
// File: Program.cs

using Microsoft.EntityFrameworkCore;
using MVCEmpDept.Models;
using MVCEmpDept.Repository;
using MVCEmpDept.Service;

namespace MVCEmpDept
{
    public class Program
    {
        public static void Main(string[] args)
        {
            // ═══════════════════════════════════════════════════════
            // PHASE 1: BUILDER CONFIGURATION (Services)
            // ═══════════════════════════════════════════════════════
            
            var builder = WebApplication.CreateBuilder(args);

            // Line 1: Add MVC services
            builder.Services.AddControllersWithViews();

            // Line 2: Add DbContext
            builder.Services.AddDbContextPool<AppdbContextRepository>(
                options => options.UseSqlServer(
                    builder.Configuration.GetConnectionString("EmployeeDBConnection")));

            // Line 3: Add application services
            builder.Services.AddScoped<IEmployeeService, SqlEmployeeService>();

            // ═══════════════════════════════════════════════════════
            // PHASE 2: BUILD APPLICATION
            // ═══════════════════════════════════════════════════════
            
            var app = builder.Build();

            // ═══════════════════════════════════════════════════════
            // PHASE 3: CONFIGURE MIDDLEWARE PIPELINE
            // ═══════════════════════════════════════════════════════

            // ─────────────────────────────────────────────────────
            // MIDDLEWARE 1: Exception Handling
            // ─────────────────────────────────────────────────────
            if (!app.Environment.IsDevelopment())
            {
                // Production: Use generic error handler
                app.UseExceptionHandler("/Home/Error");
                app.UseHsts();
            }
            else
            {
                // Development: Show detailed error pages
                // Line 4: Status code pages redirect for development
                app.UseStatusCodePagesWithRedirects("/Error/{0}");
            }

            // ─────────────────────────────────────────────────────
            // MIDDLEWARE 2: Custom Exception Handler
            // ─────────────────────────────────────────────────────
            app.UseMiddleware<ExceptionHandlingMiddleware>();

            // ─────────────────────────────────────────────────────
            // MIDDLEWARE 3: HTTPS Redirection
            // ─────────────────────────────────────────────────────
            app.UseHttpsRedirection();

            // ─────────────────────────────────────────────────────
            // MIDDLEWARE 4: Static Files
            // ─────────────────────────────────────────────────────
            // Serves files from wwwroot folder
            app.UseStaticFiles();

            // ─────────────────────────────────────────────────────
            // MIDDLEWARE 5: Routing
            // ─────────────────────────────────────────────────────
            // Matches URLs to endpoints
            app.UseRouting();

            // ─────────────────────────────────────────────────────
            // MIDDLEWARE 6: Authorization
            // ─────────────────────────────────────────────────────
            app.UseAuthorization();

            // ─────────────────────────────────────────────────────
            // ENDPOINT CONFIGURATION
            // ─────────────────────────────────────────────────────
            app.MapControllerRoute(
                name: "default",
                pattern: "{controller=Employee}/{action=Index}/{id?}");

            // ═══════════════════════════════════════════════════════
            // PHASE 4: RUN APPLICATION
            // ═══════════════════════════════════════════════════════
            app.Run();
        }
    }
}
```

### Middleware Methods Explained

| Method | What It Does |
|--------|--------------|
| `Use()` | Adds middleware that can call next |
| `UseMiddleware<T>()` | Adds class-based middleware |
| `Run()` | Terminal middleware (doesn't call next) |
| `Map()` | Branch pipeline based on path |
| `UseWhen()` | Conditional middleware execution |

---

## Code Examples

### Understanding Middleware Flow with Logs

```csharp
// File: Program.cs

var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// Middleware 1
app.Use(async (context, next) =>
{
    Console.WriteLine("1. Before Middleware 1");
    await next();  // Call next middleware
    Console.WriteLine("6. After Middleware 1");
});

// Middleware 2
app.Use(async (context, next) =>
{
    Console.WriteLine("2. Before Middleware 2");
    await next();  // Call next middleware
    Console.WriteLine("5. After Middleware 2");
});

// Middleware 3
app.Use(async (context, next) =>
{
    Console.WriteLine("3. Before Middleware 3");
    await next();  // Call next middleware
    Console.WriteLine("4. After Middleware 3");
});

// Terminal middleware (endpoint)
app.Run(async context =>
{
    Console.WriteLine("   >>> Endpoint Executing <<<");
    await context.Response.WriteAsync("Hello World!");
});

app.Run();

// OUTPUT:
// 1. Before Middleware 1
// 2. Before Middleware 2
// 3. Before Middleware 3
//    >>> Endpoint Executing <<<
// 4. After Middleware 3
// 5. After Middleware 2
// 6. After Middleware 1
```

### Short-Circuiting the Pipeline

```csharp
// Middleware that short-circuits for certain requests

app.Use(async (context, next) =>
{
    // Check if request is for maintenance page
    if (context.Request.Path.StartsWithSegments("/maintenance"))
    {
        // SHORT-CIRCUIT: Don't call next()
        context.Response.StatusCode = 503;
        await context.Response.WriteAsync("Site is under maintenance");
        return;  // Pipeline ends here
    }
    
    // Normal request: continue to next middleware
    await next();
});
```

### Branching the Pipeline with Map

```csharp
// Branch pipeline for specific paths

app.Map("/api", apiApp =>
{
    // This branch only handles /api/* requests
    apiApp.Run(async context =>
    {
        await context.Response.WriteAsync("API Branch");
    });
});

app.Map("/admin", adminApp =>
{
    // This branch only handles /admin/* requests
    adminApp.Run(async context =>
    {
        await context.Response.WriteAsync("Admin Branch");
    });
});

// Default for other paths
app.Run(async context =>
{
    await context.Response.WriteAsync("Default Branch");
});
```

---

## Best Practices

### 1. Follow Correct Middleware Order

```csharp
// ✅ CORRECT: Exception handler is FIRST
app.UseExceptionHandler("/Error");
app.UseStaticFiles();
app.UseRouting();

// ❌ WRONG: Exception handler after other middleware
app.UseStaticFiles();
app.UseExceptionHandler("/Error");  // Won't catch errors from UseStaticFiles!
```

### 2. Place Static Files Early

```csharp
// ✅ CORRECT: Static files before authentication
// Static files (CSS, JS) don't need authentication
app.UseStaticFiles();
app.UseAuthentication();

// ❌ WRONG: Forces static files through auth
app.UseAuthentication();
app.UseStaticFiles();  // Slower, unnecessary auth checks
```

### 3. Use Short-Circuit When Possible

```csharp
// ✅ GOOD: Early return for health checks
app.Use(async (context, next) =>
{
    if (context.Request.Path == "/health")
    {
        await context.Response.WriteAsync("Healthy");
        return;  // No need to run rest of pipeline
    }
    await next();
});
```

---

## Interview Questions

### Q1: What is the Request Processing Pipeline?
**Answer:** The Request Processing Pipeline is a sequence of middleware components that process HTTP requests and responses. Each middleware can inspect, modify, or short-circuit the request, and the response flows back through the same middleware in reverse order.

### Q2: What is middleware?
**Answer:** Middleware is software assembled into an application pipeline to handle requests and responses. Each middleware component can:
- Process the request before passing to the next component
- Decide whether to pass the request to the next component
- Process the response after the next component completes

### Q3: What is short-circuiting in the pipeline?
**Answer:** Short-circuiting occurs when a middleware decides not to call the next middleware and instead returns a response directly. This stops the request from traveling further down the pipeline. Examples include returning a cached response or rejecting an unauthorized request.

### Q4: Why does middleware order matter?
**Answer:** Middleware order matters because:
1. Each middleware runs in the order it's added
2. Some middleware depends on others (Authorization needs Routing)
3. Exception handlers must be first to catch all errors
4. Static files should be before auth for performance

### Q5: What is the difference between Use(), Run(), and Map()?
**Answer:**
- `Use()`: Adds middleware that can call the next middleware
- `Run()`: Adds terminal middleware that doesn't call next (ends pipeline)
- `Map()`: Branches the pipeline based on request path

---

## Quick Reference

### Pipeline Order Cheat Sheet

```
1.  UseExceptionHandler()     ← Exception handling (FIRST)
2.  UseHsts()                 ← HSTS (production only)
3.  UseHttpsRedirection()     ← HTTPS redirect
4.  UseStaticFiles()          ← Static files (before routing)
5.  UseCookiePolicy()         ← Cookie policy
6.  UseRouting()              ← Route matching
7.  UseCors()                 ← CORS
8.  UseAuthentication()       ← Who are you?
9.  UseAuthorization()        ← Are you allowed?
10. UseSession()              ← Session state
11. MapControllers()          ← Endpoint execution (LAST)
```

### Middleware Flow Pattern

```
Request  →  MW1.Before  →  MW2.Before  →  Endpoint
                                              │
Response ←  MW1.After   ←  MW2.After   ←  Response
```
