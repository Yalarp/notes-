# Layout Pages in ASP.NET Core

## Table of Contents
1. [Introduction](#introduction)
2. [What are Layout Pages?](#what-are-layout-pages)
3. [Creating Layout Pages](#creating-layout-pages)
4. [Using Layout Pages](#using-layout-pages)
5. [Sections in Layout](#sections-in-layout)
6. [_ViewStart.cshtml](#_viewstartcshtml)
7. [_ViewImports.cshtml](#_viewimportscshtml)
8. [Nested Layouts](#nested-layouts)
9. [Code Examples](#code-examples)
10. [Best Practices](#best-practices)
11. [Interview Questions](#interview-questions)

---

## Introduction

### Real-World Analogy
Layout Pages are like **building templates in construction**:

```
Building Template (Layout)
    ├── Foundation (Header)
    ├── Frame structure (Navigation)
    ├── EMPTY SPACE (where content varies - @RenderBody())
    ├── Utilities (Footer)
    └── Wiring (Scripts)

Each house uses the same template but has different interior (content).
```

---

## What are Layout Pages?

Layout pages define the common structure (header, footer, navigation) shared across multiple views, avoiding code duplication.

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Layout** | Master template with common elements |
| **@RenderBody()** | Placeholder where view content is injected |
| **Sections** | Named placeholders for optional content |
| **_ViewStart** | Sets default layout for all views |
| **_ViewImports** | Imports namespaces and tag helpers |

### Layout vs Regular View

```
┌─────────────────────────────────────────────────────────────────┐
│                    LAYOUT PAGE STRUCTURE                         │
└─────────────────────────────────────────────────────────────────┘

┌───────────────────────────────────────────────────────────────┐
│                         LAYOUT                                 │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │                    HEADER                                │  │
│  │  Logo, Site Name, Main Navigation                       │  │
│  └─────────────────────────────────────────────────────────┘  │
│                                                               │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │              @RenderBody()                               │  │
│  │                                                          │  │
│  │  ← Content from individual views injected here          │  │
│  │                                                          │  │
│  └─────────────────────────────────────────────────────────┘  │
│                                                               │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │              @RenderSection("Scripts")                   │  │
│  │  ← Optional page-specific scripts                       │  │
│  └─────────────────────────────────────────────────────────┘  │
│                                                               │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │                    FOOTER                                │  │
│  │  Copyright, Links, Social Media                         │  │
│  └─────────────────────────────────────────────────────────┘  │
└───────────────────────────────────────────────────────────────┘
```

---

## Creating Layout Pages

### Basic Layout Structure

```html
<!-- File: Views/Shared/_Layout.cshtml -->

<!DOCTYPE html>
<html lang="en">
<head>
    <!-- ═══════════════════════════════════════════════════════ -->
    <!-- HEAD SECTION - Common meta tags, CSS -->
    <!-- ═══════════════════════════════════════════════════════ -->
    
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    
    <!-- Line 1: Dynamic page title -->
    <title>@ViewData["Title"] - My Application</title>
    
    <!-- Line 2: Common CSS files -->
    <link rel="stylesheet" href="~/lib/bootstrap/dist/css/bootstrap.min.css" />
    <link rel="stylesheet" href="~/css/site.css" asp-append-version="true" />
</head>
<body>
    <!-- ═══════════════════════════════════════════════════════ -->
    <!-- HEADER SECTION -->
    <!-- ═══════════════════════════════════════════════════════ -->
    <header>
        <nav class="navbar navbar-expand-sm navbar-dark bg-dark">
            <div class="container">
                <a class="navbar-brand" asp-area="" asp-controller="Home" asp-action="Index">
                    My Application
                </a>
                <button class="navbar-toggler" type="button" data-bs-toggle="collapse" 
                        data-bs-target="#navbarNav">
                    <span class="navbar-toggler-icon"></span>
                </button>
                <div class="collapse navbar-collapse" id="navbarNav">
                    <ul class="navbar-nav ms-auto">
                        <li class="nav-item">
                            <a class="nav-link" asp-controller="Home" asp-action="Index">Home</a>
                        </li>
                        <li class="nav-item">
                            <a class="nav-link" asp-controller="Employee" asp-action="Index">Employees</a>
                        </li>
                    </ul>
                </div>
            </div>
        </nav>
    </header>
    
    <!-- ═══════════════════════════════════════════════════════ -->
    <!-- MAIN CONTENT SECTION -->
    <!-- ═══════════════════════════════════════════════════════ -->
    <main role="main" class="pb-3">
        <div class="container">
            <!-- Line 3: CRITICAL - RenderBody() injects view content here -->
            @RenderBody()
        </div>
    </main>
    
    <!-- ═══════════════════════════════════════════════════════ -->
    <!-- FOOTER SECTION -->
    <!-- ═══════════════════════════════════════════════════════ -->
    <footer class="border-top footer text-muted">
        <div class="container">
            &copy; 2024 - My Application - 
            <a asp-controller="Home" asp-action="Privacy">Privacy</a>
        </div>
    </footer>
    
    <!-- ═══════════════════════════════════════════════════════ -->
    <!-- SCRIPTS SECTION -->
    <!-- ═══════════════════════════════════════════════════════ -->
    
    <!-- Line 4: Common scripts for all pages -->
    <script src="~/lib/jquery/dist/jquery.min.js"></script>
    <script src="~/lib/bootstrap/dist/js/bootstrap.bundle.min.js"></script>
    <script src="~/js/site.js" asp-append-version="true"></script>
    
    <!-- Line 5: Optional Scripts section for page-specific scripts -->
    @await RenderSectionAsync("Scripts", required: false)
</body>
</html>
```

---

## Using Layout Pages

### Method 1: Specify in Individual View

```html
<!-- File: Views/Employee/Index.cshtml -->

@{
    // Line 1: Explicitly set layout for this view
    Layout = "_Layout";
    ViewData["Title"] = "Employee List";
}

<h1>@ViewData["Title"]</h1>

<p>This content will be rendered where @RenderBody() is in _Layout.cshtml</p>
```

### Method 2: Using _ViewStart.cshtml (Recommended)

```html
<!-- File: Views/_ViewStart.cshtml -->

@{
    // Line 1: Set default layout for ALL views
    // Every view in this folder and subfolders will use this layout
    Layout = "_Layout";
}
```

```html
<!-- File: Views/Employee/Index.cshtml -->

@{
    // Line 1: No need to specify layout - inherited from _ViewStart
    ViewData["Title"] = "Employee List";
}

<h1>@ViewData["Title"]</h1>
```

### Method 3: Override Default Layout

```html
<!-- File: Views/Employee/Print.cshtml -->

@{
    // Line 1: Override _ViewStart layout with null (no layout)
    Layout = null;
}

<!DOCTYPE html>
<html>
<head>
    <title>Print View</title>
</head>
<body>
    <h1>Print-friendly version</h1>
    <!-- No header, footer, navigation -->
</body>
</html>
```

---

## Sections in Layout

### Defining Sections in Layout

```html
<!-- File: Views/Shared/_Layout.cshtml -->

<!DOCTYPE html>
<html>
<head>
    <title>@ViewData["Title"]</title>
    
    <!-- Line 1: Optional CSS section -->
    @await RenderSectionAsync("Styles", required: false)
</head>
<body>
    <header>
        <nav>...</nav>
    </header>
    
    <main>
        @RenderBody()
    </main>
    
    <footer>...</footer>
    
    <!-- Common scripts -->
    <script src="~/lib/jquery/dist/jquery.min.js"></script>
    
    <!-- Line 2: Optional Scripts section -->
    @await RenderSectionAsync("Scripts", required: false)
    
    <!-- Line 3: Required section (must be defined in views) -->
    @await RenderSectionAsync("PageTitle", required: true)
</body>
</html>
```

### Implementing Sections in Views

```html
<!-- File: Views/Employee/Index.cshtml -->

@{
    ViewData["Title"] = "Employees";
}

<!-- ═══════════════════════════════════════════════════════════ -->
<!-- MAIN CONTENT (goes into @RenderBody()) -->
<!-- ═══════════════════════════════════════════════════════════ -->

<h1>Employee List</h1>

<table class="table">
    <!-- Employee table content -->
</table>

<!-- ═══════════════════════════════════════════════════════════ -->
<!-- OPTIONAL: Page-specific styles -->
<!-- ═══════════════════════════════════════════════════════════ -->

@section Styles {
    <style>
        .employee-table {
            border: 2px solid #007bff;
        }
    </style>
}

<!-- ═══════════════════════════════════════════════════════════ -->
<!-- OPTIONAL: Page-specific scripts -->
<!-- ═══════════════════════════════════════════════════════════ -->

@section Scripts {
    <script>
        $(document).ready(function() {
            console.log('Employee page loaded');
            
            // Initialize DataTables
            $('.table').DataTable();
        });
    </script>
}

<!-- ═══════════════════════════════════════════════════════════ -->
<!-- REQUIRED: Page title section -->
<!-- ═══════════════════════════════════════════════════════════ -->

@section PageTitle {
    <h1>Employee Management</h1>
}
```

### Checking if Section is Defined

```html
<!-- File: Views/Shared/_Layout.cshtml -->

<!-- Line 1: Render section only if defined -->
@if (IsSectionDefined("Breadcrumb"))
{
    <div class="breadcrumb-container">
        @RenderSection("Breadcrumb")
    </div>
}

<!-- Line 2: Render section with default content -->
@if (IsSectionDefined("PageHeader"))
{
    @RenderSection("PageHeader")
}
else
{
    <h1>@ViewData["Title"]</h1>
}
```

---

## _ViewStart.cshtml

### Purpose and Location

```
Views/
├── _ViewStart.cshtml          ← Root level (applies to all)
├── Home/
│   └── Index.cshtml           ← Uses root _ViewStart
├── Employee/
│   ├── _ViewStart.cshtml      ← Folder level (overrides root)
│   ├── Index.cshtml           ← Uses Employee/_ViewStart
│   └── Details.cshtml         ← Uses Employee/_ViewStart
└── Shared/
    └── _Layout.cshtml
```

### Execution Order

```
┌─────────────────────────────────────────────────────────────────┐
│                 _ViewStart EXECUTION ORDER                       │
└─────────────────────────────────────────────────────────────────┘

Request: /Employee/Index

1. Views/_ViewStart.cshtml          ← Executes FIRST (root)
        │
        ▼
2. Views/Employee/_ViewStart.cshtml ← Executes SECOND (folder)
        │                             (can override root settings)
        ▼
3. Views/Employee/Index.cshtml      ← Executes LAST (view)
        │                             (can override all)
        ▼
   Final layout determined
```

### Root _ViewStart

```html
<!-- File: Views/_ViewStart.cshtml -->

@{
    // Line 1: Set default layout for entire application
    Layout = "_Layout";
}
```

### Folder-Specific _ViewStart

```html
<!-- File: Views/Admin/_ViewStart.cshtml -->

@{
    // Line 1: Admin area uses different layout
    Layout = "_AdminLayout";
}
```

---

## _ViewImports.cshtml

### Purpose

`_ViewImports.cshtml` shares directives across all views (namespaces, tag helpers, models).

```html
<!-- File: Views/_ViewImports.cshtml -->

@* Line 1: Import namespaces *@
@using MVCEmpDept
@using MVCEmpDept.Models
@using MVCEmpDept.ViewModels
@using Microsoft.AspNetCore.Identity

@* Line 2: Enable tag helpers *@
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers

@* Line 3: Enable custom tag helpers *@
@addTagHelper *, MVCEmpDept

@* Line 4: Inject services (available in all views) *@
@inject IConfiguration Configuration
@inject UserManager<ApplicationUser> UserManager
```

### Hierarchical _ViewImports

```
Views/
├── _ViewImports.cshtml               ← Global imports
├── Employee/
│   ├── _ViewImports.cshtml           ← Additional employee-specific
│   └── Index.cshtml                  ← Gets both global + employee
└── Admin/
    ├── _ViewImports.cshtml           ← Additional admin-specific
    └── Dashboard.cshtml              ← Gets both global + admin
```

---

## Nested Layouts

### Creating Nested Layouts

```html
<!-- File: Views/Shared/_Layout.cshtml (Base Layout) -->

<!DOCTYPE html>
<html>
<head>
    <title>@ViewData["Title"]</title>
    <link rel="stylesheet" href="~/css/site.css" />
</head>
<body>
    <header>
        <h1>Company Name</h1>
    </header>
    
    <main>
        <!-- Line 1: Base layout's RenderBody -->
        @RenderBody()
    </main>
    
    <footer>
        <p>&copy; 2024 Company</p>
    </footer>
</body>
</html>
```

```html
<!-- File: Views/Shared/_AdminLayout.cshtml (Nested Layout) -->

@{
    // Line 1: This layout uses _Layout as its layout
    Layout = "_Layout";
}

<!-- Line 2: This content goes into _Layout's @RenderBody() -->
<div class="admin-container">
    <aside class="admin-sidebar">
        <nav>
            <a asp-controller="Admin" asp-action="Dashboard">Dashboard</a>
            <a asp-controller="Admin" asp-action="Users">Users</a>
            <a asp-controller="Admin" asp-action="Settings">Settings</a>
        </nav>
    </aside>
    
    <div class="admin-content">
        <!-- Line 3: Admin layout's RenderBody -->
        @RenderBody()
    </div>
</div>
```

```html
<!-- File: Views/Admin/Dashboard.cshtml (View) -->

@{
    // Line 1: Uses _AdminLayout
    Layout = "_AdminLayout";
    ViewData["Title"] = "Admin Dashboard";
}

<!-- This content goes into _AdminLayout's @RenderBody() -->
<h2>Dashboard</h2>
<p>Welcome to admin dashboard!</p>
```

### Nesting Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                      NESTED LAYOUT FLOW                          │
└─────────────────────────────────────────────────────────────────┘

Dashboard.cshtml (View)
        │
        │ Layout = "_AdminLayout"
        ▼
_AdminLayout.cshtml (Nested Layout)
        │ Content: Sidebar + Admin Container
        │ Layout = "_Layout"
        ▼
_Layout.cshtml (Base Layout)
        │ Content: Header + Footer
        │
        ▼
Final HTML Output:
    _Layout Header
        _AdminLayout Sidebar
            Dashboard.cshtml Content
        _AdminLayout Container
    _Layout Footer
```

---

## Code Examples

### Complete Layout with All Features

```html
<!-- File: Views/Shared/_Layout.cshtml -->

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>@ViewData["Title"] - Employee Management</title>
    
    <!-- Bootstrap CSS -->
    <link href="~/lib/bootstrap/dist/css/bootstrap.min.css" rel="stylesheet" />
    <link href="~/css/site.css" asp-append-version="true" rel="stylesheet" />
    
    <!-- Page-specific styles section -->
    @await RenderSectionAsync("Styles", required: false)
</head>
<body>
    <!-- Navigation Header -->
    <header>
        <nav class="navbar navbar-expand-lg navbar-dark bg-primary">
            <div class="container">
                <a class="navbar-brand" asp-controller="Home" asp-action="Index">
                    <i class="bi bi-building"></i> EmpManage
                </a>
                <button class="navbar-toggler" type="button" data-bs-toggle="collapse" 
                        data-bs-target="#navbarNav">
                    <span class="navbar-toggler-icon"></span>
                </button>
                <div class="collapse navbar-collapse" id="navbarNav">
                    <ul class="navbar-nav me-auto">
                        <li class="nav-item">
                            <a class="nav-link" asp-controller="Home" asp-action="Index">
                                Home
                            </a>
                        </li>
                        <li class="nav-item">
                            <a class="nav-link" asp-controller="Employee" asp-action="Index">
                                Employees
                            </a>
                        </li>
                        <li class="nav-item">
                            <a class="nav-link" asp-controller="Department" asp-action="Index">
                                Departments
                            </a>
                        </li>
                    </ul>
                    <ul class="navbar-nav">
                        <li class="nav-item">
                            <span class="navbar-text">
                                Welcome, @User.Identity?.Name
                            </span>
                        </li>
                    </ul>
                </div>
            </div>
        </nav>
    </header>
    
    <!-- Breadcrumb (if defined) -->
    @if (IsSectionDefined("Breadcrumb"))
    {
        <div class="container mt-2">
            @RenderSection("Breadcrumb")
        </div>
    }
    
    <!-- Main Content -->
    <main role="main" class="py-4">
        <div class="container">
            <!-- Alert messages (if any) -->
            @if (TempData["SuccessMessage"] != null)
            {
                <div class="alert alert-success alert-dismissible fade show">
                    @TempData["SuccessMessage"]
                    <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
                </div>
            }
            
            @if (TempData["ErrorMessage"] != null)
            {
                <div class="alert alert-danger alert-dismissible fade show">
                    @TempData["ErrorMessage"]
                    <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
                </div>
            }
            
            <!-- View content injected here -->
            @RenderBody()
        </div>
    </main>
    
    <!-- Footer -->
    <footer class="border-top footer text-muted mt-5">
        <div class="container py-3">
            <div class="row">
                <div class="col-md-6">
                    &copy; 2024 - Employee Management System
                </div>
                <div class="col-md-6 text-end">
                    <a asp-controller="Home" asp-action="Privacy">Privacy</a> | 
                    <a asp-controller="Home" asp-action="Contact">Contact</a>
                </div>
            </div>
        </div>
    </footer>
    
    <!-- Common Scripts -->
    <script src="~/lib/jquery/dist/jquery.min.js"></script>
    <script src="~/lib/bootstrap/dist/js/bootstrap.bundle.min.js"></script>
    <script src="~/js/site.js" asp-append-version="true"></script>
    
    <!-- Page-specific scripts section -->
    @await RenderSectionAsync("Scripts", required: false)
</body>
</html>
```

---

## Best Practices

### 1. Use _ViewStart for Default Layout

```html
<!-- ✅ GOOD: Set once in _ViewStart -->
<!-- Views/_ViewStart.cshtml -->
@{
    Layout = "_Layout";
}

<!-- ❌ BAD: Repeat in every view -->
@{
    Layout = "_Layout";  // Repeated in every view!
}
```

### 2. Make Sections Optional Unless Absolutely Required

```html
<!-- ✅ GOOD: Optional sections -->
@await RenderSectionAsync("Scripts", required: false)

<!-- ❌ BAD: Required section forces all views to define it -->
@await RenderSectionAsync("Scripts", required: true)
```

### 3. Use IsSectionDefined for Conditional Rendering

```html
<!-- ✅ GOOD: Check before rendering -->
@if (IsSectionDefined("Breadcrumb"))
{
    <div class="breadcrumb">
        @RenderSection("Breadcrumb")
    </div>
}

<!-- ❌ BAD: Always render container even if empty -->
<div class="breadcrumb">
    @await RenderSectionAsync("Breadcrumb", required: false)
</div>
```

---

## Interview Questions

### Q1: What is a Layout Page?
**Answer:** A Layout Page is a master template that defines the common structure (header, footer, navigation) shared across multiple views. It uses `@RenderBody()` to inject view-specific content.

### Q2: What is @RenderBody()?
**Answer:** `@RenderBody()` is a method in layout pages that marks the location where the content from individual views will be injected.

### Q3: What is _ViewStart.cshtml?
**Answer:** `_ViewStart.cshtml` sets the default layout for all views in its directory and subdirectories. It executes before each view and typically contains `Layout = "_Layout";`.

### Q4: What is the difference between @RenderSection and @RenderSectionAsync?
**Answer:** 
- `@RenderSection()` - Synchronous rendering (older)
- `@RenderSectionAsync()` - Asynchronous rendering (recommended, better performance)

### Q5: How do you create an optional section?
**Answer:** Use `required: false` parameter:
```html
@await RenderSectionAsync("Scripts", required: false)
```

### Q6: What is _ViewImports.cshtml?
**Answer:** `_ViewImports.cshtml` contains directives (@using, @addTagHelper, @inject) that are shared across all views, eliminating the need to repeat them in each view.

---

## Quick Reference

### Layout Essentials

```html
<!-- _Layout.cshtml -->
@RenderBody()                                      ← Required
@await RenderSectionAsync("Scripts", false)       ← Optional section
@await RenderSectionAsync("Styles", true)         ← Required section
```

### _ViewStart

```html
<!-- Views/_ViewStart.cshtml -->
@{
    Layout = "_Layout";
}
```

### _ViewImports

```html
<!-- Views/_ViewImports.cshtml -->
@using MyApp.Models
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
```

### Section in View

```html
@section Scripts {
    <script>
        // Page-specific JavaScript
    </script>
}
```
