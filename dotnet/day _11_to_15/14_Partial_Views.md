# Partial Views in ASP.NET Core

## Table of Contents
1. [Introduction](#introduction)
2. [What are Partial Views?](#what-are-partial-views)
3. [Creating Partial Views](#creating-partial-views)
4. [Rendering Partial Views](#rendering-partial-views)
5. [Passing Data to Partial Views](#passing-data-to-partial-views)
6. [Partial Tag Helper](#partial-tag-helper)
7. [Async Partial Views](#async-partial-views)
8. [Code Examples](#code-examples)
9. [Best Practices](#best-practices)
10. [Interview Questions](#interview-questions)

---

## Introduction

### Real-World Analogy
Partial Views are like **reusable modules or components** in manufacturing:

```
Building a Car (Web Page)
    ├── Engine Component (Header Partial)
    ├── Wheels Component (Navigation Partial)
    ├── Dashboard Component (Search Form Partial)
    └── Seats Component (Footer Partial)

Each component:
✓ Can be manufactured separately
✓ Reused in different car models (pages)
✓ Easier to maintain and update
```

---

## What are Partial Views?

Partial Views are reusable view components that can be embedded in other views to avoid code duplication.

### Key Characteristics

| Feature | Description |
|---------|-------------|
| **File naming** | Conventionally start with underscore `_PartialName.cshtml` |
| **Location** | Usually in `/Views/Shared/` for global use |
| **Reusability** | Can be used across multiple views |
| **No layout** | Don't use layout pages |
| **Independent** | Can have their own model |

### Partial View vs Regular View

```
┌─────────────────────────────────────────────────────────────────┐
│                  REGULAR VIEW vs PARTIAL VIEW                    │
├────────────────────────────┬────────────────────────────────────┤
│     REGULAR VIEW           │        PARTIAL VIEW                │
├────────────────────────────┼────────────────────────────────────┤
│ Full HTML page             │ Fragment of HTML                   │
│ Uses layout page           │ No layout page                     │
│ Complete page              │ Embedded component                 │
│ Index.cshtml               │ _EmployeeCard.cshtml               │
│ Returns from controller    │ Included in other views            │
└────────────────────────────┴────────────────────────────────────┘
```

---

## Creating Partial Views

### Naming Convention

```
Views/
├── Shared/
│   ├── _EmployeeCard.cshtml      ← Partial view (underscore prefix)
│   ├── _ProductList.cshtml        ← Partial view
│   ├── _SearchForm.cshtml         ← Partial view
│   └── _Layout.cshtml              ← Layout (also uses underscore)
└── Employee/
    ├── Index.cshtml                ← Regular view
    └── _EmployeeDetails.cshtml     ← Controller-specific partial
```

### Creating a Simple Partial View

```html
<!-- File: Views/Shared/_EmployeeCard.cshtml -->

@model Employee

@* Line 1: Partial views typically don't have layout *@
@* They are fragments meant to be included in other views *@

<div class="card mb-3">
    <div class="card-header">
        <h5>@Model.Name</h5>
        @if (Model.IsActive)
        {
            <span class="badge bg-success">Active</span>
        }
    </div>
    <div class="card-body">
        <p><strong>Email:</strong> @Model.Email</p>
        <p><strong>Department:</strong> @Model.Department?.Name</p>
        <p><strong>Phone:</strong> @(Model.Phone ?? "N/A")</p>
    </div>
    <div class="card-footer">
        <a asp-action="Details" asp-route-id="@Model.Id" class="btn btn-sm btn-info">
            Details
        </a>
        <a asp-action="Edit" asp-route-id="@Model.Id" class="btn btn-sm btn-warning">
            Edit
        </a>
    </div>
</div>
```

---

## Rendering Partial Views

### Method 1: Partial Tag Helper (Recommended)

```html
<!-- File: Views/Employee/Index.cshtml -->

@model IEnumerable<Employee>

<h1>Employees</h1>

<div class="row">
    @foreach (var employee in Model)
    {
        <div class="col-md-4">
            <!-- Line 1: Partial tag helper -->
            <partial name="_EmployeeCard" model="employee" />
        </div>
    }
</div>
```

### Method 2: Html.PartialAsync()

```html
<!-- File: Views/Employee/Index.cshtml -->

@model IEnumerable<Employee>

<h1>Employees</h1>

<div class="row">
    @foreach (var employee in Model)
    {
        <div class="col-md-4">
            <!-- Line 1: Html.PartialAsync - async rendering -->
            @await Html.PartialAsync("_EmployeeCard", employee)
        </div>
    }
</div>
```

### Method 3: Html.RenderPartialAsync()

```html
<!-- File: Views/Employee/Index.cshtml -->

@model IEnumerable<Employee>

<h1>Employees</h1>

<div class="row">
    @foreach (var employee in Model)
    {
        <div class="col-md-4">
            <!-- Line 1: Html.RenderPartialAsync - writes directly to response -->
            @{ await Html.RenderPartialAsync("_EmployeeCard", employee); }
            <!-- Note: Must be in code block because it returns void -->
        </div>
    }
</div>
```

### Comparison of Rendering Methods

| Method | Return Type | Performance | Usage |
|--------|-------------|-------------|-------|
| `<partial>` | Tag Helper | Good | **Recommended** - Clean syntax |
| `Html.PartialAsync()` | Task<IHtmlContent> | Good | Returns string for manipulation |
| `Html.RenderPartialAsync()` | Task (void) | **Best** | Writes directly to output |

### Performance Note

```
Html.RenderPartialAsync() is slightly faster:
    ┌─────────────────────────────────────────┐
    │ Html.PartialAsync()                     │
    │   1. Renders to string                  │
    │   2. Returns IHtmlContent               │
    │   3. Written to response                │
    └─────────────────────────────────────────┘

    ┌─────────────────────────────────────────┐
    │ Html.RenderPartialAsync()               │
    │   1. Writes directly to response stream │
    │   (Skips intermediate string)           │
    └─────────────────────────────────────────┘
```

---

## Passing Data to Partial Views

### Passing a Model

```html
<!-- Parent View: Views/Employee/Index.cshtml -->
@model IEnumerable<Employee>

@foreach (var employee in Model)
{
    <!-- Line 1: Pass specific employee as model -->
    <partial name="_EmployeeCard" model="employee" />
}
```

### Passing ViewData

```html
<!-- Parent View -->
@{
    ViewData["Title"] = "Employee List";
    ViewData["Department"] = "IT";
}

<!-- Line 1: Partial inherits parent's ViewData -->
<partial name="_Header" />

<!-- Partial View: _Header.cshtml -->
<h1>@ViewData["Title"]</h1>
<p>Department: @ViewData["Department"]</p>
```

### Using view-data Attribute

```html
<!-- Parent View -->
@{
    var customData = new ViewDataDictionary(ViewData)
    {
        { "ShowActions", true },
        { "Theme", "dark" }
    };
}

<!-- Line 1: Pass custom ViewData -->
<partial name="_EmployeeCard" model="employee" view-data="customData" />

<!-- Partial View: _EmployeeCard.cshtml -->
@model Employee

<div class="card @(ViewData["Theme"] == "dark" ? "bg-dark text-white" : "")">
    <div class="card-body">
        <h5>@Model.Name</h5>
        
        @if ((bool?)ViewData["ShowActions"] == true)
        {
            <a asp-action="Edit" asp-route-id="@Model.Id">Edit</a>
        }
    </div>
</div>
```

---

## Partial Tag Helper

### Basic Syntax

```html
<!-- ═══════════════════════════════════════════════════════════ -->
<!-- PARTIAL TAG HELPER ATTRIBUTES -->
<!-- ═══════════════════════════════════════════════════════════ -->

<!-- Line 1: name - Required, specifies partial view name -->
<partial name="_EmployeeCard" />

<!-- Line 2: model - Optional, passes model to partial -->
<partial name="_EmployeeCard" model="employee" />

<!-- Line 3: view-data - Optional, passes ViewData dictionary -->
<partial name="_EmployeeCard" view-data="customViewData" />

<!-- Line 4: for - Use with asp-for for model binding -->
<partial name="_AddressForm" for="Address" />

<!-- Line 5: Combination -->
<partial name="_EmployeeCard" 
         model="employee" 
         view-data="customData" />
```

### Partial Discovery

```
Partial View Search Order:
──────────────────────────────────────────────────────────────
<partial name="_EmployeeCard" />
from /Views/Employee/Index.cshtml

1. /Views/Employee/_EmployeeCard.cshtml    ← Same folder
2. /Views/Shared/_EmployeeCard.cshtml      ← Shared folder

<partial name="~/Views/Custom/_MyPartial.cshtml" />
                     ↑
            Explicit path (starts with ~/)
──────────────────────────────────────────────────────────────
```

---

## Async Partial Views

### Why Use Async?

Async rendering prevents blocking the UI thread while waiting for data.

```csharp
// File: Controllers/EmployeeController.cs

public class EmployeeController : Controller
{
    private readonly IEmployeeService _service;
    
    public EmployeeController(IEmployeeService service)
    {
        _service = service;
    }
    
    // ═══════════════════════════════════════════════════════════
    // Return partial view result
    // ═══════════════════════════════════════════════════════════
    public async Task<IActionResult> LoadEmployeeCard(int id)
    {
        var employee = await _service.GetEmployeeAsync(id);
        
        // Line 1: Return partial view
        return PartialView("_EmployeeCard", employee);
    }
}
```

### Using with AJAX

```html
<!-- Parent View: Views/Employee/Index.cshtml -->

<div id="employee-container">
    <button onclick="loadEmployee(5)" class="btn btn-primary">
        Load Employee
    </button>
</div>

<script>
    function loadEmployee(id) {
        // Line 1: AJAX call to load partial view
        $.ajax({
            url: '@Url.Action("LoadEmployeeCard", "Employee")',
            type: 'GET',
            data: { id: id },
            success: function(result) {
                // Line 2: Insert partial view HTML
                $('#employee-container').html(result);
            },
            error: function(xhr, status, error) {
                console.error('Error loading employee:', error);
            }
        });
    }
</script>
```

---

## Code Examples

### Complete Example: Employee List with Cards

```html
<!-- File: Views/Employee/Index.cshtml -->

@model IEnumerable<Employee>

@{
    ViewData["Title"] = "Employees";
}

<div class="container mt-4">
    <!-- Header -->
    <div class="row mb-3">
        <div class="col-md-8">
            <h1>@ViewData["Title"]</h1>
        </div>
        <div class="col-md-4 text-end">
            <a asp-action="Create" class="btn btn-primary">
                Add Employee
            </a>
        </div>
    </div>
    
    <!-- Employee Cards using Partial View -->
    <div class="row">
        @if (Model.Any())
        {
            @foreach (var employee in Model)
            {
                <div class="col-md-4 mb-3">
                    <!-- Render partial view for each employee -->
                    <partial name="_EmployeeCard" model="employee" />
                </div>
            }
        }
        else
        {
            <div class="col-12">
                <div class="alert alert-info">
                    No employees found.
                </div>
            </div>
        }
    </div>
</div>
```

### Reusable Search Form Partial

```html
<!-- File: Views/Shared/_SearchForm.cshtml -->

@model SearchViewModel

<form method="get" class="card">
    <div class="card-body">
        <h5 class="card-title">Search Employees</h5>
        
        <div class="row g-3">
            <div class="col-md-6">
                <label asp-for="SearchTerm" class="form-label"></label>
                <input asp-for="SearchTerm" class="form-control" placeholder="Name or email" />
            </div>
            
            <div class="col-md-4">
                <label asp-for="DepartmentId" class="form-label"></label>
                <select asp-for="DepartmentId" 
                        asp-items="Model.Departments" 
                        class="form-select">
                    <option value="">All Departments</option>
                </select>
            </div>
            
            <div class="col-md-2 d-flex align-items-end">
                <button type="submit" class="btn btn-primary w-100">
                    <i class="bi bi-search"></i> Search
                </button>
            </div>
        </div>
    </div>
</form>
```

```html
<!-- Usage in multiple views -->

<!-- In Index.cshtml -->
<partial name="_SearchForm" model="searchModel" />

<!-- In Archive.cshtml -->
<partial name="_SearchForm" model="searchModel" />

<!-- In Reports.cshtml -->
<partial name="_SearchForm" model="searchModel" />
```

### Dynamic Partial Loading

```html
<!-- Parent View -->
<div class="row">
    <div class="col-md-8">
        <!-- Main content -->
        <h2>Employee Details</h2>
        <div id="employee-details">
            <p>Select an employee from the sidebar</p>
        </div>
    </div>
    
    <div class="col-md-4">
        <!-- Sidebar with employee list -->
        <div class="list-group">
            @foreach (var emp in Model)
            {
                <a href="#" 
                   class="list-group-item list-group-item-action"
                   onclick="loadDetails(@emp.Id); return false;">
                    @emp.Name
                </a>
            }
        </div>
    </div>
</div>

<script>
    function loadDetails(id) {
        // Load partial view via AJAX
        fetch(`/Employee/GetDetailsPartial/${id}`)
            .then(response => response.text())
            .then(html => {
                document.getElementById('employee-details').innerHTML = html;
            })
            .catch(error => console.error('Error:', error));
    }
</script>
```

```csharp
// Controller action for AJAX
public async Task<IActionResult> GetDetailsPartial(int id)
{
    var employee = await _service.GetEmployeeAsync(id);
    return PartialView("_EmployeeDetails", employee);
}
```

---

## Best Practices

### 1. Use Underscore Naming Convention

```
✅ GOOD: Clear naming
_EmployeeCard.cshtml
_SearchForm.cshtml
_ProductList.cshtml

❌ BAD: No underscore
EmployeeCard.cshtml (looks like regular view)
```

### 2. Keep Partials Focused

```html
<!-- ✅ GOOD: Single responsibility -->
<!-- _EmployeeCard.cshtml - Only displays employee card -->

<!-- ❌ BAD: Multiple responsibilities -->
<!-- _EmployeeCardAndSearchAndFilter.cshtml - Too complex! -->
```

### 3. Place Shared Partials in /Shared/

```
✅ GOOD:
Views/Shared/_EmployeeCard.cshtml  ← Reusable across all controllers

❌ BAD:
Views/Employee/_EmployeeCard.cshtml ← Only if specific to Employee
                                     and not reused elsewhere
```

### 4. Use Partial Tag Helper

```html
<!-- ✅ GOOD: Modern, clean syntax -->
<partial name="_EmployeeCard" model="employee" />

<!-- ⚠️ ACCEPTABLE: But more verbose -->
@await Html.PartialAsync("_EmployeeCard", employee)

<!-- ❌ OLD: Synchronous, blocking -->
@Html.Partial("_EmployeeCard", employee)
```

---

## Interview Questions

### Q1: What is a Partial View?
**Answer:** A Partial View is a reusable view component that renders a fragment of HTML. It doesn't use a layout page and is meant to be embedded within other views to avoid code duplication.

### Q2: What is the naming convention for Partial Views?
**Answer:** Partial Views conventionally start with an underscore (_) prefix, e.g., `_EmployeeCard.cshtml`. This distinguishes them from regular views and indicates they are not meant to be rendered directly.

### Q3: What are the different ways to render a Partial View?
**Answer:**
1. `<partial name="_Name" model="model" />` - Tag Helper (recommended)
2. `@await Html.PartialAsync("_Name", model)` - Returns IHtmlContent
3. `@{ await Html.RenderPartialAsync("_Name", model); }` - Writes directly to response (fastest)

### Q4: What is the difference between PartialAsync and RenderPartialAsync?
**Answer:**
- `PartialAsync()` - Returns `Task<IHtmlContent>`, can be assigned to variable
- `RenderPartialAsync()` - Returns `Task` (void), writes directly to response stream (slightly better performance)

### Q5: Can a Partial View have its own model?
**Answer:** Yes, a Partial View can have its own strongly-typed model different from the parent view by using `@model` directive.

### Q6: Where does ASP.NET Core look for Partial Views?
**Answer:** Search order:
1. `/Views/{ControllerName}/{PartialName}.cshtml`
2. `/Views/Shared/{PartialName}.cshtml`
Or use explicit path: `~/Views/Custom/_Partial.cshtml`

---

## Quick Reference

### Rendering Methods

```html
<!-- Partial Tag Helper (Recommended) -->
<partial name="_Name" model="model" />

<!-- Html.PartialAsync -->
@await Html.PartialAsync("_Name", model)

<!-- Html.RenderPartialAsync (Fastest) -->
@{ await Html.RenderPartialAsync("_Name", model); }
```

### Passing Data

```html
<!-- Pass model only -->
<partial name="_Name" model="employee" />

<!-- Pass model + ViewData -->
<partial name="_Name" model="employee" view-data="customData" />

<!-- Use asp-for -->
<partial name="_AddressForm" for="Address" />
```

### File Structure

```
Views/
├── Shared/
│   ├── _EmployeeCard.cshtml     ← Global partial
│   ├── _SearchForm.cshtml       ← Global partial
│   └── _Layout.cshtml
└── Employee/
    ├── Index.cshtml             ← Regular view
    └── _EmployeeRow.cshtml      ← Controller-specific partial
```
