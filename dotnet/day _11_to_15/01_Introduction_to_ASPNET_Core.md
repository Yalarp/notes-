# Introduction to ASP.NET Core

## Table of Contents
1. [Introduction](#introduction)
2. [What is ASP.NET Core?](#what-is-aspnet-core)
3. [ASP.NET Core vs ASP.NET Framework](#aspnet-core-vs-aspnet-framework)
4. [Key Features](#key-features)
5. [Architecture Overview](#architecture-overview)
6. [Project Structure](#project-structure)
7. [Program.cs Deep Dive](#programcs-deep-dive)
8. [WebApplication and WebApplicationBuilder](#webapplication-and-webapplicationbuilder)
9. [Best Practices](#best-practices)
10. [Interview Questions](#interview-questions)

---

## Introduction

### Real-World Analogy
Think of ASP.NET Core like a **modern restaurant kitchen**:
- The **kitchen equipment** (framework) is modular - you only install what you need
- It can work in **any building** (cross-platform - Windows, Linux, macOS)
- The **cooking process** (request pipeline) is streamlined and efficient
- You can **customize the menu** (middleware) based on customer needs

### Why This Topic Matters
ASP.NET Core is Microsoft's modern, open-source, cross-platform framework for building web applications. Understanding its fundamentals is essential for:
- Building scalable web applications
- Creating RESTful APIs
- Developing real-time applications with SignalR
- Understanding modern .NET development practices

---

## What is ASP.NET Core?

ASP.NET Core is a **cross-platform, high-performance, open-source framework** for building modern, cloud-enabled, internet-connected applications.

### Key Characteristics

```
┌─────────────────────────────────────────────────────────────────┐
│                      ASP.NET Core                               │
├─────────────────────────────────────────────────────────────────┤
│  ✓ Cross-Platform (Windows, Linux, macOS)                      │
│  ✓ Open Source (MIT License)                                   │
│  ✓ High Performance                                             │
│  ✓ Modular Architecture                                         │
│  ✓ Built-in Dependency Injection                                │
│  ✓ Cloud-Ready                                                  │
│  ✓ Unified MVC and Web API                                      │
└─────────────────────────────────────────────────────────────────┘
```

### Types of Applications You Can Build

| Application Type | Description |
|-----------------|-------------|
| **MVC Web Apps** | Traditional server-rendered web applications |
| **Web APIs** | RESTful services for mobile/SPA clients |
| **Razor Pages** | Page-focused web applications |
| **Blazor** | Interactive web UIs with C# |
| **SignalR** | Real-time web applications |
| **gRPC Services** | High-performance RPC services |

---

## ASP.NET Core vs ASP.NET Framework

### Comparison Table

| Feature | ASP.NET Framework | ASP.NET Core |
|---------|------------------|--------------|
| **Platform** | Windows only | Cross-platform |
| **Performance** | Good | Excellent (up to 10x faster) |
| **Hosting** | IIS only | IIS, Kestrel, Nginx, Apache |
| **Open Source** | Partially | Fully open source |
| **Dependency Injection** | Third-party required | Built-in |
| **Configuration** | web.config (XML) | appsettings.json (JSON) |
| **Startup** | Global.asax | Program.cs |
| **Modularity** | Monolithic | Modular (NuGet packages) |

### Visual Comparison

```
ASP.NET Framework                    ASP.NET Core
┌─────────────────┐                  ┌─────────────────┐
│   Windows Only  │                  │  Cross-Platform │
│  ┌───────────┐  │                  │  ┌───────────┐  │
│  │    IIS    │  │                  │  │  Kestrel  │  │
│  └───────────┘  │                  │  │  + IIS    │  │
│  System.Web.dll │                  │  │  + Nginx  │  │
│   (Monolithic)  │                  │  └───────────┘  │
└─────────────────┘                  │  Modular NuGet  │
                                     └─────────────────┘
```

---

## Key Features

### 1. Cross-Platform Development
```csharp
// The same code runs on Windows, Linux, and macOS
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();
app.Run(); // Works everywhere!
```

### 2. Built-in Dependency Injection

```csharp
// Program.cs - Registering services
var builder = WebApplication.CreateBuilder(args);

// Line 1: AddControllersWithViews() registers MVC services
// This includes controllers, views, model binding, validation, etc.
builder.Services.AddControllersWithViews();

// Line 2: AddScoped registers a service with scoped lifetime
// A new instance is created once per HTTP request
builder.Services.AddScoped<IEmployeeService, SqlEmployeeService>();

// Line 3: AddDbContextPool registers DbContext with connection pooling
// This improves performance by reusing database connections
builder.Services.AddDbContextPool<AppdbContext>(
    options => options.UseSqlServer(connectionString)
);
```

**Explanation of Each Line:**
| Line | Code | Purpose |
|------|------|---------|
| 1 | `AddControllersWithViews()` | Registers MVC controller and view services |
| 2 | `AddScoped<I, T>()` | Registers service with scoped lifetime (per-request) |
| 3 | `AddDbContextPool<T>()` | Registers DbContext with connection pooling |

### 3. Modular Architecture

```csharp
// Only add what you need - no bloated dependencies
builder.Services.AddControllers();           // For Web API only
builder.Services.AddControllersWithViews();  // For MVC with views
builder.Services.AddRazorPages();            // For Razor Pages
```

### 4. Unified Programming Model

```csharp
// MVC Controller
public class HomeController : Controller
{
    public IActionResult Index() => View();
}

// Web API Controller - Same base patterns
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpGet]
    public IActionResult GetAll() => Ok(products);
}
```

---

## Architecture Overview

### Request Flow Diagram

```
┌──────────────────────────────────────────────────────────────────────┐
│                         CLIENT REQUEST                                │
└───────────────────────────────┬──────────────────────────────────────┘
                                │
                                ▼
┌──────────────────────────────────────────────────────────────────────┐
│                         WEB SERVER                                    │
│                   (IIS / Kestrel / Nginx)                            │
└───────────────────────────────┬──────────────────────────────────────┘
                                │
                                ▼
┌──────────────────────────────────────────────────────────────────────┐
│                      MIDDLEWARE PIPELINE                              │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐    │
│  │ Static  │→ │ Routing │→ │  Auth   │→ │  CORS   │→ │Exception│    │
│  │ Files   │  │         │  │         │  │         │  │ Handler │    │
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘  └─────────┘    │
└───────────────────────────────┬──────────────────────────────────────┘
                                │
                                ▼
┌──────────────────────────────────────────────────────────────────────┐
│                         ENDPOINT                                      │
│              (MVC Controller / Razor Page / Minimal API)             │
└───────────────────────────────┬──────────────────────────────────────┘
                                │
                                ▼
┌──────────────────────────────────────────────────────────────────────┐
│                         RESPONSE                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## Project Structure

### Typical ASP.NET Core MVC Project

```
MyWebApp/
├── Controllers/              # MVC Controllers
│   ├── HomeController.cs
│   └── EmployeeController.cs
├── Models/                   # Data models and ViewModels
│   ├── Employee.cs
│   └── ErrorViewModel.cs
├── Views/                    # Razor views (.cshtml)
│   ├── Home/
│   │   └── Index.cshtml
│   ├── Shared/
│   │   ├── _Layout.cshtml
│   │   └── _ValidationScriptsPartial.cshtml
│   ├── _ViewImports.cshtml
│   └── _ViewStart.cshtml
├── wwwroot/                  # Static files (CSS, JS, images)
│   ├── css/
│   ├── js/
│   └── lib/
├── Properties/
│   └── launchSettings.json   # Development launch profiles
├── appsettings.json          # Application configuration
├── appsettings.Development.json
└── Program.cs                # Application entry point
```

### File Purposes

| File/Folder | Purpose |
|-------------|---------|
| `Controllers/` | Contains MVC controllers that handle HTTP requests |
| `Models/` | Data models, DTOs, and ViewModels |
| `Views/` | Razor view files for rendering HTML |
| `wwwroot/` | Static files served directly to clients |
| `Program.cs` | Application entry point and configuration |
| `appsettings.json` | Application settings and connection strings |
| `launchSettings.json` | Development environment settings |

---

## Program.cs Deep Dive

### Complete Program.cs with Line-by-Line Explanation

```csharp
// Line 1: Import Entity Framework Core namespace for database operations
using Microsoft.EntityFrameworkCore;

// Line 2: Import dependency injection extensions
using Microsoft.Extensions.DependencyInjection.Extensions;

// Line 3-5: Import application-specific namespaces
using MVCEmpDept.Models;
using MVCEmpDept.Repository;
using MVCEmpDept.Service;

// Line 6: Import for JSON serialization options
using System.Text.Json.Serialization;

// Line 7: Define the namespace for this application
namespace MVCEmpDept
{
    // Line 8: Define the Program class - entry point of the application
    public class Program
    {
        // Line 9: Main method - the starting point of the application
        // 'args' contains command-line arguments passed to the application
        public static void Main(string[] args)
        {
            // Line 10: Create a WebApplicationBuilder
            // This is the first step - it sets up the application host
            // CreateBuilder configures default settings for logging, configuration, etc.
            var builder = WebApplication.CreateBuilder(args);

            // Line 11: Add MVC services (controllers + views) to the DI container
            // This enables the MVC pattern in our application
            builder.Services.AddControllersWithViews();

            // Line 12-14: Configure Entity Framework Core with SQL Server
            // AddDbContextPool: Creates a pool of DbContext instances for better performance
            // UseSqlServer: Specifies SQL Server as the database provider
            // GetConnectionString: Reads connection string from appsettings.json
            builder.Services.AddDbContextPool<AppdbContextRepository>(
                options => options.UseSqlServer(
                    builder.Configuration.GetConnectionString("EmployeeDBConnection")
                )
            );

            // Line 15: Register the employee service with scoped lifetime
            // Scoped: New instance created per HTTP request
            // IEmployeeService: Interface (abstraction)
            // SqlEmployeeService: Implementation (concrete class)
            builder.Services.AddScoped<IEmployeeService, SqlEmployeeService>();

            // Line 16: Build the WebApplication from the configured builder
            // This creates the application host with all configured services
            var app = builder.Build();

            // Line 17-23: Configure the HTTP request pipeline (middleware)
            // Different configuration for development vs production
            if (!app.Environment.IsDevelopment())
            {
                // Line 18: Use exception handler in production
                // Redirects to /Home/Error when an exception occurs
                app.UseExceptionHandler("/Home/Error");
                
                // Line 19: Enable HTTP Strict Transport Security (HSTS)
                // Forces browsers to use HTTPS for 30 days
                app.UseHsts();
            }
            else
            {
                // Line 20-21: Development-only error handling
                // Redirects to custom error page with status code
                app.UseStatusCodePagesWithRedirects("/Error/{0}");
            }

            // Line 22: Add custom exception handling middleware
            // This catches all unhandled exceptions globally
            app.UseMiddleware<ExceptionHandlingMiddleware>();

            // Line 23: Redirect HTTP requests to HTTPS
            app.UseHttpsRedirection();

            // Line 24: Enable serving static files from wwwroot folder
            app.UseStaticFiles();

            // Line 25: Enable routing - matches URLs to endpoints
            app.UseRouting();

            // Line 26: Enable authorization middleware
            // Checks if user has permission to access the resource
            app.UseAuthorization();

            // Line 27-29: Map controller routes
            // This defines the default URL pattern: /Controller/Action/Id
            app.MapControllerRoute(
                name: "default",
                pattern: "{controller=Employee}/{action=Index}/{id?}"
            );

            // Line 30: Run the application
            // Starts listening for incoming HTTP requests
            app.Run();
        }
    }
}
```

### Execution Flow Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                    APPLICATION STARTUP                           │
└───────────────────────────────┬─────────────────────────────────┘
                                │
                    ┌───────────▼───────────┐
                    │  CreateBuilder(args)   │
                    │  Creates host builder  │
                    └───────────┬───────────┘
                                │
                    ┌───────────▼───────────┐
                    │   Configure Services   │
                    │  - AddControllers      │
                    │  - AddDbContext        │
                    │  - AddScoped services  │
                    └───────────┬───────────┘
                                │
                    ┌───────────▼───────────┐
                    │      Build()           │
                    │  Creates WebApplication│
                    └───────────┬───────────┘
                                │
                    ┌───────────▼───────────┐
                    │  Configure Middleware  │
                    │  - UseStaticFiles      │
                    │  - UseRouting          │
                    │  - UseAuthorization    │
                    └───────────┬───────────┘
                                │
                    ┌───────────▼───────────┐
                    │        Run()           │
                    │  Start listening       │
                    └─────────────────────────┘
```

---

## WebApplication and WebApplicationBuilder

### WebApplicationBuilder

The `WebApplicationBuilder` is used to configure services and the application before it starts.

```csharp
var builder = WebApplication.CreateBuilder(args);
```

**What CreateBuilder Does:**
1. Sets the content root to the current directory
2. Configures the host for web scenarios
3. Loads configuration from multiple sources
4. Sets up logging providers
5. Adds default services

### Key Properties of WebApplicationBuilder

| Property | Type | Description |
|----------|------|-------------|
| `Services` | `IServiceCollection` | Collection to register services for DI |
| `Configuration` | `ConfigurationManager` | Access to configuration settings |
| `Environment` | `IWebHostEnvironment` | Information about hosting environment |
| `Logging` | `ILoggingBuilder` | Configure logging providers |
| `Host` | `ConfigureHostBuilder` | Configure generic host settings |
| `WebHost` | `ConfigureWebHostBuilder` | Configure web host settings |

### WebApplication

The `WebApplication` is the configured application ready to handle HTTP requests.

```csharp
var app = builder.Build();
```

**What Build() Returns:**
- A `WebApplication` instance
- Contains the dependency injection container
- Has middleware pipeline configured
- Ready to process HTTP requests

### Key Methods of WebApplication

| Method | Description |
|--------|-------------|
| `Run()` | Starts the application and blocks until shutdown |
| `RunAsync()` | Async version of Run |
| `UseMiddleware<T>()` | Adds middleware to the pipeline |
| `MapControllers()` | Maps controller endpoints |
| `MapControllerRoute()` | Maps conventional routes |

---

## Best Practices

### 1. Use Dependency Injection Properly

```csharp
// ✅ GOOD: Register interfaces with implementations
builder.Services.AddScoped<IEmployeeService, SqlEmployeeService>();

// ❌ BAD: Don't create instances manually
var service = new SqlEmployeeService(); // Avoid this!
```

### 2. Keep Program.cs Clean

```csharp
// ✅ GOOD: Use extension methods for complex configuration
builder.Services.AddApplicationServices();
builder.Services.AddInfrastructureServices(builder.Configuration);

// ❌ BAD: Long configuration blocks in Program.cs
```

### 3. Use Configuration Files

```csharp
// ✅ GOOD: Read from configuration
var connectionString = builder.Configuration.GetConnectionString("DefaultConnection");

// ❌ BAD: Hardcoded values
var connectionString = "Server=localhost;Database=MyDb;...";
```

### 4. Environment-Specific Configuration

```csharp
// ✅ GOOD: Different behavior for environments
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}
else
{
    app.UseExceptionHandler("/Error");
}
```

---

## Common Pitfalls

### 1. Middleware Order Matters

```csharp
// ❌ WRONG ORDER - Authorization before Routing
app.UseAuthorization();
app.UseRouting();

// ✅ CORRECT ORDER - Routing before Authorization
app.UseRouting();
app.UseAuthorization();
```

### 2. Forgetting to Add Services

```csharp
// ❌ Missing service registration
// This will throw: Unable to resolve service for type 'IEmployeeService'

// ✅ Always register services
builder.Services.AddScoped<IEmployeeService, SqlEmployeeService>();
```

### 3. Wrong Lifetime Selection

| Lifetime | Use When |
|----------|----------|
| **Singleton** | Stateless services, caches |
| **Scoped** | Database contexts, per-request services |
| **Transient** | Lightweight, stateless services |

---

## Interview Questions

### Q1: What is ASP.NET Core?
**Answer:** ASP.NET Core is a cross-platform, high-performance, open-source framework for building modern, cloud-enabled, internet-connected applications. It runs on Windows, Linux, and macOS, and is the redesigned version of ASP.NET Framework with improved modularity and performance.

### Q2: What is the difference between `CreateBuilder()` and `Build()`?
**Answer:** 
- `CreateBuilder()` returns a `WebApplicationBuilder` used to configure services and settings
- `Build()` creates the `WebApplication` from the configured builder, finalizing the DI container and middleware pipeline

### Q3: What is the return type of `WebApplication.CreateBuilder(args)`?
**Answer:** `WebApplicationBuilder` class

### Q4: What is the return type of `builder.Build()`?
**Answer:** `WebApplication` class

### Q5: Which class is used to configure the HTTP pipeline?
**Answer:** `WebApplication` class is used to configure the HTTP request pipeline through middleware methods like `UseRouting()`, `UseAuthorization()`, etc.

### Q6: In which method is `app.Run()` available?
**Answer:** `Run()` method is available in the `WebApplication` class.

### Q7: Why is ASP.NET Core faster than ASP.NET Framework?
**Answer:**
1. Cross-platform design with optimized code paths
2. Kestrel web server is highly optimized for performance
3. No dependency on System.Web.dll
4. Modular architecture - only load what you need
5. Improved memory management and garbage collection

### Q8: What are the main advantages of ASP.NET Core?
**Answer:**
1. Cross-platform support
2. High performance
3. Built-in dependency injection
4. Modular architecture
5. Open source
6. Side-by-side versioning
7. Unified MVC and Web API

---

## Quick Reference

```csharp
// Minimal ASP.NET Core Application
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllersWithViews();

var app = builder.Build();

app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();
```

### Service Lifetimes Cheat Sheet
| Method | Lifetime | Instance Created |
|--------|----------|------------------|
| `AddSingleton<T>()` | Singleton | Once per application |
| `AddScoped<T>()` | Scoped | Once per HTTP request |
| `AddTransient<T>()` | Transient | Every time requested |

### Essential Middleware Order
```csharp
app.UseExceptionHandler();    // 1. Exception handling
app.UseHsts();                // 2. HTTPS strict transport
app.UseHttpsRedirection();    // 3. Redirect to HTTPS
app.UseStaticFiles();         // 4. Serve static files
app.UseRouting();             // 5. Route matching
app.UseAuthentication();      // 6. Who are you?
app.UseAuthorization();       // 7. Are you allowed?
app.MapControllers();         // 8. Execute endpoint
```
