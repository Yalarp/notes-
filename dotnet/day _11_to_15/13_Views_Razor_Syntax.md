# Views and Razor Syntax in ASP.NET Core

## Table of Contents
1. [Introduction](#introduction)
2. [What are Views?](#what-are-views)
3. [Razor Syntax Basics](#razor-syntax-basics)
4. [Razor Code Blocks](#razor-code-blocks)
5. [Razor Expressions](#razor-expressions)
6. [Control Structures](#control-structures)
7. [Razor Comments](#razor-comments)
8. [Strongly-Typed Views](#strongly-typed-views)
9. [View Discovery](#view-discovery)
10. [Code Examples](#code-examples)
11. [Best Practices](#best-practices)
12. [Interview Questions](#interview-questions)

---

## Introduction

### Real-World Analogy
Views are like **restaurant menus**:

```
Controller (Kitchen) prepares data (ingredients)
        │
        ▼
View (Menu) presents data to customer in readable format
        │
        ▼
HTML Response (Beautiful plate presentation)
```

Razor is the **template language** that mixes HTML with C# code to create dynamic web pages.

---

## What are Views?

Views are responsible for presenting content to the user. They are HTML templates with embedded Razor markup.

### View Structure

```
MyApp/
├── Views/
│   ├── Employee/              ← Views for EmployeeController
│   │   ├── Index.cshtml       ← List view
│   │   ├── Details.cshtml     ← Details view
│   │   ├── Create.cshtml      ← Create form
│   │   ├── Edit.cshtml        ← Edit form
│   │   └── Delete.cshtml      ← Delete confirmation
│   ├── Home/                  ← Views for HomeController
│   │   ├── Index.cshtml
│   │   └── Privacy.cshtml
│   └── Shared/                ← Shared views
│       ├── _Layout.cshtml     ← Master layout
│       ├── _ValidationScriptsPartial.cshtml
│       └── Error.cshtml       ← Error page
├── Pages/                     ← Razor Pages (alternative)
└── wwwroot/                   ← Static files
```

### View Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                        VIEW RENDERING FLOW                       │
└─────────────────────────────────────────────────────────────────┘

  Controller Action
         │
         ▼
  return View(model);
         │
         ▼
┌──────────────────────────────────────────────────────────────┐
│  VIEW DISCOVERY                                               │
│  1. Check Views/{ControllerName}/{ActionName}.cshtml         │
│  2. Check Views/Shared/{ActionName}.cshtml                   │
└────────────────────────────┬─────────────────────────────────┘
                             │
                             ▼
┌──────────────────────────────────────────────────────────────┐
│  RAZOR ENGINE                                                 │
│  1. Parse Razor syntax                                        │
│  2. Execute C# code                                          │
│  3. Generate HTML                                            │
└────────────────────────────┬─────────────────────────────────┘
                             │
                             ▼
                     HTML Response to Client
```

---

## Razor Syntax Basics

### The @ Symbol

The `@` symbol transitions from HTML to C# code.

```html
<!-- ═══════════════════════════════════════════════════════════ -->
<!-- RAZOR TRANSITIONS WITH @ -->
<!-- ═══════════════════════════════════════════════════════════ -->

<!-- Line 1: Output C# expression -->
<p>Current time: @DateTime.Now</p>
<!-- Output: <p>Current time: 12/19/2025 4:30:00 PM</p> -->

<!-- Line 2: Access model property -->
<h2>@Model.Name</h2>
<!-- Output: <h2>John Doe</h2> -->

<!-- Line 3: Call C# method -->
<p>Welcome, @User.Identity.Name!</p>

<!-- Line 4: Inline C# with parentheses -->
<p>Total: @(Model.Price * Model.Quantity)</p>
<!-- Parentheses needed for complex expressions -->

<!-- Line 5: String with @ (email example) -->
<p>Email: name@@company.com</p>
<!-- Use @@ to output literal @ -->
```

### Implicit vs Explicit Razor Expressions

```html
<!-- ═══════════════════════════════════════════════════════════ -->
<!-- IMPLICIT EXPRESSIONS -->
<!-- ═══════════════════════════════════════════════════════════ -->

<!-- Line 1: Simple property access -->
@Model.Name
<!-- Razor automatically detects end of expression -->

<!-- ═══════════════════════════════════════════════════════════ -->
<!-- EXPLICIT EXPRESSIONS (use parentheses) -->
<!-- ═══════════════════════════════════════════════════════════ -->

<!-- Line 2: Complex expression -->
@(Model.FirstName + " " + Model.LastName)

<!-- Line 3: Method call with parameters -->
@(String.Format("{0:C}", Model.Price))

<!-- Line 4: LINQ expression -->
@(Model.Items.Where(x => x.IsActive).Count())

<!-- Line 5: Ternary operator -->
@(Model.IsActive ? "Active" : "Inactive")
```

---

## Razor Code Blocks

### Single-Line Statements

```html
<!-- ═══════════════════════════════════════════════════════════ -->
<!-- SINGLE-LINE CODE STATEMENTS -->
<!-- ═══════════════════════════════════════════════════════════ -->

@{
    // Line 1: Declare variable
    var message = "Hello, World!";
}

@{
    // Line 2: Set ViewData
    ViewData["Title"] = "Employee List";
}

@{
    // Line 3: Calculate value
    var total = Model.Price * Model.Quantity;
}

<h1>@ViewData["Title"]</h1>
<p>@message</p>
<p>Total: @total</p>
```

### Multi-Line Code Blocks

```html
@{
    // ═══════════════════════════════════════════════════════════
    // MULTI-LINE CODE BLOCK
    // ═══════════════════════════════════════════════════════════
    
    // Line 1: Set page title
    ViewData["Title"] = "Employee Details";
    
    // Line 2: Declare and initialize variables
    var fullName = $"{Model.FirstName} {Model.LastName}";
    var greeting = "Welcome";
    
    // Line 3: Complex calculations
    var yearsOfService = DateTime.Now.Year - Model.JoinDate.Year;
    var isEligibleForBonus = yearsOfService >= 5;
    
    // Line 4: Conditional logic
    string status;
    if (Model.IsActive)
    {
        status = "Active";
    }
    else
    {
        status = "Inactive";
    }
    
    // Line 5: Collection manipulation
    var activeDepartments = Model.Departments
        .Where(d => d.IsActive)
        .OrderBy(d => d.Name)
        .ToList();
}

<!-- Use variables in HTML -->
<h1>@greeting, @fullName!</h1>
<p>Status: @status</p>
<p>Years of Service: @yearsOfService</p>
```

---

## Razor Expressions

### Outputting HTML

```html
<!-- ═══════════════════════════════════════════════════════════ -->
<!-- OUTPUTTING VALUES -->
<!-- ═══════════════════════════════════════════════════════════ -->

<!-- Line 1: Simple text -->
<p>Name: @Model.Name</p>

<!-- Line 2: HTML encoded by default (safe from XSS) -->
<p>@Model.Description</p>
<!-- If Description = "<script>alert('XSS')</script>" -->
<!-- Output: &lt;script&gt;alert('XSS')&lt;/script&gt; -->

<!-- Line 3: Raw HTML (use with caution!) -->
<div>@Html.Raw(Model.HtmlContent)</div>
<!-- Renders actual HTML - only use for trusted content! -->

<!-- Line 4: Formatted output -->
<p>Price: @Model.Price.ToString("C")</p>
<!-- Output: Price: $19.99 -->

<p>Date: @Model.OrderDate.ToString("MM/dd/yyyy")</p>
<!-- Output: Date: 12/19/2025 -->
```

### Null-Conditional Operators

```html
<!-- ═══════════════════════════════════════════════════════════ -->
<!-- NULL SAFETY -->
<!-- ═══════════════════════════════════════════════════════════ -->

<!-- Line 1: Null-conditional operator -->
<p>Department: @Model.Department?.Name</p>
<!-- If Department is null, outputs nothing (no error) -->

<!-- Line 2: Null-coalescing operator -->
<p>Phone: @(Model.Phone ?? "Not provided")</p>
<!-- If Phone is null, shows "Not provided" -->

<!-- Line 3: Combined -->
<p>Manager: @(Model.Department?.Manager?.Name ?? "No manager assigned")</p>
```

---

## Control Structures

### If-Else Statements

```html
<!-- ═══════════════════════════════════════════════════════════ -->
<!-- IF-ELSE STATEMENTS -->
<!-- ═══════════════════════════════════════════════════════════ -->

<!-- Line 1: Simple if -->
@if (Model.IsActive)
{
    <span class="badge bg-success">Active</span>
}

<!-- Line 2: If-else -->
@if (Model.Salary > 100000)
{
    <span class="badge bg-warning">High earner</span>
}
else
{
    <span class="badge bg-info">Standard</span>
}

<!-- Line 3: If-else if-else -->
@if (Model.Age < 18)
{
    <p>Minor</p>
}
else if (Model.Age < 65)
{
    <p>Adult</p>
}
else
{
    <p>Senior</p>
}

<!-- Line 4: Complex condition -->
@if (Model.IsActive && Model.Department != null)
{
    <div class="alert alert-success">
        @Model.Name is active in @Model.Department.Name
    </div>
}
```

### For Loops

```html
<!-- ═══════════════════════════════════════════════════════════ -->
<!-- FOR LOOPS -->
<!-- ═══════════════════════════════════════════════════════════ -->

<!-- Line 1: Standard for loop -->
@for (int i = 0; i < Model.Count; i++)
{
    <p>Item @i: @Model[i].Name</p>
}

<!-- Line 2: Creating numbered list -->
<ol>
    @for (int i = 1; i <= 10; i++)
    {
        <li>Item number @i</li>
    }
</ol>

<!-- Line 3: Table rows with alternating classes -->
<table class="table">
    @for (int i = 0; i < Model.Count; i++)
    {
        var cssClass = i % 2 == 0 ? "even-row" : "odd-row";
        <tr class="@cssClass">
            <td>@Model[i].Name</td>
            <td>@Model[i].Email</td>
        </tr>
    }
</table>
```

### Foreach Loops

```html
<!-- ═══════════════════════════════════════════════════════════ -->
<!-- FOREACH LOOPS -->
<!-- ═══════════════════════════════════════════════════════════ -->

<!-- Line 1: Simple foreach -->
@foreach (var employee in Model)
{
    <div class="card">
        <h3>@employee.Name</h3>
        <p>@employee.Email</p>
    </div>
}

<!-- Line 2: Table with foreach -->
<table class="table">
    <thead>
        <tr>
            <th>Name</th>
            <th>Email</th>
            <th>Department</th>
            <th>Actions</th>
        </tr>
    </thead>
    <tbody>
        @foreach (var item in Model)
        {
            <tr>
                <td>@item.Name</td>
                <td>@item.Email</td>
                <td>@item.Department?.Name</td>
                <td>
                    <a asp-action="Details" asp-route-id="@item.Id">Details</a> |
                    <a asp-action="Edit" asp-route-id="@item.Id">Edit</a> |
                    <a asp-action="Delete" asp-route-id="@item.Id">Delete</a>
                </td>
            </tr>
        }
    </tbody>
</table>

<!-- Line 3: Nested foreach -->
@foreach (var department in Model.Departments)
{
    <div class="department">
        <h3>@department.Name</h3>
        <ul>
            @foreach (var employee in department.Employees)
            {
                <li>@employee.Name - @employee.Position</li>
            }
        </ul>
    </div>
}
```

### While Loops

```html
<!-- ═══════════════════════════════════════════════════════════ -->
<!-- WHILE LOOPS -->
<!-- ═══════════════════════════════════════════════════════════ -->

@{
    var counter = 0;
}

@while (counter < 5)
{
    <p>Count: @counter</p>
    counter++;
}
```

### Switch Statements

```html
<!-- ═══════════════════════════════════════════════════════════ -->
<!-- SWITCH STATEMENTS -->
<!-- ═══════════════════════════════════════════════════════════ -->

@switch (Model.Status)
{
    case "Active":
        <span class="badge bg-success">Active</span>
        break;
    case "Inactive":
        <span class="badge bg-secondary">Inactive</span>
        break;
    case "Pending":
        <span class="badge bg-warning">Pending</span>
        break;
    default:
        <span class="badge bg-secondary">Unknown</span>
        break;
}

<!-- Modern C# switch expression -->
@{
    var badgeClass = Model.Status switch
    {
        "Active" => "bg-success",
        "Inactive" => "bg-secondary",
        "Pending" => "bg-warning",
        _ => "bg-secondary"
    };
}
<span class="badge @badgeClass">@Model.Status</span>
```

---

## Razor Comments

### Comment Types

```html
<!-- ═══════════════════════════════════════════════════════════ -->
<!-- COMMENTS -->
<!-- ═══════════════════════════════════════════════════════════ -->

<!-- Line 1: HTML Comment (visible in source) -->
<!-- This is an HTML comment -->
<!-- Users can see this in "View Source" -->

@* Line 2: Razor Comment (not sent to client) *@
@* This is a Razor comment *@
@* Users CANNOT see this in "View Source" *@
@* Better for sensitive notes or server-side documentation *@

@* 
   Line 3: Multi-line Razor comment
   This can span
   multiple lines
   and won't appear in HTML source
*@

@{
    // Line 4: C# comment inside code block
    // This is a C# comment
    var name = "John";
    /* This is a C#
       multi-line comment */
}
```

---

## Strongly-Typed Views

### @model Directive

```html
<!-- File: Views/Employee/Details.cshtml -->

@* Line 1: Declare view model type *@
@model Employee

@* Now we have IntelliSense for Model properties *@

<div class="card">
    <div class="card-header">
        <h2>@Model.Name</h2>
    </div>
    <div class="card-body">
        <dl class="row">
            <dt class="col-sm-3">Email</dt>
            <dd class="col-sm-9">@Model.Email</dd>
            
            <dt class="col-sm-3">Phone</dt>
            <dd class="col-sm-9">@Model.Phone</dd>
            
            <dt class="col-sm-3">Department</dt>
            <dd class="col-sm-9">@Model.Department?.Name</dd>
            
            <dt class="col-sm-3">Join Date</dt>
            <dd class="col-sm-9">@Model.JoinDate.ToString("MM/dd/yyyy")</dd>
        </dl>
    </div>
</div>
```

### Collection Views

```html
<!-- File: Views/Employee/Index.cshtml -->

@* Line 1: View model is a collection *@
@model IEnumerable<Employee>

<h1>Employees</h1>

@if (Model.Any())
{
    <table class="table table-striped">
        <thead>
            <tr>
                <th>Name</th>
                <th>Email</th>
                <th>Department</th>
                <th>Actions</th>
            </tr>
        </thead>
        <tbody>
            @foreach (var employee in Model)
            {
                <tr>
                    <td>@employee.Name</td>
                    <td>@employee.Email</td>
                    <td>@employee.Department?.Name</td>
                    <td>
                        <a asp-action="Details" asp-route-id="@employee.Id">Details</a>
                    </td>
                </tr>
            }
        </tbody>
    </table>
}
else
{
    <div class="alert alert-info">
        No employees found.
    </div>
}
```

---

## View Discovery

### How ASP.NET Core Finds Views

```
┌─────────────────────────────────────────────────────────────────┐
│                      VIEW DISCOVERY ORDER                        │
└─────────────────────────────────────────────────────────────────┘

Controller Action: return View("Details");
from EmployeeController

Search Order:
─────────────────────────────────────────────────────────────────
1. /Views/Employee/Details.cshtml           ← Specific to controller
2. /Views/Shared/Details.cshtml             ← Shared across controllers

If view name not specified: return View();
Uses action name as view name (Details)

Specifying explicit path: return View("~/Views/Custom/MyView.cshtml");
─────────────────────────────────────────────────────────────────
```

### Different View Return Methods

```csharp
// File: Controllers/EmployeeController.cs

public class EmployeeController : Controller
{
    // ═══════════════════════════════════════════════════════════
    // METHOD 1: Default view for action
    // ═══════════════════════════════════════════════════════════
    public IActionResult Index()
    {
        var model = GetEmployees();
        return View();  // Looks for Views/Employee/Index.cshtml
    }
    
    // ═══════════════════════════════════════════════════════════
    // METHOD 2: Default view with model
    // ═══════════════════════════════════════════════════════════
    public IActionResult Details(int id)
    {
        var employee = GetEmployee(id);
        return View(employee);  // Views/Employee/Details.cshtml with model
    }
    
    // ═══════════════════════════════════════════════════════════
    // METHOD 3: Specific view name
    // ═══════════════════════════════════════════════════════════
    public IActionResult Edit(int id)
    {
        var employee = GetEmployee(id);
        return View("EditForm", employee);  // Views/Employee/EditForm.cshtml
    }
    
    // ═══════════════════════════════════════════════════════════
    // METHOD 4: View in different folder
    // ═══════════════════════════════════════════════════════════
    public IActionResult Custom()
    {
        return View("~/Views/Shared/CustomView.cshtml");
    }
    
    // ═══════════════════════════════════════════════════════════
    // METHOD 5: Partial view
    // ═══════════════════════════════════════════════════════════
    public IActionResult LoadPartial(int id)
    {
        var employee = GetEmployee(id);
        return PartialView("_EmployeeCard", employee);
    }
}
```

---

## Code Examples

### Complete View Example

```html
@* File: Views/Employee/Index.cshtml *@

@model IEnumerable<Employee>

@{
    ViewData["Title"] = "Employees";
    var totalEmployees = Model.Count();
    var activeEmployees = Model.Count(e => e.IsActive);
}

<div class="container mt-4">
    <!-- Page Header -->
    <div class="row mb-3">
        <div class="col-md-8">
            <h1>@ViewData["Title"]</h1>
            <p class="text-muted">
                Total: @totalEmployees | Active: @activeEmployees
            </p>
        </div>
        <div class="col-md-4 text-end">
            <a asp-action="Create" class="btn btn-primary">
                <i class="bi bi-plus-circle"></i> Add New Employee
            </a>
        </div>
    </div>
    
    <!-- Search and Filter -->
    <div class="row mb-3">
        <div class="col-md-12">
            <form method="get" class="row g-3">
                <div class="col-md-4">
                    <input type="text" 
                           name="search" 
                           class="form-control" 
                           placeholder="Search by name or email" 
                           value="@ViewData["SearchTerm"]" />
                </div>
                <div class="col-md-3">
                    <select name="department" class="form-select">
                        <option value="">All Departments</option>
                        @foreach (var dept in ViewBag.Departments)
                        {
                            <option value="@dept.Id">@dept.Name</option>
                        }
                    </select>
                </div>
                <div class="col-md-2">
                    <button type="submit" class="btn btn-secondary">
                        <i class="bi bi-search"></i> Filter
                    </button>
                </div>
            </form>
        </div>
    </div>
    
    <!-- Employee Table -->
    <div class="row">
        <div class="col-md-12">
            @if (Model.Any())
            {
                <table class="table table-hover table-striped">
                    <thead class="table-dark">
                        <tr>
                            <th>Name</th>
                            <th>Email</th>
                            <th>Phone</th>
                            <th>Department</th>
                            <th>Status</th>
                            <th>Actions</th>
                        </tr>
                    </thead>
                    <tbody>
                        @foreach (var employee in Model)
                        {
                            <tr>
                                <td>
                                    <strong>@employee.Name</strong>
                                    @if (employee.IsManager)
                                    {
                                        <span class="badge bg-primary ms-2">Manager</span>
                                    }
                                </td>
                                <td>@employee.Email</td>
                                <td>@(employee.Phone ?? "N/A")</td>
                                <td>@employee.Department?.Name</td>
                                <td>
                                    @if (employee.IsActive)
                                    {
                                        <span class="badge bg-success">Active</span>
                                    }
                                    else
                                    {
                                        <span class="badge bg-secondary">Inactive</span>
                                    }
                                </td>
                                <td>
                                    <div class="btn-group" role="group">
                                        <a asp-action="Details" 
                                           asp-route-id="@employee.Id" 
                                           class="btn btn-sm btn-info"
                                           title="View Details">
                                            <i class="bi bi-eye"></i>
                                        </a>
                                        <a asp-action="Edit" 
                                           asp-route-id="@employee.Id" 
                                           class="btn btn-sm btn-warning"
                                           title="Edit">
                                            <i class="bi bi-pencil"></i>
                                        </a>
                                        <a asp-action="Delete" 
                                           asp-route-id="@employee.Id" 
                                           class="btn btn-sm btn-danger"
                                           title="Delete">
                                            <i class="bi bi-trash"></i>
                                        </a>
                                    </div>
                                </td>
                            </tr>
                        }
                    </tbody>
                </table>
            }
            else
            {
                <div class="alert alert-info">
                    <i class="bi bi-info-circle"></i>
                    No employees found. <a asp-action="Create">Create one now</a>.
                </div>
            }
        </div>
    </div>
</div>

@section Scripts {
    <script>
        $(document).ready(function() {
            console.log('Employee list loaded with @totalEmployees employees');
        });
    </script>
}
```

---

## Best Practices

### 1. Use Strongly-Typed Views

```html
<!-- ✅ GOOD: Strong typing with IntelliSense -->
@model Employee
<h2>@Model.Name</h2>

<!-- ❌ BAD: Weak typing, no IntelliSense -->
@{
    var emp = ViewData["Employee"] as Employee;
}
<h2>@emp.Name</h2>
```

### 2. Null Safety

```html
<!-- ✅ GOOD: Null-conditional operators -->
<p>Department: @Model.Department?.Name</p>
<p>Manager: @(Model.Manager?.Name ?? "No manager")</p>

<!-- ❌ BAD: Can throw NullReferenceException -->
<p>Department: @Model.Department.Name</p>
```

### 3. Keep Logic Minimal

```html
<!-- ✅ GOOD: Simple display logic -->
@if (Model.IsActive)
{
    <span class="badge bg-success">Active</span>
}

<!-- ❌ BAD: Complex business logic in view -->
@{
    var salary = Model.BaseSalary;
    if (Model.YearsOfService > 5)
        salary += 10000;
    if (Model.Department.Name == "IT")
        salary *= 1.2;
    // ... more complex calculations
}
```

---

## Interview Questions

### Q1: What is Razor syntax?
**Answer:** Razor is a markup syntax for embedding server-side code (C#) into web pages. It uses the `@` symbol to transition from HTML to C# code, making it easy to mix static HTML with dynamic content.

### Q2: What is the difference between @ and @@ in Razor?
**Answer:**
- `@` - Transitions from HTML to C# code
- `@@` - Outputs a literal @ symbol (escape sequence)

### Q3: What is the difference between HTML comments and Razor comments?
**Answer:**
- `<!-- HTML comment -->` - Sent to client, visible in page source
- `@* Razor comment *@` - Not sent to client, server-side only

### Q4: What is @model directive?
**Answer:** `@model` specifies the type of the model passed to the view, enabling strong typing and IntelliSense support for the `Model` property.

### Q5: How does view discovery work?
**Answer:** When a controller returns `View()`, ASP.NET Core searches:
1. `/Views/{ControllerName}/{ActionName}.cshtml`
2. `/Views/Shared/{ActionName}.cshtml`

### Q6: What is the difference between @{} and @()?
**Answer:**
- `@{}` - Code block for statements (doesn't output)
- `@()` - Explicit expression (outputs result)

---

## Quick Reference

### Razor Syntax Cheat Sheet

```html
@* Basic output *@
@Model.Property

@* Explicit expression *@
@(expression)

@* Code block *@
@{
    var x = 10;
}

@* If statement *@
@if (condition) { }

@* For loop *@
@for (int i = 0; i < 10; i++) { }

@* Foreach loop *@
@foreach (var item in Model) { }

@* Razor comment *@
@* This is a comment *@

@* Literal @ *@
Email: name@@domain.com
```

### Common Patterns

```html
@* Null-safe property access *@
@Model.Property?.NestedProperty

@* Null coalescing *@
@(Model.Property ?? "Default")

@* String interpolation *@
@{
    var message = $"Hello, {Model.Name}!";
}

@* Ternary operator *@
@(Model.IsActive ? "Yes" : "No")
```
