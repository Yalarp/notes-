# API Versioning in ASP.NET Core

## Table of Contents
1. [Introduction](#introduction)
2. [Why Version Your API?](#why-version-your-api)
3. [Versioning Strategies](#versioning-strategies)
4. [Setting Up API Versioning](#setting-up-api-versioning)
5. [URL Path Versioning](#url-path-versioning)
6. [Query String Versioning](#query-string-versioning)
7. [Header Versioning](#header-versioning)
8. [Deprecating Versions](#deprecating-versions)
9. [Best Practices](#best-practices)
10. [Interview Questions](#interview-questions)

---

## Introduction

### Real-World Analogy
API versioning is like **software version updates**:

```
v1.0 App → Users on old phones
v2.0 App → Users with new features
v3.0 App → Latest features

Same app, different versions running simultaneously!
Users upgrade when ready, old versions still work.
```

---

## Why Version Your API?

### Scenarios Requiring Versioning

| Scenario | Problem | Solution |
|----------|---------|----------|
| Breaking changes | Old clients break | Keep old version, new version for new clients |
| New features | Can't modify existing endpoints | Add features to new version |
| Different clients | Mobile needs different data | Version per client requirements |
| Deprecation | Need to phase out old API | Mark old version deprecated |

### Without Versioning

```
┌─────────────────────────────────────────────────────────────────┐
│                    WITHOUT VERSIONING                            │
└─────────────────────────────────────────────────────────────────┘

Old Response:                    New Response (Breaking!):
{                               {
  "name": "John",                 "firstName": "John",
  "surname": "Doe"                "lastName": "Doe"
}                               }

❌ All old clients break immediately!
```

### With Versioning

```
┌─────────────────────────────────────────────────────────────────┐
│                    WITH VERSIONING                               │
└─────────────────────────────────────────────────────────────────┘

/api/v1/users                    /api/v2/users
{                               {
  "name": "John",                 "firstName": "John",
  "surname": "Doe"                "lastName": "Doe"
}                               }

✅ Old clients continue working with v1
✅ New clients use v2
```

---

## Versioning Strategies

### Comparison

| Strategy | URL Example | Pros | Cons |
|----------|-------------|------|------|
| **URL Path** | `/api/v1/users` | Clear, cacheable | URL changes |
| **Query String** | `/api/users?v=1` | Clean URLs | Easy to miss |
| **Header** | `X-Api-Version: 1` | Clean URLs | Hidden |
| **Media Type** | `Accept: application/vnd.api.v1+json` | RESTful | Complex |

---

## Setting Up API Versioning

### Install NuGet Package

```bash
dotnet add package Asp.Versioning.Mvc
# OR for API projects
dotnet add package Asp.Versioning.Mvc.ApiExplorer
```

### Basic Configuration

```csharp
// File: Program.cs

using Asp.Versioning;

var builder = WebApplication.CreateBuilder(args);

// ═══════════════════════════════════════════════════════════════
// CONFIGURE API VERSIONING
// ═══════════════════════════════════════════════════════════════

builder.Services.AddApiVersioning(options =>
{
    // Default version when not specified
    options.DefaultApiVersion = new ApiVersion(1, 0);
    
    // Assume default version if not specified
    options.AssumeDefaultVersionWhenUnspecified = true;
    
    // Report available versions in response header
    options.ReportApiVersions = true;
    
    // How to read version from request
    options.ApiVersionReader = ApiVersionReader.Combine(
        new UrlSegmentApiVersionReader(),
        new QueryStringApiVersionReader("api-version"),
        new HeaderApiVersionReader("X-Api-Version")
    );
})
.AddApiExplorer(options =>
{
    // Format: 'v'major[.minor]
    options.GroupNameFormat = "'v'VVV";
    options.SubstituteApiVersionInUrl = true;
});

builder.Services.AddControllers();

var app = builder.Build();

app.MapControllers();
app.Run();
```

---

## URL Path Versioning

### Most Common Approach

```csharp
// File: Controllers/V1/ProductsController.cs

namespace MyApi.Controllers.V1
{
    [ApiController]
    [ApiVersion("1.0")]
    [Route("api/v{version:apiVersion}/[controller]")]
    public class ProductsController : ControllerBase
    {
        [HttpGet]
        public IActionResult GetProducts()
        {
            return Ok(new[]
            {
                new { Id = 1, Name = "Product A", Price = 10.00m }
            });
        }
    }
}

// File: Controllers/V2/ProductsController.cs

namespace MyApi.Controllers.V2
{
    [ApiController]
    [ApiVersion("2.0")]
    [Route("api/v{version:apiVersion}/[controller]")]
    public class ProductsController : ControllerBase
    {
        [HttpGet]
        public IActionResult GetProducts()
        {
            return Ok(new[]
            {
                new 
                { 
                    Id = 1, 
                    ProductName = "Product A",  // Changed property name
                    UnitPrice = 10.00m,
                    Currency = "USD"            // New property
                }
            });
        }
    }
}
```

### Request Examples

```
GET /api/v1/products → Uses V1 controller
GET /api/v2/products → Uses V2 controller
```

---

## Query String Versioning

### Configuration

```csharp
builder.Services.AddApiVersioning(options =>
{
    options.ApiVersionReader = new QueryStringApiVersionReader("api-version");
});
```

### Controller

```csharp
[ApiController]
[ApiVersion("1.0")]
[ApiVersion("2.0")]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpGet]
    [MapToApiVersion("1.0")]
    public IActionResult GetV1()
    {
        return Ok("Version 1.0 response");
    }

    [HttpGet]
    [MapToApiVersion("2.0")]
    public IActionResult GetV2()
    {
        return Ok("Version 2.0 response");
    }
}
```

### Request Examples

```
GET /api/products                  → Default version (1.0)
GET /api/products?api-version=1.0  → Version 1.0
GET /api/products?api-version=2.0  → Version 2.0
```

---

## Header Versioning

### Configuration

```csharp
builder.Services.AddApiVersioning(options =>
{
    options.ApiVersionReader = new HeaderApiVersionReader("X-Api-Version");
});
```

### Request Examples

```
GET /api/products
X-Api-Version: 1.0
→ Returns Version 1.0 response

GET /api/products
X-Api-Version: 2.0
→ Returns Version 2.0 response
```

---

## Multiple Version Readers

### Combine Strategies

```csharp
builder.Services.AddApiVersioning(options =>
{
    options.ApiVersionReader = ApiVersionReader.Combine(
        // URL: /api/v1/products
        new UrlSegmentApiVersionReader(),
        
        // Query: /api/products?api-version=1.0
        new QueryStringApiVersionReader("api-version"),
        
        // Header: X-Api-Version: 1.0
        new HeaderApiVersionReader("X-Api-Version"),
        
        // Media Type: Accept: application/json;v=1.0
        new MediaTypeApiVersionReader("v")
    );
});
```

---

## Version Mapping in Controllers

### Single Controller, Multiple Versions

```csharp
[ApiController]
[Route("api/v{version:apiVersion}/users")]
[ApiVersion("1.0")]
[ApiVersion("2.0")]
[ApiVersion("3.0")]
public class UsersController : ControllerBase
{
    // Available in all versions (1.0, 2.0, 3.0)
    [HttpGet]
    public IActionResult GetUsers()
    {
        return Ok("Available in all versions");
    }

    // Only version 1.0
    [HttpGet("legacy")]
    [MapToApiVersion("1.0")]
    public IActionResult GetLegacy()
    {
        return Ok("Only in v1.0");
    }

    // Only version 2.0 and above
    [HttpGet("new-feature")]
    [MapToApiVersion("2.0")]
    [MapToApiVersion("3.0")]
    public IActionResult GetNewFeature()
    {
        return Ok("Available in v2.0 and v3.0");
    }

    // Only version 3.0
    [HttpGet("latest")]
    [MapToApiVersion("3.0")]
    public IActionResult GetLatest()
    {
        return Ok("Only in v3.0");
    }
}
```

---

## Deprecating Versions

### Mark Version as Deprecated

```csharp
[ApiController]
[Route("api/v{version:apiVersion}/[controller]")]
[ApiVersion("1.0", Deprecated = true)]  // This version is deprecated
[ApiVersion("2.0")]
public class ProductsController : ControllerBase
{
    [HttpGet]
    [MapToApiVersion("1.0")]
    public IActionResult GetV1()
    {
        return Ok("Deprecated version 1.0");
    }

    [HttpGet]
    [MapToApiVersion("2.0")]
    public IActionResult GetV2()
    {
        return Ok("Current version 2.0");
    }
}
```

### Response Headers

```
Response to GET /api/v1/products:

api-deprecated-versions: 1.0
api-supported-versions: 2.0
```

---

## Swagger Integration

### Configure Swagger for Multiple Versions

```csharp
// Program.cs
builder.Services.AddSwaggerGen(options =>
{
    options.SwaggerDoc("v1", new OpenApiInfo
    {
        Title = "My API",
        Version = "v1",
        Description = "API Version 1.0"
    });
    
    options.SwaggerDoc("v2", new OpenApiInfo
    {
        Title = "My API",
        Version = "v2",
        Description = "API Version 2.0"
    });
});

// Configure Swagger UI
app.UseSwaggerUI(options =>
{
    options.SwaggerEndpoint("/swagger/v1/swagger.json", "V1");
    options.SwaggerEndpoint("/swagger/v2/swagger.json", "V2");
});
```

---

## Best Practices

### 1. Use Semantic Versioning

```csharp
[ApiVersion("1.0")]     // Major.Minor
[ApiVersion("1.1")]     // Minor update (backward compatible)
[ApiVersion("2.0")]     // Major update (breaking changes)
```

### 2. Start with URL Path Versioning

```csharp
// Most explicit and widely understood
[Route("api/v{version:apiVersion}/[controller]")]
```

### 3. Document Deprecation Schedule

```csharp
[ApiVersion("1.0", Deprecated = true)]
// Add documentation about sunset date
/// <summary>
/// Deprecated. Will be removed on 2025-01-01.
/// Use v2.0 instead.
/// </summary>
```

### 4. Maintain Backward Compatibility When Possible

```csharp
// ✅ GOOD: Add new optional properties
public class ProductV2 : ProductV1
{
    public string? NewOptionalField { get; set; }
}

// ❌ BAD: Change required property names
public class ProductV2  // Breaking change!
{
    public string ProductName { get; set; }  // Was "Name" in V1
}
```

---

## Interview Questions

### Q1: Why should you version your API?
**Answer:**
- Maintain backward compatibility for existing clients
- Introduce breaking changes safely
- Allow gradual migration to new versions
- Support multiple clients with different needs

### Q2: What are the common API versioning strategies?
**Answer:**
- **URL Path**: `/api/v1/resource`
- **Query String**: `/api/resource?v=1`
- **Header**: `X-Api-Version: 1`
- **Media Type**: `Accept: application/vnd.api.v1+json`

### Q3: Which versioning strategy is most commonly used?
**Answer:** URL path versioning because it's explicit, discoverable, and cacheable. Clients can immediately see which version they're using.

### Q4: How do you deprecate an API version?
**Answer:** Use `[ApiVersion("1.0", Deprecated = true)]`. This adds deprecation headers to responses, informing clients that the version will be removed.

### Q5: What is semantic versioning?
**Answer:** A versioning scheme: MAJOR.MINOR.PATCH
- MAJOR: Breaking changes
- MINOR: New features (backward compatible)
- PATCH: Bug fixes

---

## Quick Reference

### Setup

```csharp
builder.Services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.ReportApiVersions = true;
});
```

### Controller Attributes

```csharp
[ApiVersion("1.0")]              // Supports version 1.0
[ApiVersion("2.0")]              // Also supports version 2.0
[ApiVersion("1.0", Deprecated = true)]  // Deprecated

[MapToApiVersion("1.0")]         // Map action to specific version

[Route("api/v{version:apiVersion}/[controller]")]  // URL versioning
```

### Response Headers

```
api-supported-versions: 1.0, 2.0
api-deprecated-versions: 1.0
```
