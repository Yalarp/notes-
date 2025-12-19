# Static Files and Default Pages in ASP.NET Core

## Table of Contents
1. [Introduction](#introduction)
2. [wwwroot Folder](#wwwroot-folder)
3. [Static Files Middleware](#static-files-middleware)
4. [Default Documents](#default-documents)
5. [Custom Static File Paths](#custom-static-file-paths)
6. [File Server Middleware](#file-server-middleware)
7. [Security Considerations](#security-considerations)
8. [Code Examples](#code-examples)
9. [Best Practices](#best-practices)
10. [Interview Questions](#interview-questions)

---

## Introduction

### Real-World Analogy
Static files are like **ready-to-serve items in a cafeteria**:

```
Customer Request: "Give me a sandwich"
                        │
                        ▼
┌───────────────────────────────────────────────────────────┐
│                  CAFETERIA COUNTER                         │
│                                                            │
│   ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐  │
│   │Sandwich │   │  Chips  │   │  Drinks │   │ Cookies │  │
│   │ (CSS)   │   │  (JS)   │   │ (Images)│   │ (Fonts) │  │
│   └─────────┘   └─────────┘   └─────────┘   └─────────┘  │
│                                                            │
│   No cooking needed - just grab and serve!                │
│   (No processing - direct file serving)                   │
└───────────────────────────────────────────────────────────┘
```

Static files don't require processing by the server - they're served as-is directly to the client.

---

## wwwroot Folder

### What is wwwroot?

`wwwroot` is the **web root** folder in ASP.NET Core applications. It's the only folder whose contents are served directly to clients by default.

### Standard Structure

```
MyApp/
├── Controllers/           ← Server-side (NOT accessible via web)
├── Models/                ← Server-side (NOT accessible via web)
├── Views/                 ← Server-side (NOT accessible via web)
├── wwwroot/               ← Web root (ACCESSIBLE via web)
│   ├── css/
│   │   ├── site.css
│   │   └── bootstrap.min.css
│   ├── js/
│   │   ├── site.js
│   │   └── jquery.min.js
│   ├── images/
│   │   ├── logo.png
│   │   └── banner.jpg
│   ├── lib/
│   │   ├── bootstrap/
│   │   └── jquery/
│   └── favicon.ico
├── appsettings.json       ← Server-side (NOT accessible via web)
└── Program.cs             ← Server-side (NOT accessible via web)
```

### URL Mapping

```
URL Request                    →    File Path
────────────────────────────────────────────────────────
http://localhost/css/site.css  →    wwwroot/css/site.css
http://localhost/js/site.js    →    wwwroot/js/site.js
http://localhost/images/logo.png →  wwwroot/images/logo.png
http://localhost/favicon.ico   →    wwwroot/favicon.ico

Note: "wwwroot" is NOT part of the URL
```

### Visual Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                     URL TO FILE MAPPING                          │
└─────────────────────────────────────────────────────────────────┘

Client Request: GET /css/site.css
                    │
                    ▼
┌─────────────────────────────────────────────────────────────────┐
│                 STATIC FILES MIDDLEWARE                          │
│                                                                  │
│   Request Path: /css/site.css                                   │
│          +                                                       │
│   Web Root: wwwroot/                                            │
│          =                                                       │
│   File Path: wwwroot/css/site.css                               │
│                                                                  │
│   File exists? YES → Return file content                        │
│   File exists? NO  → Pass to next middleware                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Static Files Middleware

### Basic Usage

```csharp
// File: Program.cs

var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// Line 1: Enable static file serving from wwwroot
// Without this, static files will NOT be served!
app.UseStaticFiles();

app.MapControllers();
app.Run();
```

### How It Works

```csharp
// When a request comes in:
// 1. Static Files Middleware checks if the URL matches a file in wwwroot
// 2. If file exists: Return the file (short-circuit)
// 3. If file doesn't exist: Pass to next middleware

// Example:
// GET /css/site.css → Check wwwroot/css/site.css → Found → Return file
// GET /api/products → Check wwwroot/api/products → Not found → Next middleware
```

### Content Types

The middleware automatically sets the correct `Content-Type` header based on file extension:

| Extension | Content-Type |
|-----------|--------------|
| `.html` | `text/html` |
| `.css` | `text/css` |
| `.js` | `application/javascript` |
| `.json` | `application/json` |
| `.png` | `image/png` |
| `.jpg/.jpeg` | `image/jpeg` |
| `.svg` | `image/svg+xml` |
| `.woff2` | `font/woff2` |
| `.pdf` | `application/pdf` |

---

## Default Documents

### What are Default Documents?

Default documents are files that are served when a user requests a directory (e.g., `/` or `/docs/`).

### Default Document Options

```csharp
// File: Program.cs

var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// ═══════════════════════════════════════════════════════════════
// OPTION 1: UseDefaultFiles (Must be BEFORE UseStaticFiles)
// ═══════════════════════════════════════════════════════════════

app.UseDefaultFiles();  // Must be called FIRST
app.UseStaticFiles();

// Default files searched in order:
// 1. default.htm
// 2. default.html
// 3. index.htm
// 4. index.html

// ═══════════════════════════════════════════════════════════════
// OPTION 2: Custom Default Files
// ═══════════════════════════════════════════════════════════════

var defaultOptions = new DefaultFilesOptions();
defaultOptions.DefaultFileNames.Clear();
defaultOptions.DefaultFileNames.Add("home.html");  // Custom default

app.UseDefaultFiles(defaultOptions);
app.UseStaticFiles();

// ═══════════════════════════════════════════════════════════════
// OPTION 3: UseFileServer (Combines Default + Static + Directory)
// ═══════════════════════════════════════════════════════════════

app.UseFileServer();  // Combines UseDefaultFiles + UseStaticFiles

app.Run();
```

### Default Files Flow

```
Request: GET /
           │
           ▼
┌──────────────────────────────────────────────────────────────┐
│              UseDefaultFiles Middleware                       │
│                                                              │
│  1. Check for /default.htm  → Not found                     │
│  2. Check for /default.html → Not found                     │
│  3. Check for /index.htm    → Not found                     │
│  4. Check for /index.html   → FOUND!                        │
│                                                              │
│  Rewrite request path: / → /index.html                      │
└──────────────────────────────────────────────────────────────┘
           │
           ▼
┌──────────────────────────────────────────────────────────────┐
│              UseStaticFiles Middleware                        │
│                                                              │
│  Serve wwwroot/index.html                                    │
└──────────────────────────────────────────────────────────────┘
```

### Important: Order Matters!

```csharp
// ✅ CORRECT: UseDefaultFiles BEFORE UseStaticFiles
app.UseDefaultFiles();
app.UseStaticFiles();

// ❌ WRONG: UseStaticFiles first - default files won't work
app.UseStaticFiles();
app.UseDefaultFiles();  // Too late - static files already processed!
```

---

## Custom Static File Paths

### Serving Files from Additional Folders

```csharp
// File: Program.cs

var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// ═══════════════════════════════════════════════════════════════
// DEFAULT: Serve from wwwroot
// ═══════════════════════════════════════════════════════════════
app.UseStaticFiles();

// ═══════════════════════════════════════════════════════════════
// ADDITIONAL: Serve from custom folder
// ═══════════════════════════════════════════════════════════════

app.UseStaticFiles(new StaticFileOptions
{
    // Line 1: Physical file provider points to actual folder
    FileProvider = new PhysicalFileProvider(
        Path.Combine(builder.Environment.ContentRootPath, "MyDocuments")),
    
    // Line 2: Request path prefix for URLs
    RequestPath = "/documents"
});

// URL: /documents/report.pdf → MyDocuments/report.pdf

// ═══════════════════════════════════════════════════════════════
// MULTIPLE FOLDERS
// ═══════════════════════════════════════════════════════════════

// Serve from Images folder
app.UseStaticFiles(new StaticFileOptions
{
    FileProvider = new PhysicalFileProvider(
        Path.Combine(builder.Environment.ContentRootPath, "Images")),
    RequestPath = "/images"
});

// Serve from Downloads folder
app.UseStaticFiles(new StaticFileOptions
{
    FileProvider = new PhysicalFileProvider(
        Path.Combine(builder.Environment.ContentRootPath, "Downloads")),
    RequestPath = "/downloads"
});

app.Run();
```

### Folder Structure Example

```
MyApp/
├── wwwroot/               ← URL: /css/site.css
│   └── css/
│       └── site.css
├── MyDocuments/           ← URL: /documents/report.pdf
│   └── report.pdf
├── Images/                ← URL: /images/photo.jpg
│   └── photo.jpg
└── Downloads/             ← URL: /downloads/file.zip
    └── file.zip
```

---

## File Server Middleware

### UseFileServer

`UseFileServer` combines multiple middleware:
- `UseDefaultFiles`
- `UseStaticFiles`
- `UseDirectoryBrowser` (optional)

```csharp
// File: Program.cs

var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// ═══════════════════════════════════════════════════════════════
// OPTION 1: Basic FileServer (DefaultFiles + StaticFiles)
// ═══════════════════════════════════════════════════════════════

app.UseFileServer();  // Equivalent to UseDefaultFiles + UseStaticFiles

// ═══════════════════════════════════════════════════════════════
// OPTION 2: FileServer with Directory Browsing
// ═══════════════════════════════════════════════════════════════

app.UseFileServer(new FileServerOptions
{
    // Enable directory listing (shows files in folder)
    EnableDirectoryBrowsing = true  // ⚠️ Security risk in production!
});

// ═══════════════════════════════════════════════════════════════
// OPTION 3: FileServer from Custom Path
// ═══════════════════════════════════════════════════════════════

app.UseFileServer(new FileServerOptions
{
    FileProvider = new PhysicalFileProvider(
        Path.Combine(builder.Environment.ContentRootPath, "SharedFiles")),
    RequestPath = "/files",
    EnableDefaultFiles = true,
    EnableDirectoryBrowsing = false  // Keep disabled for security
});

app.Run();
```

### Directory Browsing

```csharp
// Enable directory browsing separately
builder.Services.AddDirectoryBrowser();

app.UseDirectoryBrowser(new DirectoryBrowserOptions
{
    FileProvider = new PhysicalFileProvider(
        Path.Combine(builder.Environment.ContentRootPath, "SharedFiles")),
    RequestPath = "/files"
});

// ⚠️ WARNING: Directory browsing exposes file structure
// Only enable in controlled environments
```

---

## Security Considerations

### Content Type Restrictions

```csharp
// Restrict which file types can be served
app.UseStaticFiles(new StaticFileOptions
{
    ContentTypeProvider = new FileExtensionContentTypeProvider(
        new Dictionary<string, string>
        {
            // Only allow these file types
            { ".html", "text/html" },
            { ".css", "text/css" },
            { ".js", "application/javascript" },
            { ".png", "image/png" },
            { ".jpg", "image/jpeg" }
            // .config, .json, etc. NOT served
        })
});
```

### Serving Unknown File Types

```csharp
// By default, unknown file types are NOT served (404)
// To serve all file types (⚠️ security risk):

app.UseStaticFiles(new StaticFileOptions
{
    ServeUnknownFileTypes = true,  // ⚠️ Be careful!
    DefaultContentType = "application/octet-stream"
});
```

### Security Best Practices

```
┌─────────────────────────────────────────────────────────────────┐
│                  STATIC FILES SECURITY                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ✅ DO:                                                         │
│  • Keep static files ONLY in wwwroot                            │
│  • Use specific file type restrictions                          │
│  • Enable response caching for performance                      │
│  • Use CDN for production                                       │
│                                                                  │
│  ❌ DON'T:                                                      │
│  • Enable directory browsing in production                      │
│  • Serve unknown file types                                     │
│  • Put sensitive files in wwwroot                               │
│  • Expose appsettings.json or other config files               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Code Examples

### Complete Static Files Configuration

```csharp
// File: Program.cs

using Microsoft.Extensions.FileProviders;

var builder = WebApplication.CreateBuilder(args);

// Add required services
builder.Services.AddControllersWithViews();

var app = builder.Build();

// ═══════════════════════════════════════════════════════════════
// ERROR HANDLING (First)
// ═══════════════════════════════════════════════════════════════
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Error");
    app.UseHsts();
}

app.UseHttpsRedirection();

// ═══════════════════════════════════════════════════════════════
// STATIC FILES CONFIGURATION
// ═══════════════════════════════════════════════════════════════

// Line 1: Enable default files (index.html, etc.)
app.UseDefaultFiles();

// Line 2: Serve static files from wwwroot with caching
app.UseStaticFiles(new StaticFileOptions
{
    // Line 3: Add caching headers for better performance
    OnPrepareResponse = ctx =>
    {
        // Cache static files for 1 year
        ctx.Context.Response.Headers.Append(
            "Cache-Control", "public,max-age=31536000");
    }
});

// Line 4: Serve files from custom folder
app.UseStaticFiles(new StaticFileOptions
{
    FileProvider = new PhysicalFileProvider(
        Path.Combine(builder.Environment.ContentRootPath, "Documents")),
    RequestPath = "/docs",
    OnPrepareResponse = ctx =>
    {
        // Cache documents for 1 hour
        ctx.Context.Response.Headers.Append(
            "Cache-Control", "public,max-age=3600");
    }
});

// ═══════════════════════════════════════════════════════════════
// ROUTING AND ENDPOINTS
// ═══════════════════════════════════════════════════════════════
app.UseRouting();
app.UseAuthorization();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();
```

### Using Static Files in Views

```html
<!-- File: Views/Shared/_Layout.cshtml -->
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <title>@ViewData["Title"]</title>
    
    <!-- Line 1: Reference CSS from wwwroot/css -->
    <link rel="stylesheet" href="~/css/bootstrap.min.css" />
    <link rel="stylesheet" href="~/css/site.css" />
</head>
<body>
    <header>
        <!-- Line 2: Reference image from wwwroot/images -->
        <img src="~/images/logo.png" alt="Logo" />
    </header>
    
    <main>
        @RenderBody()
    </main>
    
    <!-- Line 3: Reference JS from wwwroot/js -->
    <script src="~/js/jquery.min.js"></script>
    <script src="~/js/bootstrap.bundle.min.js"></script>
    <script src="~/js/site.js"></script>
    
    @await RenderSectionAsync("Scripts", required: false)
</body>
</html>
```

### Tilde (~) Path Resolution

```html
<!-- The tilde (~) resolves to the web root (wwwroot) -->

<!-- These are equivalent: -->
<link href="~/css/site.css" rel="stylesheet" />
<link href="/css/site.css" rel="stylesheet" />

<!-- Use tilde for app-relative paths -->
<!-- Works correctly even if app is hosted in a sub-folder -->
```

---

## Best Practices

### 1. Place Static Files Middleware Early

```csharp
// ✅ CORRECT: Static files before authentication
app.UseStaticFiles();      // Fast - no auth needed for CSS/JS
app.UseAuthentication();

// ❌ WRONG: Static files after authentication (slower)
app.UseAuthentication();
app.UseStaticFiles();      // Auth runs even for static files!
```

### 2. Enable Response Caching

```csharp
app.UseStaticFiles(new StaticFileOptions
{
    OnPrepareResponse = ctx =>
    {
        // Cache for 1 year (common for versioned files)
        ctx.Context.Response.Headers.Append(
            "Cache-Control", "public,max-age=31536000");
    }
});
```

### 3. Use CDN in Production

```html
<!-- Development: Use local files -->
@if (Environment.IsDevelopment())
{
    <link href="~/lib/bootstrap/css/bootstrap.css" rel="stylesheet" />
}
else
{
    <!-- Production: Use CDN with fallback -->
    <link href="https://cdn.example.com/bootstrap.css" rel="stylesheet" />
}
```

### 4. Version Your Static Files

```html
<!-- Add version query string for cache busting -->
<link href="~/css/site.css?v=1.0.0" rel="stylesheet" />

<!-- Or use asp-append-version tag helper -->
<link href="~/css/site.css" asp-append-version="true" rel="stylesheet" />
<!-- Outputs: /css/site.css?v=abc123hash -->
```

---

## Interview Questions

### Q1: What is wwwroot folder?
**Answer:** wwwroot is the web root folder in ASP.NET Core. It's the default location for static files (CSS, JavaScript, images). Files in wwwroot are served directly to clients without any server-side processing.

### Q2: Why must UseDefaultFiles() be called before UseStaticFiles()?
**Answer:** UseDefaultFiles() rewrites the request path (e.g., `/` to `/index.html`) but doesn't serve files. UseStaticFiles() actually serves the files. If the order is reversed, UseStaticFiles() won't find anything at `/` because it hasn't been rewritten yet.

### Q3: How do you serve files from a folder outside wwwroot?
**Answer:** Use StaticFileOptions with a PhysicalFileProvider:
```csharp
app.UseStaticFiles(new StaticFileOptions
{
    FileProvider = new PhysicalFileProvider(
        Path.Combine(builder.Environment.ContentRootPath, "MyFolder")),
    RequestPath = "/files"
});
```

### Q4: What is the difference between UseStaticFiles and UseFileServer?
**Answer:**
- `UseStaticFiles`: Only serves static files
- `UseFileServer`: Combines UseDefaultFiles + UseStaticFiles + optionally UseDirectoryBrowser

### Q5: Why should static files middleware be placed before authentication middleware?
**Answer:** Static files (CSS, JS, images) typically don't require authentication. Placing UseStaticFiles() before UseAuthentication() improves performance because:
1. Static files are served immediately without auth checks
2. Reduces unnecessary processing overhead
3. Faster page load times

---

## Quick Reference

### Middleware Methods

| Method | Purpose |
|--------|---------|
| `UseStaticFiles()` | Serve static files from wwwroot |
| `UseDefaultFiles()` | Set default document (index.html) |
| `UseFileServer()` | Combine default files + static files |
| `UseDirectoryBrowser()` | Show directory listing |

### Common Static File Paths

```
wwwroot/
├── css/          → href="~/css/..."
├── js/           → src="~/js/..."
├── images/       → src="~/images/..."
├── lib/          → Third-party libraries
└── favicon.ico   → Browser tab icon
```

### StaticFileOptions

```csharp
new StaticFileOptions
{
    FileProvider = ...,           // Custom file location
    RequestPath = "/path",        // URL prefix
    ContentTypeProvider = ...,    // Custom content types
    ServeUnknownFileTypes = false, // Block unknown types
    OnPrepareResponse = ctx => {} // Response customization
}
```
