# Kestrel Server in ASP.NET Core

## Table of Contents
1. [Introduction](#introduction)
2. [What is Kestrel?](#what-is-kestrel)
3. [Kestrel Architecture](#kestrel-architecture)
4. [Kestrel Configuration](#kestrel-configuration)
5. [Port Configuration](#port-configuration)
6. [HTTPS Configuration](#https-configuration)
7. [Kestrel with Reverse Proxy](#kestrel-with-reverse-proxy)
8. [Performance Features](#performance-features)
9. [Code Examples](#code-examples)
10. [Best Practices](#best-practices)
11. [Interview Questions](#interview-questions)

---

## Introduction

### Real-World Analogy
Think of Kestrel like a **high-speed reception desk**:
- It's lightweight and extremely fast at handling incoming requests
- It can work alone (for internal applications)
- Or work behind a security guard (reverse proxy) for public-facing apps
- It's part of the building (cross-platform, built into ASP.NET Core)

```
Without Reverse Proxy              With Reverse Proxy
(Development/Internal)             (Production)

┌─────────────┐                   ┌─────────────┐
│   Client    │                   │   Client    │
└──────┬──────┘                   └──────┬──────┘
       │                                 │
       ▼                                 ▼
┌─────────────┐                   ┌─────────────┐
│   Kestrel   │                   │  IIS/Nginx  │ ← Security, SSL
│   (Fast)    │                   │  (Proxy)    │   Load Balancing
└─────────────┘                   └──────┬──────┘
                                         │
                                         ▼
                                  ┌─────────────┐
                                  │   Kestrel   │
                                  │   (Fast)    │
                                  └─────────────┘
```

---

## What is Kestrel?

**Kestrel** is a cross-platform, high-performance web server for ASP.NET Core applications.

### Key Characteristics

| Feature | Description |
|---------|-------------|
| **Cross-Platform** | Works on Windows, Linux, macOS |
| **High-Performance** | Built for speed using modern async I/O |
| **Built-In** | Included with ASP.NET Core by default |
| **Lightweight** | Minimal overhead, efficient resource usage |
| **HTTP/2 Support** | Modern protocol support |
| **WebSocket Support** | Real-time communication |

### How Kestrel Fits In

```
┌──────────────────────────────────────────────────────────────┐
│                    ASP.NET Core Application                   │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐  │
│  │                     KESTREL                            │  │
│  │                                                        │  │
│  │   ┌──────────────┐    ┌──────────────────────────────┐   │
│  │   │   Listener   │───▶│     Request Processing      │   │
│  │   │  (Port 5000) │    │     (Your Controllers)      │   │
│  │   └──────────────┘    └──────────────────────────────┘   │
│  │                                                        │  │
│  │   Features:                                            │  │
│  │   ✓ HTTP/1.1, HTTP/2, HTTP/3                          │  │
│  │   ✓ HTTPS with TLS                                    │  │
│  │   ✓ WebSockets                                        │  │
│  │   ✓ Unix sockets                                      │  │
│  └────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────┘
```

---

## Kestrel Architecture

### Request Pipeline Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                      KESTREL ARCHITECTURE                        │
└─────────────────────────────────────────────────────────────────┘

         ┌─────────────────────────────────────────┐
         │            NETWORK LAYER                │
         │  (TCP/IP Socket, Named Pipes, Unix)     │
         └───────────────────┬─────────────────────┘
                             │
                             ▼
         ┌─────────────────────────────────────────┐
         │           CONNECTION LAYER              │
         │  - Connection Management                │
         │  - TLS/SSL Handshake                    │
         │  - Keep-Alive Management                │
         └───────────────────┬─────────────────────┘
                             │
                             ▼
         ┌─────────────────────────────────────────┐
         │            HTTP LAYER                   │
         │  - HTTP/1.1, HTTP/2, HTTP/3 Parsing     │
         │  - Request/Response Handling            │
         │  - Header Processing                    │
         └───────────────────┬─────────────────────┘
                             │
                             ▼
         ┌─────────────────────────────────────────┐
         │         APPLICATION LAYER               │
         │  - Middleware Pipeline                  │
         │  - Controllers/Endpoints                │
         │  - Your Application Code                │
         └─────────────────────────────────────────┘
```

### Internal Components

| Component | Responsibility |
|-----------|---------------|
| **Transport Layer** | Handles network I/O (sockets) |
| **Connection Manager** | Manages active connections |
| **HTTP Parser** | Parses HTTP requests |
| **Response Writer** | Formats and sends responses |

---

## Kestrel Configuration

### Basic Configuration in Program.cs

```csharp
// File: Program.cs

var builder = WebApplication.CreateBuilder(args);

// Line 1: Configure Kestrel options
builder.WebHost.ConfigureKestrel(serverOptions =>
{
    // Line 2: Set maximum request body size (default: 30MB)
    // null = unlimited
    serverOptions.Limits.MaxRequestBodySize = 10 * 1024 * 1024; // 10 MB
    
    // Line 3: Set maximum concurrent connections
    // null = unlimited (use with caution)
    serverOptions.Limits.MaxConcurrentConnections = 100;
    
    // Line 4: Set maximum concurrent upgraded connections (WebSockets)
    serverOptions.Limits.MaxConcurrentUpgradedConnections = 100;
    
    // Line 5: Set keep-alive timeout
    serverOptions.Limits.KeepAliveTimeout = TimeSpan.FromMinutes(2);
    
    // Line 6: Set request headers timeout
    serverOptions.Limits.RequestHeadersTimeout = TimeSpan.FromSeconds(30);
});

var app = builder.Build();
app.MapGet("/", () => "Hello from Kestrel!");
app.Run();
```

### Configuration via appsettings.json

```json
{
  "Kestrel": {
    "Limits": {
      "MaxConcurrentConnections": 100,
      "MaxConcurrentUpgradedConnections": 100,
      "MaxRequestBodySize": 10485760,
      "KeepAliveTimeout": "00:02:00",
      "RequestHeadersTimeout": "00:00:30"
    },
    "Endpoints": {
      "Http": {
        "Url": "http://localhost:5000"
      },
      "Https": {
        "Url": "https://localhost:5001"
      }
    }
  }
}
```

### Loading Configuration from appsettings.json

```csharp
// File: Program.cs

var builder = WebApplication.CreateBuilder(args);

// Line 1: Configure Kestrel from configuration
builder.WebHost.ConfigureKestrel((context, serverOptions) =>
{
    // Line 2: Load Kestrel configuration from appsettings.json
    serverOptions.Configure(context.Configuration.GetSection("Kestrel"));
});

var app = builder.Build();
app.Run();
```

---

## Port Configuration

### Method 1: launchSettings.json (Development)

```json
{
  "profiles": {
    "MyApp": {
      "commandName": "Project",
      "dotnetRunMessages": true,
      "launchBrowser": true,
      "applicationUrl": "https://localhost:7001;http://localhost:5001",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    }
  }
}
```

### Method 2: Code Configuration

```csharp
// File: Program.cs

var builder = WebApplication.CreateBuilder(args);

// Line 1: Configure specific endpoints
builder.WebHost.ConfigureKestrel(serverOptions =>
{
    // Line 2: Listen on specific IP and port for HTTP
    serverOptions.Listen(System.Net.IPAddress.Any, 5000);
    
    // Line 3: Listen on localhost only (more secure for dev)
    serverOptions.ListenLocalhost(5001);
    
    // Line 4: Listen on all interfaces with HTTPS
    serverOptions.ListenAnyIP(5002, listenOptions =>
    {
        listenOptions.UseHttps();
    });
});

var app = builder.Build();
app.Run();
```

### Method 3: Environment Variables

```bash
# Windows Command Prompt
set ASPNETCORE_URLS=http://localhost:5000;https://localhost:5001
dotnet run

# Windows PowerShell
$env:ASPNETCORE_URLS="http://localhost:5000;https://localhost:5001"
dotnet run

# Linux/macOS
export ASPNETCORE_URLS="http://localhost:5000;https://localhost:5001"
dotnet run
```

### Method 4: Command Line Arguments

```bash
# Specify URLs via command line
dotnet run --urls "http://localhost:5000;https://localhost:5001"
```

### Port Configuration Priority

```
┌─────────────────────────────────────────────────────────────┐
│                  URL CONFIGURATION PRIORITY                  │
│                    (Highest to Lowest)                       │
├─────────────────────────────────────────────────────────────┤
│  1. Command line --urls argument                            │
│                         ▼                                    │
│  2. ASPNETCORE_URLS environment variable                    │
│                         ▼                                    │
│  3. Kestrel code configuration                              │
│                         ▼                                    │
│  4. appsettings.json                                         │
│                         ▼                                    │
│  5. launchSettings.json (Development only)                  │
│                         ▼                                    │
│  6. Default: http://localhost:5000                          │
└─────────────────────────────────────────────────────────────┘
```

---

## HTTPS Configuration

### Using Development Certificate

```csharp
// File: Program.cs

var builder = WebApplication.CreateBuilder(args);

// Line 1: Configure HTTPS with development certificate
builder.WebHost.ConfigureKestrel(serverOptions =>
{
    // Line 2: HTTP endpoint
    serverOptions.ListenLocalhost(5000);
    
    // Line 3: HTTPS endpoint with development certificate
    serverOptions.ListenLocalhost(5001, listenOptions =>
    {
        // UseHttps() uses the development certificate automatically
        listenOptions.UseHttps();
    });
});

var app = builder.Build();

// Line 4: Redirect HTTP to HTTPS
app.UseHttpsRedirection();

app.MapGet("/", () => "Secure Hello!");
app.Run();
```

### Using Custom Certificate

```csharp
// File: Program.cs

var builder = WebApplication.CreateBuilder(args);

builder.WebHost.ConfigureKestrel(serverOptions =>
{
    serverOptions.ListenLocalhost(5001, listenOptions =>
    {
        // Line 1: Use certificate file with password
        listenOptions.UseHttps("certificate.pfx", "password123");
    });
});

var app = builder.Build();
app.Run();
```

### Certificate Configuration in appsettings.json

```json
{
  "Kestrel": {
    "Endpoints": {
      "HttpsFromPem": {
        "Url": "https://localhost:5001",
        "Certificate": {
          "Path": "/path/to/certificate.crt",
          "KeyPath": "/path/to/certificate.key"
        }
      },
      "HttpsFromPfx": {
        "Url": "https://localhost:5002",
        "Certificate": {
          "Path": "/path/to/certificate.pfx",
          "Password": "your-password"
        }
      }
    }
  }
}
```

### Trust Development Certificate (First-Time Setup)

```bash
# Trust the development certificate
dotnet dev-certs https --trust

# Clean and regenerate if needed
dotnet dev-certs https --clean
dotnet dev-certs https --trust
```

---

## Kestrel with Reverse Proxy

### Why Use Reverse Proxy?

```
┌───────────────────────────────────────────────────────────────┐
│                   REVERSE PROXY BENEFITS                       │
├───────────────────────────────────────────────────────────────┤
│  ✓ SSL/TLS Termination    - Handle HTTPS at proxy level      │
│  ✓ Load Balancing         - Distribute traffic               │
│  ✓ Caching                - Static content caching           │
│  ✓ Request Filtering      - Block malicious requests         │
│  ✓ IP Allowlisting        - Control access                   │
│  ✓ Rate Limiting          - Prevent DDoS attacks             │
│  ✓ Compression            - Gzip/Brotli compression          │
│  ✓ URL Rewriting          - Clean URLs                       │
└───────────────────────────────────────────────────────────────┘
```

### Architecture with Reverse Proxy

```
┌─────────────────────────────────────────────────────────────────┐
│                      PRODUCTION SETUP                            │
└─────────────────────────────────────────────────────────────────┘

    Internet
        │
        ▼
┌──────────────────────┐
│    REVERSE PROXY     │
│  (IIS / Nginx /      │
│   Apache)            │
│                      │
│  - SSL Termination   │
│  - Load Balancing    │
│  - Static Files      │
│  - Security          │
└──────────┬───────────┘
           │ HTTP (internal)
           ▼
┌──────────────────────┐     ┌──────────────────────┐
│      KESTREL         │     │      KESTREL         │
│    (Server 1)        │     │    (Server 2)        │
│                      │     │                      │
│  - App Logic         │     │  - App Logic         │
│  - Controllers       │     │  - Controllers       │
└──────────────────────┘     └──────────────────────┘
```

### Configure for Reverse Proxy

```csharp
// File: Program.cs

var builder = WebApplication.CreateBuilder(args);

// Line 1: Add forwarded headers service
builder.Services.Configure<ForwardedHeadersOptions>(options =>
{
    // Line 2: Forward X-Forwarded-For and X-Forwarded-Proto headers
    options.ForwardedHeaders = ForwardedHeaders.XForwardedFor | 
                               ForwardedHeaders.XForwardedProto;
});

var app = builder.Build();

// Line 3: Use forwarded headers middleware (MUST be first)
app.UseForwardedHeaders();

// Line 4: Rest of your middleware
app.UseHttpsRedirection();
app.UseRouting();
app.UseAuthorization();

app.MapControllers();
app.Run();
```

### Nginx Configuration Example

```nginx
# File: /etc/nginx/sites-available/myapp

upstream kestrel {
    server 127.0.0.1:5000;
}

server {
    listen 80;
    server_name myapp.com;
    
    # Redirect HTTP to HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl;
    server_name myapp.com;
    
    ssl_certificate /etc/ssl/certs/myapp.crt;
    ssl_certificate_key /etc/ssl/private/myapp.key;
    
    location / {
        proxy_pass http://kestrel;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection keep-alive;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```

---

## Performance Features

### HTTP/2 Support

```csharp
// File: Program.cs

var builder = WebApplication.CreateBuilder(args);

builder.WebHost.ConfigureKestrel(serverOptions =>
{
    serverOptions.ListenLocalhost(5001, listenOptions =>
    {
        listenOptions.UseHttps();
        // Line 1: HTTP/2 is enabled by default with HTTPS
        // HTTP/2 provides:
        // - Multiplexing (multiple requests over single connection)
        // - Header compression
        // - Server push
        listenOptions.Protocols = HttpProtocols.Http1AndHttp2;
    });
});

var app = builder.Build();
app.Run();
```

### Connection Limits

```csharp
builder.WebHost.ConfigureKestrel(serverOptions =>
{
    // Line 1: Limit concurrent connections to prevent resource exhaustion
    serverOptions.Limits.MaxConcurrentConnections = 100;
    
    // Line 2: Limit WebSocket connections separately
    serverOptions.Limits.MaxConcurrentUpgradedConnections = 100;
    
    // Line 3: Limit request body size to prevent large uploads
    serverOptions.Limits.MaxRequestBodySize = 10 * 1024 * 1024; // 10 MB
    
    // Line 4: Limit request line length
    serverOptions.Limits.MaxRequestLineSize = 8 * 1024; // 8 KB
    
    // Line 5: Limit request header size
    serverOptions.Limits.MaxRequestHeadersTotalSize = 32 * 1024; // 32 KB
});
```

### Performance Comparison

```
Kestrel vs Other Servers (Requests/Second)
═════════════════════════════════════════════════════════

Kestrel         ████████████████████████████████████  ~2,000,000
nginx           ████████████████████████████          ~1,500,000
Node.js         ████████████████████                  ~1,000,000
IIS             ████████████████                      ~800,000
Apache          ████████████                          ~600,000

Note: Numbers are approximate and depend on hardware/configuration
```

---

## Code Examples

### Complete Kestrel Configuration Example

```csharp
// File: Program.cs

using Microsoft.AspNetCore.Server.Kestrel.Core;
using System.Net;

var builder = WebApplication.CreateBuilder(args);

// ═══════════════════════════════════════════════════════════════
// KESTREL CONFIGURATION
// ═══════════════════════════════════════════════════════════════

builder.WebHost.ConfigureKestrel((context, serverOptions) =>
{
    // ─────────────────────────────────────────────────────────────
    // SECTION 1: GENERAL LIMITS
    // ─────────────────────────────────────────────────────────────
    
    // Line 1: Maximum request body size (10 MB)
    // Prevents large file uploads from consuming resources
    serverOptions.Limits.MaxRequestBodySize = 10 * 1024 * 1024;
    
    // Line 2: Maximum concurrent connections
    // Null = unlimited (be careful in production)
    serverOptions.Limits.MaxConcurrentConnections = 100;
    
    // Line 3: Keep-alive timeout
    // How long to keep connection open between requests
    serverOptions.Limits.KeepAliveTimeout = TimeSpan.FromMinutes(2);
    
    // Line 4: Request headers timeout
    // Time allowed to receive request headers
    serverOptions.Limits.RequestHeadersTimeout = TimeSpan.FromSeconds(30);
    
    // ─────────────────────────────────────────────────────────────
    // SECTION 2: ENDPOINTS
    // ─────────────────────────────────────────────────────────────
    
    // Line 5: HTTP endpoint on localhost only (development)
    serverOptions.Listen(IPAddress.Loopback, 5000);
    
    // Line 6: HTTPS endpoint with configuration
    serverOptions.Listen(IPAddress.Loopback, 5001, listenOptions =>
    {
        // Use development certificate
        listenOptions.UseHttps();
        
        // Enable HTTP/1.1 and HTTP/2
        listenOptions.Protocols = HttpProtocols.Http1AndHttp2;
    });
    
    // Line 7: (Production) Listen on all network interfaces
    // serverOptions.Listen(IPAddress.Any, 80);
    
    // ─────────────────────────────────────────────────────────────
    // SECTION 3: HTTP/2 SETTINGS
    // ─────────────────────────────────────────────────────────────
    
    // Line 8: Maximum streams per connection (HTTP/2)
    serverOptions.Limits.Http2.MaxStreamsPerConnection = 100;
    
    // Line 9: Initial connection window size
    serverOptions.Limits.Http2.InitialConnectionWindowSize = 128 * 1024;
    
    // Line 10: Initial stream window size
    serverOptions.Limits.Http2.InitialStreamWindowSize = 96 * 1024;
});

// ═══════════════════════════════════════════════════════════════
// BUILD AND RUN
// ═══════════════════════════════════════════════════════════════

var app = builder.Build();

// Middleware
app.UseHttpsRedirection();

// Simple test endpoint
app.MapGet("/", () => "Hello from Kestrel!");

// Server info endpoint
app.MapGet("/server-info", (HttpContext context) =>
{
    return Results.Ok(new
    {
        Protocol = context.Request.Protocol,
        Scheme = context.Request.Scheme,
        Host = context.Request.Host.ToString(),
        Port = context.Connection.LocalPort,
        IsHttps = context.Request.IsHttps
    });
});

app.Run();
```

### Running with Kestrel via CLI

```bash
# Method 1: Default run (uses launchSettings.json in Development)
dotnet run

# Method 2: Specify URLs directly
dotnet run --urls "http://localhost:5000;https://localhost:5001"

# Method 3: Run in Release mode
dotnet run --configuration Release

# Method 4: Run published application
dotnet publish -c Release
cd bin/Release/net8.0/publish
dotnet MyApp.dll --urls "http://localhost:5000"
```

---

## Best Practices

### 1. Always Use Reverse Proxy in Production

```
✅ DO: Use IIS/Nginx in front of Kestrel for public-facing apps
❌ DON'T: Expose Kestrel directly to the internet
```

### 2. Configure Appropriate Limits

```csharp
// Set limits based on your application needs
serverOptions.Limits.MaxConcurrentConnections = 100;
serverOptions.Limits.MaxRequestBodySize = 10 * 1024 * 1024;
```

### 3. Use HTTPS

```csharp
// Always use HTTPS in production
serverOptions.ListenAnyIP(443, listenOptions =>
{
    listenOptions.UseHttps("certificate.pfx", "password");
});
```

### 4. Enable HTTP/2

```csharp
// HTTP/2 provides better performance
listenOptions.Protocols = HttpProtocols.Http1AndHttp2;
```

### 5. Handle Forwarded Headers with Proxy

```csharp
// Required when behind reverse proxy
app.UseForwardedHeaders();
```

---

## Common Pitfalls

### 1. Port Already in Use

```
Error: Unable to bind to http://localhost:5000

Solution: 
1. Check if another process is using the port
2. Change the port in configuration
3. Stop the other process
```

### 2. HTTPS Certificate Issues

```
Error: The certificate was not found

Solution:
1. Run: dotnet dev-certs https --trust
2. Or provide a valid certificate file
```

### 3. Missing Forwarded Headers

```
Problem: Wrong client IP address in logs (shows proxy IP)

Solution: Configure ForwardedHeadersOptions and use UseForwardedHeaders()
```

---

## Interview Questions

### Q1: What is Kestrel?
**Answer:** Kestrel is a cross-platform, high-performance web server for ASP.NET Core. It's built on top of libuv (earlier versions) and now uses managed sockets. It can run standalone or behind a reverse proxy like IIS, Nginx, or Apache.

### Q2: How do you change the port number for Kestrel Server?
**Answer:** You can change the port by:
1. Modifying `launchSettings.json` (applicationUrl property)
2. Setting `ASPNETCORE_URLS` environment variable
3. Using `--urls` command line argument
4. Configuring in code: `serverOptions.Listen(IPAddress.Any, 60222)`
5. Configuring in `appsettings.json` under Kestrel section

### Q3: What happens when we run ASP.NET Core application using .NET Core CLI?
**Answer:** When running with `dotnet run` or `dotnet` command, the .NET Core runtime uses Kestrel as the web server. The application runs in the dotnet.exe process with Kestrel handling HTTP requests.

### Q4: Should Kestrel be exposed directly to the internet?
**Answer:** No. For production public-facing applications, Kestrel should be used behind a reverse proxy (IIS, Nginx, Apache) that provides:
- SSL/TLS termination
- Load balancing
- Request filtering
- Additional security features

### Q5: What is the default port Kestrel listens on?
**Answer:** By default, Kestrel listens on:
- HTTP: `http://localhost:5000`
- HTTPS: `https://localhost:5001` (when configured)

### Q6: How does Kestrel achieve high performance?
**Answer:**
1. Asynchronous I/O throughout
2. Efficient memory management with spans
3. HTTP/2 multiplexing support
4. Connection pooling
5. Minimal overhead - only what's needed

---

## Quick Reference

### Configuration Methods

```csharp
// Method 1: Code configuration
builder.WebHost.ConfigureKestrel(serverOptions => { ... });

// Method 2: appsettings.json
"Kestrel": { "Endpoints": { ... } }

// Method 3: Environment variable
ASPNETCORE_URLS=http://localhost:5000

// Method 4: Command line
dotnet run --urls "http://localhost:5000"
```

### Common Limits

| Limit | Default | Description |
|-------|---------|-------------|
| MaxConcurrentConnections | Unlimited | Max simultaneous connections |
| MaxRequestBodySize | 30 MB | Max request body |
| KeepAliveTimeout | 2 minutes | Connection keep-alive |
| RequestHeadersTimeout | 30 seconds | Header receive timeout |

### Listen Methods

```csharp
// Listen on localhost only
serverOptions.ListenLocalhost(5000);

// Listen on all interfaces
serverOptions.ListenAnyIP(5000);

// Listen on specific IP
serverOptions.Listen(IPAddress.Parse("192.168.1.100"), 5000);

// Listen with HTTPS
serverOptions.ListenLocalhost(5001, o => o.UseHttps());
```
