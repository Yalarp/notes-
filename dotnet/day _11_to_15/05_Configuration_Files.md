# Configuration Files in ASP.NET Core

## Table of Contents
1. [Introduction](#introduction)
2. [launchSettings.json](#launchsettingsjson)
3. [appsettings.json](#appsettingsjson)
4. [Startup Configuration](#startup-configuration)
5. [Environment-Based Configuration](#environment-based-configuration)
6. [Reading Configuration Values](#reading-configuration-values)
7. [Connection Strings](#connection-strings)
8. [Configuration Providers](#configuration-providers)
9. [Code Examples](#code-examples)
10. [Best Practices](#best-practices)
11. [Interview Questions](#interview-questions)

---

## Introduction

### Real-World Analogy
Configuration files are like **restaurant menus and procedures**:
- **launchSettings.json** = Opening procedures (what time to open, which door to use)
- **appsettings.json** = The main recipe book (how to make each dish)
- **appsettings.Development.json** = Training recipes for new chefs (development settings)
- **appsettings.Production.json** = Actual recipes used for customers (production settings)

```
┌─────────────────────────────────────────────────────────────────┐
│                  CONFIGURATION HIERARCHY                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Development Only          All Environments                     │
│   ┌───────────────┐         ┌─────────────────┐                 │
│   │launchSettings │         │  appsettings    │                 │
│   │   .json       │         │    .json        │                 │
│   │               │         │                 │                 │
│   │ - Profiles    │         │ - App settings  │                 │
│   │ - URLs        │         │ - Conn strings  │                 │
│   │ - Environment │         │ - Logging       │                 │
│   └───────────────┘         └─────────────────┘                 │
│                                     │                            │
│                                     ▼                            │
│                             ┌─────────────────┐                 │
│                             │appsettings.     │                 │
│                             │{Environment}    │                 │
│                             │  .json          │                 │
│                             │                 │                 │
│                             │ - Overrides     │                 │
│                             └─────────────────┘                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## launchSettings.json

### What is launchSettings.json?

`launchSettings.json` is located in the **Properties** folder and is used **only during development**. It configures how the application starts in Visual Studio or with `dotnet run`.

### File Location

```
MyApp/
├── Properties/
│   └── launchSettings.json    ← This file
├── Controllers/
├── Views/
├── wwwroot/
├── appsettings.json
└── Program.cs
```

### Complete launchSettings.json with Explanation

```json
{
  // Line 1: Define IIS Settings (for IIS Express)
  "iisSettings": {
    // Line 2: Enable Windows Authentication
    "windowsAuthentication": false,
    
    // Line 3: Enable Anonymous Authentication
    "anonymousAuthentication": true,
    
    // Line 4: IIS Express specific settings
    "iisExpress": {
      // Line 5: HTTP port for IIS Express
      "applicationUrl": "http://localhost:52431",
      
      // Line 6: SSL port for HTTPS
      "sslPort": 44355
    }
  },
  
  // Line 7: Define Launch Profiles
  "profiles": {
    
    // ─────────────────────────────────────────────────────
    // PROFILE 1: HTTP Only (Kestrel)
    // ─────────────────────────────────────────────────────
    "http": {
      // Line 8: commandName "Project" means use Kestrel
      "commandName": "Project",
      
      // Line 9: Show dotnet run messages in console
      "dotnetRunMessages": true,
      
      // Line 10: Open browser automatically
      "launchBrowser": true,
      
      // Line 11: URL to use (HTTP only)
      "applicationUrl": "http://localhost:5000",
      
      // Line 12: Environment variables
      "environmentVariables": {
        // Line 13: Set to Development environment
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    },
    
    // ─────────────────────────────────────────────────────
    // PROFILE 2: HTTPS (Kestrel)
    // ─────────────────────────────────────────────────────
    "https": {
      "commandName": "Project",
      "dotnetRunMessages": true,
      "launchBrowser": true,
      
      // Line 14: Multiple URLs - HTTPS first, then HTTP
      "applicationUrl": "https://localhost:7001;http://localhost:5001",
      
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    },
    
    // ─────────────────────────────────────────────────────
    // PROFILE 3: IIS Express
    // ─────────────────────────────────────────────────────
    "IIS Express": {
      // Line 15: commandName "IISExpress" uses IIS Express
      "commandName": "IISExpress",
      
      "launchBrowser": true,
      
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    }
  }
}
```

### Key Properties Explained

| Property | Description | Values |
|----------|-------------|--------|
| `commandName` | How to run the app | `Project` (Kestrel), `IISExpress`, `IIS` |
| `applicationUrl` | URL(s) to listen on | `http://localhost:5000` |
| `launchBrowser` | Auto-open browser | `true` / `false` |
| `dotnetRunMessages` | Show CLI messages | `true` / `false` |
| `environmentVariables` | Environment vars | Object with key-value pairs |
| `sslPort` | HTTPS port for IIS | Port number |

### commandName Values

```
commandName: "Project"
────────────────────────────────────────
• Uses Kestrel web server
• Runs as dotnet.exe process
• Cross-platform compatible
• AspNetCoreHostingModel setting IGNORED

commandName: "IISExpress"
────────────────────────────────────────
• Uses IIS Express
• Windows only
• Respects AspNetCoreHostingModel setting
• Uses iisSettings configuration

commandName: "IIS"
────────────────────────────────────────
• Uses full IIS
• Production-like environment
• Requires IIS to be installed
• Uses AspNetCoreHostingModel setting
```

---

## appsettings.json

### What is appsettings.json?

`appsettings.json` is the **primary configuration file** for ASP.NET Core applications. It is used in **all environments** and contains application settings, connection strings, and other configuration data.

### File Location

```
MyApp/
├── Properties/
│   └── launchSettings.json
├── Controllers/
├── appsettings.json               ← Base settings (all environments)
├── appsettings.Development.json   ← Development overrides
├── appsettings.Production.json    ← Production overrides
└── Program.cs
```

### Complete appsettings.json with Explanation

```json
{
  // ═══════════════════════════════════════════════════════════════
  // SECTION 1: LOGGING CONFIGURATION
  // ═══════════════════════════════════════════════════════════════
  "Logging": {
    "LogLevel": {
      // Line 1: Default log level for all categories
      // Options: Trace, Debug, Information, Warning, Error, Critical, None
      "Default": "Information",
      
      // Line 2: Microsoft framework logs - show only warnings and above
      "Microsoft.AspNetCore": "Warning",
      
      // Line 3: Entity Framework logs - show only errors
      "Microsoft.EntityFrameworkCore": "Error"
    }
  },
  
  // ═══════════════════════════════════════════════════════════════
  // SECTION 2: CONNECTION STRINGS
  // ═══════════════════════════════════════════════════════════════
  "ConnectionStrings": {
    // Line 4: SQL Server connection string
    // Named "EmployeeDBConnection" - referenced in code
    "EmployeeDBConnection": "Server=(localdb)\\mssqllocaldb;Database=EmployeeDB;Trusted_Connection=True;MultipleActiveResultSets=true",
    
    // Line 5: Alternative connection string example
    "DefaultConnection": "Server=myserver;Database=mydb;User Id=user;Password=pass;"
  },
  
  // ═══════════════════════════════════════════════════════════════
  // SECTION 3: ALLOWED HOSTS
  // ═══════════════════════════════════════════════════════════════
  // Line 6: Which hosts can access this application
  // "*" means all hosts (be careful in production)
  "AllowedHosts": "*",
  
  // ═══════════════════════════════════════════════════════════════
  // SECTION 4: CUSTOM APPLICATION SETTINGS
  // ═══════════════════════════════════════════════════════════════
  "ApplicationSettings": {
    // Line 7: Custom application name
    "ApplicationName": "Employee Management System",
    
    // Line 8: Application version
    "Version": "1.0.0",
    
    // Line 9: Contact email
    "SupportEmail": "support@company.com",
    
    // Line 10: Feature flags
    "EnableNewFeature": true,
    
    // Line 11: Pagination settings
    "PageSize": 10
  },
  
  // ═══════════════════════════════════════════════════════════════
  // SECTION 5: JWT SETTINGS (for authentication)
  // ═══════════════════════════════════════════════════════════════
  "JwtSettings": {
    // Line 12: Secret key for JWT signing (keep secret in production!)
    "SecretKey": "YourSecretKeyHere-MustBe32CharsOrMore!",
    
    // Line 13: Token issuer
    "Issuer": "MyApp",
    
    // Line 14: Token audience
    "Audience": "MyAppUsers",
    
    // Line 15: Token expiration in minutes
    "ExpirationMinutes": 60
  },
  
  // ═══════════════════════════════════════════════════════════════
  // SECTION 6: KESTREL CONFIGURATION (optional)
  // ═══════════════════════════════════════════════════════════════
  "Kestrel": {
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

### Configuration Hierarchy

```
┌─────────────────────────────────────────────────────────────────┐
│              CONFIGURATION LOADING ORDER                         │
│               (Later sources override earlier)                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. appsettings.json                     (Base settings)        │
│                  ▼                                               │
│  2. appsettings.{Environment}.json       (Env overrides)        │
│                  ▼                                               │
│  3. User Secrets                         (Dev secrets)          │
│                  ▼                                               │
│  4. Environment Variables                (System vars)          │
│                  ▼                                               │
│  5. Command Line Arguments               (CLI args)             │
│                                                                  │
│  Example: If "PageSize" is in both appsettings.json (10)        │
│           and appsettings.Development.json (20),                 │
│           Development uses 20, Production uses 10               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Startup Configuration

### Modern Program.cs Configuration

```csharp
// File: Program.cs

// Line 1: Create WebApplicationBuilder
// This automatically loads configuration from multiple sources
var builder = WebApplication.CreateBuilder(args);

// ═══════════════════════════════════════════════════════════════
// CONFIGURATION IS AUTOMATICALLY LOADED FROM:
// 1. appsettings.json
// 2. appsettings.{Environment}.json
// 3. User Secrets (Development only)
// 4. Environment Variables
// 5. Command Line Arguments
// ═══════════════════════════════════════════════════════════════

// Line 2: Access configuration values
var connectionString = builder.Configuration.GetConnectionString("EmployeeDBConnection");
var appName = builder.Configuration["ApplicationSettings:ApplicationName"];

// Line 3: Register DbContext with connection string from configuration
builder.Services.AddDbContextPool<AppDbContext>(options =>
    options.UseSqlServer(connectionString));

// Line 4: Add MVC services
builder.Services.AddControllersWithViews();

// Line 5: Build the application
var app = builder.Build();

// Line 6: Access configuration in middleware
var pageSize = app.Configuration["ApplicationSettings:PageSize"];

// Line 7: Configure middleware pipeline
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}
else
{
    app.UseExceptionHandler("/Error");
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

### Older Startup.cs Pattern (Pre-.NET 6)

```csharp
// File: Startup.cs (for reference - older pattern)

public class Startup
{
    // Line 1: Configuration property
    public IConfiguration Configuration { get; }
    
    // Line 2: Constructor receives IConfiguration from DI
    public Startup(IConfiguration configuration)
    {
        Configuration = configuration;
    }
    
    // Line 3: ConfigureServices - Register services
    public void ConfigureServices(IServiceCollection services)
    {
        // Read connection string
        var connStr = Configuration.GetConnectionString("EmployeeDBConnection");
        
        services.AddDbContext<AppDbContext>(options =>
            options.UseSqlServer(connStr));
        
        services.AddControllersWithViews();
    }
    
    // Line 4: Configure - Set up middleware pipeline
    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        if (env.IsDevelopment())
        {
            app.UseDeveloperExceptionPage();
        }
        
        app.UseRouting();
        app.UseAuthorization();
        
        app.UseEndpoints(endpoints =>
        {
            endpoints.MapControllerRoute(
                name: "default",
                pattern: "{controller=Home}/{action=Index}/{id?}");
        });
    }
}
```

---

## Environment-Based Configuration

### Understanding Environments

ASP.NET Core supports three default environments:

| Environment | Use Case | Configuration File |
|-------------|----------|-------------------|
| `Development` | Local development | `appsettings.Development.json` |
| `Staging` | Pre-production testing | `appsettings.Staging.json` |
| `Production` | Live deployment | `appsettings.Production.json` |

### Setting the Environment

```bash
# Method 1: Environment Variable (Windows CMD)
set ASPNETCORE_ENVIRONMENT=Development
dotnet run

# Method 2: Environment Variable (PowerShell)
$env:ASPNETCORE_ENVIRONMENT = "Development"
dotnet run

# Method 3: Environment Variable (Linux/macOS)
export ASPNETCORE_ENVIRONMENT=Development
dotnet run

# Method 4: In launchSettings.json
"environmentVariables": {
    "ASPNETCORE_ENVIRONMENT": "Development"
}
```

### Environment-Specific Configuration Files

```json
// File: appsettings.json (Base/Shared settings)
{
  "Logging": {
    "LogLevel": {
      "Default": "Information"
    }
  },
  "ConnectionStrings": {
    "DefaultConnection": "Server=prodserver;Database=proddb;"
  },
  "ApplicationSettings": {
    "ShowDetailedErrors": false
  }
}
```

```json
// File: appsettings.Development.json (Development overrides)
{
  "Logging": {
    "LogLevel": {
      // More verbose logging in development
      "Default": "Debug",
      "Microsoft.EntityFrameworkCore": "Information"
    }
  },
  "ConnectionStrings": {
    // Local database for development
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=DevDB;"
  },
  "ApplicationSettings": {
    // Show detailed errors in development
    "ShowDetailedErrors": true
  }
}
```

### Checking Environment in Code

```csharp
// File: Program.cs

var app = builder.Build();

// Line 1: Check if running in Development
if (app.Environment.IsDevelopment())
{
    // Show detailed error pages
    app.UseDeveloperExceptionPage();
    Console.WriteLine("Running in DEVELOPMENT mode");
}
// Line 2: Check if running in Staging
else if (app.Environment.IsStaging())
{
    app.UseExceptionHandler("/Error");
    Console.WriteLine("Running in STAGING mode");
}
// Line 3: Check if running in Production
else if (app.Environment.IsProduction())
{
    app.UseExceptionHandler("/Error");
    app.UseHsts();
    Console.WriteLine("Running in PRODUCTION mode");
}
// Line 4: Check for custom environment
else if (app.Environment.IsEnvironment("Testing"))
{
    Console.WriteLine("Running in TESTING mode");
}
```

---

## Reading Configuration Values

### Method 1: Direct Access with Indexer

```csharp
// File: Program.cs or any class with IConfiguration

// Line 1: Simple value
string appName = builder.Configuration["ApplicationSettings:ApplicationName"];
// Result: "Employee Management System"

// Line 2: Nested value using colon separator
string logLevel = builder.Configuration["Logging:LogLevel:Default"];
// Result: "Information"

// Line 3: Connection string (special helper)
string connStr = builder.Configuration.GetConnectionString("DefaultConnection");
// Equivalent to: Configuration["ConnectionStrings:DefaultConnection"]
```

### Method 2: GetSection and GetValue

```csharp
// Line 1: Get a section
IConfigurationSection appSection = builder.Configuration.GetSection("ApplicationSettings");

// Line 2: Get typed value with default
int pageSize = builder.Configuration.GetValue<int>("ApplicationSettings:PageSize", 10);
// Returns 10 if not found

// Line 3: Get boolean value
bool enableFeature = builder.Configuration.GetValue<bool>("ApplicationSettings:EnableNewFeature");
```

### Method 3: Options Pattern (Recommended)

```csharp
// Step 1: Create a strongly-typed options class
// File: Models/ApplicationSettings.cs

public class ApplicationSettings
{
    // Line 1: Properties match JSON structure
    public string ApplicationName { get; set; } = string.Empty;
    public string Version { get; set; } = string.Empty;
    public string SupportEmail { get; set; } = string.Empty;
    public bool EnableNewFeature { get; set; }
    public int PageSize { get; set; }
}
```

```csharp
// Step 2: Configure in Program.cs
// File: Program.cs

var builder = WebApplication.CreateBuilder(args);

// Line 1: Bind configuration section to class
builder.Services.Configure<ApplicationSettings>(
    builder.Configuration.GetSection("ApplicationSettings"));

// Line 2: Also register as singleton for direct injection
builder.Services.AddSingleton(
    builder.Configuration.GetSection("ApplicationSettings").Get<ApplicationSettings>()!);
```

```csharp
// Step 3: Use in Controller
// File: Controllers/HomeController.cs

public class HomeController : Controller
{
    private readonly ApplicationSettings _settings;

    // Line 1: Inject IOptions<T>
    public HomeController(IOptions<ApplicationSettings> options)
    {
        _settings = options.Value;
    }

    public IActionResult Index()
    {
        // Line 2: Use strongly-typed settings
        ViewBag.AppName = _settings.ApplicationName;
        ViewBag.Version = _settings.Version;
        return View();
    }
}
```

### Method 4: Configuration in Controllers (Direct IConfiguration)

```csharp
// File: Controllers/EmployeeController.cs

public class EmployeeController : Controller
{
    private readonly IConfiguration _configuration;

    // Line 1: Inject IConfiguration
    public EmployeeController(IConfiguration configuration)
    {
        _configuration = configuration;
    }

    public IActionResult Settings()
    {
        // Line 2: Read values directly
        var appName = _configuration["ApplicationSettings:ApplicationName"];
        var pageSize = _configuration.GetValue<int>("ApplicationSettings:PageSize");
        
        return Json(new { appName, pageSize });
    }
}
```

---

## Connection Strings

### Defining Connection Strings

```json
// File: appsettings.json
{
  "ConnectionStrings": {
    // SQL Server with Windows Authentication
    "EmployeeDBConnection": "Server=(localdb)\\mssqllocaldb;Database=EmployeeDB;Trusted_Connection=True;MultipleActiveResultSets=true",
    
    // SQL Server with SQL Authentication
    "SqlAuthConnection": "Server=myserver;Database=mydb;User Id=myuser;Password=mypassword;",
    
    // With additional options
    "ProductionConnection": "Server=prod-server;Database=ProdDB;User Id=app;Password=secret;Encrypt=true;TrustServerCertificate=false;Connection Timeout=30;"
  }
}
```

### Reading Connection Strings

```csharp
// File: Program.cs

var builder = WebApplication.CreateBuilder(args);

// Method 1: GetConnectionString helper method
string? connStr = builder.Configuration.GetConnectionString("EmployeeDBConnection");

// Method 2: Direct access
string? connStr2 = builder.Configuration["ConnectionStrings:EmployeeDBConnection"];

// Method 3: Use in DbContext registration
builder.Services.AddDbContextPool<AppDbContext>(options =>
{
    options.UseSqlServer(
        builder.Configuration.GetConnectionString("EmployeeDBConnection")
    );
});
```

### Secure Connection Strings

```csharp
// For Development: Use User Secrets
// dotnet user-secrets init
// dotnet user-secrets set "ConnectionStrings:DefaultConnection" "your-connection-string"

// For Production: Use Environment Variables
// Set environment variable: ConnectionStrings__DefaultConnection=your-connection-string
// Note: Use double underscore (__) for nested keys
```

---

## Configuration Providers

### Default Configuration Sources

```csharp
// These are automatically configured by CreateBuilder()

var builder = WebApplication.CreateBuilder(args);

// Equivalent to:
/*
builder.Configuration
    .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
    .AddJsonFile($"appsettings.{environment}.json", optional: true, reloadOnChange: true)
    .AddUserSecrets<Program>(optional: true)  // Development only
    .AddEnvironmentVariables()
    .AddCommandLine(args);
*/
```

### Custom Configuration Provider

```csharp
// Add custom configuration sources
var builder = WebApplication.CreateBuilder(args);

// Line 1: Add XML configuration file
builder.Configuration.AddXmlFile("customsettings.xml", optional: true);

// Line 2: Add INI file
builder.Configuration.AddIniFile("settings.ini", optional: true);

// Line 3: Add in-memory configuration
builder.Configuration.AddInMemoryCollection(new Dictionary<string, string>
{
    ["ApplicationSettings:MaxItems"] = "100",
    ["ApplicationSettings:MinItems"] = "1"
});

// Line 4: Add custom environment variable prefix
builder.Configuration.AddEnvironmentVariables(prefix: "MYAPP_");
```

### Configuration Provider Priority

```
┌─────────────────────────────────────────────────────────────────┐
│              CONFIGURATION PRIORITY                              │
│       (Later sources override earlier - Last wins)              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  LOWEST PRIORITY (Loaded First)                                 │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  1. appsettings.json (base settings)                    │    │
│  └─────────────────────────────────────────────────────────┘    │
│                         ▼                                        │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  2. appsettings.{Environment}.json                      │    │
│  └─────────────────────────────────────────────────────────┘    │
│                         ▼                                        │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  3. User Secrets (Development only)                     │    │
│  └─────────────────────────────────────────────────────────┘    │
│                         ▼                                        │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  4. Environment Variables                               │    │
│  └─────────────────────────────────────────────────────────┘    │
│                         ▼                                        │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  5. Command Line Arguments                              │    │
│  └─────────────────────────────────────────────────────────┘    │
│  HIGHEST PRIORITY (Loaded Last - Wins)                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Best Practices

### 1. Never Store Secrets in appsettings.json

```csharp
// ❌ BAD: Secrets in appsettings.json (gets committed to source control)
{
  "ConnectionStrings": {
    "Default": "Server=prod;User=admin;Password=secret123;"  // NEVER DO THIS!
  }
}

// ✅ GOOD: Use User Secrets for development
// dotnet user-secrets set "ConnectionStrings:Default" "connection-string"

// ✅ GOOD: Use Environment Variables for production
// Set: ConnectionStrings__Default=connection-string
```

### 2. Use Strongly-Typed Configuration

```csharp
// ✅ GOOD: Strongly typed with Options pattern
builder.Services.Configure<JwtSettings>(
    builder.Configuration.GetSection("JwtSettings"));

// ❌ BAD: Magic strings everywhere
var key = _configuration["JwtSettings:SecretKey"];
```

### 3. Validate Configuration at Startup

```csharp
// Validate required configuration
builder.Services.AddOptions<JwtSettings>()
    .Bind(builder.Configuration.GetSection("JwtSettings"))
    .ValidateDataAnnotations()  // Validate using data annotations
    .ValidateOnStart();         // Fail fast if invalid
```

### 4. Use Environment-Specific Files

```
appsettings.json              → Shared settings
appsettings.Development.json  → Development overrides
appsettings.Staging.json      → Staging overrides
appsettings.Production.json   → Production overrides
```

---

## Interview Questions

### Q1: Where is launchSettings.json located?
**Answer:** launchSettings.json is located in the **Properties** folder of your application.

### Q2: What is the difference between appsettings.json and launchSettings.json?
**Answer:**
| Feature | launchSettings.json | appsettings.json |
|---------|-------------------|------------------|
| **Location** | Properties folder | Project root |
| **Usage** | Development only | All environments |
| **Purpose** | Launch profiles, URLs | App configuration |
| **Deployed** | No | Yes |
| **Contains** | Environment vars, ports | Settings, connection strings |

### Q3: How do you read a connection string from configuration?
**Answer:**
```csharp
// Using helper method (recommended)
var connStr = builder.Configuration.GetConnectionString("DefaultConnection");

// Using indexer
var connStr = builder.Configuration["ConnectionStrings:DefaultConnection"];
```

### Q4: What is the Options pattern?
**Answer:** The Options pattern is a way to access configuration using strongly-typed classes instead of magic strings. You create a class matching the configuration structure, register it with `services.Configure<T>()`, and inject `IOptions<T>` where needed.

### Q5: What is the priority order of configuration sources?
**Answer:** Later sources override earlier ones:
1. appsettings.json (lowest priority)
2. appsettings.{Environment}.json
3. User Secrets
4. Environment Variables
5. Command Line Arguments (highest priority)

### Q6: What environment variable sets the hosting environment?
**Answer:** `ASPNETCORE_ENVIRONMENT` sets the environment. Common values: Development, Staging, Production.

---

## Quick Reference

### Configuration Access Cheat Sheet

```csharp
// Simple value
var value = Configuration["Section:Key"];

// Typed value with default
var count = Configuration.GetValue<int>("Section:Count", 10);

// Connection string
var connStr = Configuration.GetConnectionString("Name");

// Bind to object
var settings = Configuration.GetSection("Section").Get<MyClass>();

// Options pattern
services.Configure<MyOptions>(Configuration.GetSection("Section"));
```

### Environment Detection

```csharp
app.Environment.IsDevelopment()   // true if Development
app.Environment.IsStaging()       // true if Staging
app.Environment.IsProduction()    // true if Production
app.Environment.IsEnvironment("Custom")  // Check custom
```

### File Locations

```
Project/
├── Properties/
│   └── launchSettings.json     ← Development launch settings
├── appsettings.json            ← Base configuration
├── appsettings.Development.json ← Dev overrides
├── appsettings.Production.json  ← Prod overrides
└── Program.cs
```
