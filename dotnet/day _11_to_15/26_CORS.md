# CORS in ASP.NET Core

## Table of Contents
1. [Introduction](#introduction)
2. [What is CORS?](#what-is-cors)
3. [How CORS Works](#how-cors-works)
4. [Configuring CORS](#configuring-cors)
5. [CORS Policies](#cors-policies)
6. [Preflight Requests](#preflight-requests)
7. [Common Scenarios](#common-scenarios)
8. [Best Practices](#best-practices)
9. [Interview Questions](#interview-questions)

---

## Introduction

### Real-World Analogy
CORS is like a **building security system**:

```
Without CORS (Same-Origin Policy):
    You can only enter your own building
    ❌ Can't visit other buildings

With CORS (Cross-Origin Allowed):
    Building security checks visitor list
    ✅ If you're on the list, you can enter
    ❌ If not on list, access denied
```

---

## What is CORS?

**CORS** (Cross-Origin Resource Sharing) is a security feature that allows or restricts web applications from making requests to a different domain than the one serving the web page.

### Same-Origin Policy

Browsers enforce the Same-Origin Policy which blocks requests from different origins.

```
Origin = Protocol + Domain + Port

https://myapp.com:443     (Origin A)
https://api.myapp.com:443 (Origin B - Different subdomain)
https://myapp.com:8080    (Origin C - Different port)
http://myapp.com:443      (Origin D - Different protocol)

All are considered DIFFERENT origins!
```

### When CORS Applies

```
┌─────────────────────────────────────────────────────────────────┐
│                    CORS SCENARIO                                 │
└─────────────────────────────────────────────────────────────────┘

Frontend: https://myapp.com (React, Angular, Vue)
                    │
                    │ API Request (fetch/axios)
                    ▼
Backend: https://api.myapp.com (ASP.NET Core API)

Browser says: "Different origins! Let me check CORS headers..."

If API allows it → Request succeeds ✅
If API blocks it → CORS error ❌
```

---

## How CORS Works

### CORS Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                    SIMPLE CORS REQUEST                           │
└─────────────────────────────────────────────────────────────────┘

1. Browser sends request with Origin header:
   
   Request:
   GET /api/products
   Origin: https://myapp.com

2. Server responds with Access-Control headers:
   
   Response:
   Access-Control-Allow-Origin: https://myapp.com
   [data...]

3. Browser checks:
   - Does Origin match Allow-Origin? 
   - Yes → Allow response ✅
   - No → Block response ❌
```

### CORS Headers

| Header | Description |
|--------|-------------|
| `Access-Control-Allow-Origin` | Which origins are allowed |
| `Access-Control-Allow-Methods` | Which HTTP methods are allowed |
| `Access-Control-Allow-Headers` | Which request headers are allowed |
| `Access-Control-Allow-Credentials` | Whether cookies/auth can be sent |
| `Access-Control-Max-Age` | How long to cache preflight |
| `Access-Control-Expose-Headers` | Which headers client can read |

---

## Configuring CORS

### Method 1: Named Policy

```csharp
// File: Program.cs

var builder = WebApplication.CreateBuilder(args);

// ═══════════════════════════════════════════════════════════════
// STEP 1: Configure CORS (Add Services)
// ═══════════════════════════════════════════════════════════════

builder.Services.AddCors(options =>
{
    // Named policy
    options.AddPolicy("AllowMyApp", policy =>
    {
        policy.WithOrigins("https://myapp.com", "https://www.myapp.com")
              .WithMethods("GET", "POST", "PUT", "DELETE")
              .WithHeaders("Content-Type", "Authorization")
              .AllowCredentials();
    });
    
    // Another policy for different scenario
    options.AddPolicy("AllowAll", policy =>
    {
        policy.AllowAnyOrigin()
              .AllowAnyMethod()
              .AllowAnyHeader();
    });
});

builder.Services.AddControllers();

var app = builder.Build();

// ═══════════════════════════════════════════════════════════════
// STEP 2: Use CORS Middleware
// ═══════════════════════════════════════════════════════════════

// Use specific policy
app.UseCors("AllowMyApp");

// OR use per-endpoint (see below)

app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();
app.Run();
```

### Method 2: Default Policy

```csharp
builder.Services.AddCors(options =>
{
    options.AddDefaultPolicy(policy =>
    {
        policy.WithOrigins("https://myapp.com")
              .AllowAnyMethod()
              .AllowAnyHeader();
    });
});

// Use default policy
app.UseCors();  // No policy name needed
```

### Method 3: Per-Endpoint

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    // Apply CORS to specific action
    [HttpGet]
    [EnableCors("AllowAll")]
    public IActionResult GetProducts() { }
    
    // Disable CORS for specific action
    [HttpPost]
    [DisableCors]
    public IActionResult CreateProduct() { }
}

// OR at controller level
[EnableCors("AllowMyApp")]
[ApiController]
[Route("api/[controller]")]
public class OrdersController : ControllerBase
{
    // All actions use "AllowMyApp" policy
}
```

---

## CORS Policies

### Allow Specific Origins

```csharp
options.AddPolicy("Production", policy =>
{
    policy.WithOrigins(
        "https://myapp.com",
        "https://www.myapp.com",
        "https://admin.myapp.com"
    );
});
```

### Allow Any Origin

```csharp
// ⚠️ Use with caution - allows all origins
options.AddPolicy("Public", policy =>
{
    policy.AllowAnyOrigin()
          .AllowAnyMethod()
          .AllowAnyHeader();
});

// Note: Cannot use AllowCredentials with AllowAnyOrigin!
```

### Allow Credentials (Cookies/Auth)

```csharp
options.AddPolicy("WithCredentials", policy =>
{
    policy.WithOrigins("https://myapp.com")  // Must specify origin
          .AllowCredentials()                 // Allow cookies
          .AllowAnyMethod()
          .AllowAnyHeader();
});
```

### Expose Custom Headers

```csharp
options.AddPolicy("ExposeHeaders", policy =>
{
    policy.WithOrigins("https://myapp.com")
          .WithExposedHeaders("X-Custom-Header", "X-Pagination");
          // Client JavaScript can now read these headers
});
```

### Dynamic Origin Validation

```csharp
options.AddPolicy("Dynamic", policy =>
{
    policy.SetIsOriginAllowed(origin =>
    {
        // Custom logic to determine if origin is allowed
        var uri = new Uri(origin);
        return uri.Host.EndsWith(".mycompany.com");
    })
    .AllowAnyMethod()
    .AllowAnyHeader();
});
```

---

## Preflight Requests

### What is Preflight?

For "complex" requests, browsers send a preflight OPTIONS request first.

```
┌─────────────────────────────────────────────────────────────────┐
│                    PREFLIGHT REQUEST FLOW                        │
└─────────────────────────────────────────────────────────────────┘

1. Browser wants to send POST with JSON:
   
   POST /api/products
   Content-Type: application/json
   Authorization: Bearer token123
   
2. Browser sends preflight first:
   
   OPTIONS /api/products
   Origin: https://myapp.com
   Access-Control-Request-Method: POST
   Access-Control-Request-Headers: Content-Type, Authorization

3. Server responds with permissions:
   
   Access-Control-Allow-Origin: https://myapp.com
   Access-Control-Allow-Methods: POST
   Access-Control-Allow-Headers: Content-Type, Authorization
   Access-Control-Max-Age: 86400

4. Browser checks response → If allowed, sends actual request
```

### When Preflight Happens

| Condition | Simple Request | Preflight Required |
|-----------|----------------|-------------------|
| Methods | GET, HEAD, POST | PUT, DELETE, PATCH |
| Content-Type | text/plain, form-data | application/json |
| Custom Headers | None | Authorization, X-Custom |

### Cache Preflight

```csharp
options.AddPolicy("CachePreflight", policy =>
{
    policy.WithOrigins("https://myapp.com")
          .AllowAnyMethod()
          .AllowAnyHeader()
          .SetPreflightMaxAge(TimeSpan.FromHours(24));
          // Browser caches preflight result for 24 hours
});
```

---

## Common Scenarios

### Scenario 1: React + ASP.NET Core API

```csharp
// Program.cs - Development
builder.Services.AddCors(options =>
{
    options.AddPolicy("Development", policy =>
    {
        policy.WithOrigins("http://localhost:3000")  // React dev server
              .AllowAnyMethod()
              .AllowAnyHeader()
              .AllowCredentials();
    });
});

if (app.Environment.IsDevelopment())
{
    app.UseCors("Development");
}
```

### Scenario 2: Multiple Environments

```csharp
// Get allowed origins from configuration
var allowedOrigins = builder.Configuration
    .GetSection("Cors:AllowedOrigins")
    .Get<string[]>();

builder.Services.AddCors(options =>
{
    options.AddDefaultPolicy(policy =>
    {
        policy.WithOrigins(allowedOrigins)
              .AllowAnyMethod()
              .AllowAnyHeader()
              .AllowCredentials();
    });
});
```

```json
// appsettings.Development.json
{
  "Cors": {
    "AllowedOrigins": ["http://localhost:3000", "http://localhost:4200"]
  }
}

// appsettings.Production.json
{
  "Cors": {
    "AllowedOrigins": ["https://myapp.com", "https://www.myapp.com"]
  }
}
```

### Scenario 3: Minimal API

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddCors();

var app = builder.Build();

app.UseCors(policy => policy
    .WithOrigins("https://myapp.com")
    .AllowAnyMethod()
    .AllowAnyHeader());

app.MapGet("/api/data", () => new { message = "Hello" });

app.Run();
```

---

## Best Practices

### 1. Never Use AllowAnyOrigin in Production

```csharp
// ❌ BAD: Opens API to all websites
policy.AllowAnyOrigin();

// ✅ GOOD: Specify allowed origins
policy.WithOrigins("https://myapp.com");
```

### 2. Be Specific with Methods and Headers

```csharp
// ❌ BAD: Allows everything
policy.AllowAnyMethod().AllowAnyHeader();

// ✅ GOOD: Only allow what's needed
policy.WithMethods("GET", "POST")
      .WithHeaders("Content-Type", "Authorization");
```

### 3. Use Environment-Specific Configuration

```csharp
if (app.Environment.IsDevelopment())
{
    app.UseCors("Development");  // Relaxed for dev
}
else
{
    app.UseCors("Production");   // Strict for prod
}
```

### 4. CORS Middleware Order

```csharp
// ✅ Correct order
app.UseRouting();
app.UseCors();          // After routing
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();
```

---

## Interview Questions

### Q1: What is CORS?
**Answer:** CORS (Cross-Origin Resource Sharing) is a security mechanism that allows web browsers to make requests to a different domain than the one serving the web page. It uses HTTP headers to tell browsers to allow requests from specific origins.

### Q2: Why is CORS needed?
**Answer:** Browsers enforce the Same-Origin Policy for security. CORS provides a controlled way to relax this restriction, allowing legitimate cross-origin requests while still protecting against malicious ones.

### Q3: What is a preflight request?
**Answer:** A preflight request is an OPTIONS request sent by the browser before the actual request to check if the server allows the cross-origin request. It's triggered by "complex" requests (non-GET/POST, custom headers, JSON content-type).

### Q4: Difference between AllowAnyOrigin and WithOrigins?
**Answer:**
- `AllowAnyOrigin`: Allows requests from any origin (risky)
- `WithOrigins`: Allows only specified origins (secure)
- Note: `AllowCredentials` cannot be used with `AllowAnyOrigin`

### Q5: How do you enable CORS for specific endpoints?
**Answer:** Use `[EnableCors("PolicyName")]` attribute on controller or action, or use endpoint routing: `app.MapGet(...).RequireCors("PolicyName")`.

---

## Quick Reference

### Basic Setup

```csharp
// Add service
builder.Services.AddCors(options =>
{
    options.AddDefaultPolicy(policy =>
    {
        policy.WithOrigins("https://myapp.com")
              .AllowAnyMethod()
              .AllowAnyHeader();
    });
});

// Use middleware
app.UseCors();
```

### Common Headers

```
Response Headers:
Access-Control-Allow-Origin: https://myapp.com
Access-Control-Allow-Methods: GET, POST
Access-Control-Allow-Headers: Content-Type
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 86400
```
