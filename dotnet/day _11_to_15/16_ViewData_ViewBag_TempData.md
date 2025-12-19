# ViewData, ViewBag, and TempData in ASP.NET Core

## Table of Contents
1. [Introduction](#introduction)
2. [Data Passing Mechanisms](#data-passing-mechanisms)
3. [ViewData](#viewdata)
4. [ViewBag](#viewbag)
5. [TempData](#tempdata)
6. [Comparison](#comparison)
7. [When to Use Each](#when-to-use-each)
8. [Code Examples](#code-examples)
9. [Best Practices](#best-practices)
10. [Interview Questions](#interview-questions)

---

## Introduction

### Real-World Analogy
Data passing mechanisms are like different **mail delivery methods**:

```
ViewData    →  Sticky Note (key-value dictionary)
                ├── Must know key name
                ├── Needs type casting
                └── One-way delivery (controller→view)

ViewBag     →  Smart Envelope (dynamic wrapper)
                ├── Property-based access
                ├── No casting needed
                └── One-way delivery (controller→view)

TempData    →  Express Mail (survives redirect)
                ├── Persists across requests
                ├── Auto-deleted after read
                └── Two-way delivery (controller↔controller)
```

---

## Data Passing Mechanisms

### Overview

| Mechanism | Type | Lifetime | Use Case |
|-----------|------|----------|----------|
| **ViewData** | Dictionary | Single request | Controller → View |
| **ViewBag** | Dynamic | Single request | Controller → View |
| **TempData** | Dictionary | Across redirects | Controller → Controller |
| **Model** | Strongly-typed | Single request | **Preferred** for view data |

### Data Flow Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                  DATA PASSING MECHANISMS                         │
└─────────────────────────────────────────────────────────────────┘

Request 1: /Employee/Create (GET)
────────────────────────────────────────────────────────────────
Controller Action
    │
    ├──→ ViewData["Message"] = "Hello"      ← Dies after response
    ├──→ ViewBag.Title = "Create"           ← Dies after response
    └──→ TempData["Success"] = "Saved!"     ← SURVIVES
                │
                ▼
              View
        (Renders response)
                │
                ▼
     Response sent to client

Request 2: /Employee/Index (Redirect)
────────────────────────────────────────────────────────────────
Controller Action
    │
    ├──→ ViewData["Message"]     ← NULL (new request)
    ├──→ ViewBag.Title          ← NULL (new request)
    └──→ TempData["Success"]    ← "Saved!" (still available!)
                │
                ▼
              View
        (Can access TempData)
```

---

## ViewData

### What is ViewData?

ViewData is a **dictionary** that passes data from controller to view. It's a property of `ControllerBase` class.

```csharp
// Definition
public ViewDataDictionary ViewData { get; set; }
```

### Using ViewData

```csharp
// File: Controllers/EmployeeController.cs

public class EmployeeController : Controller
{
    // ═══════════════════════════════════════════════════════════
    // SETTING ViewData in Controller
    // ═══════════════════════════════════════════════════════════
    
    public IActionResult Index()
    {
        // Line 1: Store string
        ViewData["Message"] = "Welcome to Employee Management";
        
        // Line 2: Store number (boxed as object)
        ViewData["EmployeeCount"] = 150;
        
        // Line 3: Store complex object
        ViewData["CurrentUser"] = new User 
        { 
            Name = "John Doe", 
            Role = "Admin" 
        };
        
        // Line 4: Store list
        ViewData["Departments"] = new List<string> 
        { 
            "IT", "HR", "Finance" 
        };
        
        return View();
    }
    
    public IActionResult Create()
    {
        // Line 5: Common use - populate dropdown
        var departments = _repository.GetAllDepartment();
        ViewData["DepartmentId"] = new SelectList(
            departments, "Id", "Name");
        
        return View();
    }
}
```

```html
<!-- File: Views/Employee/Index.cshtml -->

<!-- ═══════════════════════════════════════════════════════════ -->
<!-- READING ViewData in View -->
<!-- ═══════════════════════════════════════════════════════════ -->

<!-- Line 1: Read string (explicit cast) -->
<h1>@ViewData["Message"]</h1>

<!-- Line 2: Read number (requires casting) -->
<p>Total Employees: @((int)ViewData["EmployeeCount"])</p>

<!-- Line 3: Read complex object (requires casting) -->
@{
    var user = ViewData["CurrentUser"] as User;
}
<p>Current User: @user?.Name (@user?.Role)</p>

<!-- Line 4: Read list (requires casting) -->
@{
    var departments = ViewData["Departments"] as List<string>;
}
<ul>
    @foreach (var dept in departments)
    {
        <li>@dept</li>
    }
</ul>

<!-- Line 5: Using dropdown -->
<select asp-items="ViewData["DepartmentId"] as SelectList"></select>
```

### ViewData Characteristics

```
┌─────────────────────────────────────────────────────────────────┐
│                     ViewData PROPERTIES                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Type:          ViewDataDictionary                              │
│  Base Type:     Dictionary<string, object>                      │
│  Key:           String                                          │
│  Value:         Object (requires casting)                       │
│  Lifetime:      Current request only                            │
│  Null Check:    Required before use                             │
│  IntelliSense:  No (string keys)                                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## ViewBag

### What is ViewBag?

ViewBag is a **dynamic wrapper** around ViewData that allows property-style access without casting.

```csharp
// ViewBag is just dynamic access to ViewData
// ViewBag.Title is equivalent to ViewData["Title"]
```

### Using ViewBag

```csharp
// File: Controllers/EmployeeController.cs

public class EmployeeController : Controller
{
    // ═══════════════════════════════════════════════════════════
    // SETTING ViewBag in Controller
    // ═══════════════════════════════════════════════════════════
    
    public IActionResult Index()
    {
        // Line 1: Store string (no quotes needed for property)
        ViewBag.Message = "Welcome to Employee Management";
        
        // Line 2: Store number (no boxing/casting needed)
        ViewBag.EmployeeCount = 150;
        
        // Line 3: Store complex object
        ViewBag.CurrentUser = new User 
        { 
            Name = "John Doe", 
            Role = "Admin" 
        };
        
        // Line 4: Store list
        ViewBag.Departments = new List<string> 
        { 
            "IT", "HR", "Finance" 
        };
        
        // Line 5: Create dynamic properties on the fly
        ViewBag.Title = "Employee List";
        ViewBag.ShowFilter = true;
        ViewBag.PageSize = 10;
        
        return View();
    }
}
```

```html
<!-- File: Views/Employee/Index.cshtml -->

<!-- ═══════════════════════════════════════════════════════════ -->
<!-- READING ViewBag in View -->
<!-- ═══════════════════════════════════════════════════════════ -->

<!-- Line 1: Read string (no casting) -->
<h1>@ViewBag.Message</h1>

<!-- Line 2: Read number (no casting) -->
<p>Total Employees: @ViewBag.EmployeeCount</p>

<!-- Line 3: Read complex object (direct access) -->
<p>Current User: @ViewBag.CurrentUser.Name (@ViewBag.CurrentUser.Role)</p>

<!-- Line 4: Read list (no casting) -->
<ul>
    @foreach (var dept in ViewBag.Departments)
    {
        <li>@dept</li>
    }
</ul>

<!-- Line 5: Conditional rendering -->
@if (ViewBag.ShowFilter == true)
{
    <div class="filter-section">
        <!-- Filter UI -->
    </div>
}
```

### ViewBag vs ViewData

```csharp
// These are EXACTLY THE SAME:

// ViewBag (dynamic property)
ViewBag.Message = "Hello";
var message = ViewBag.Message;

// ViewData (dictionary)
ViewData["Message"] = "Hello";
var message = ViewData["Message"];

// ViewBag is just syntactic sugar over ViewData!
```

---

## TempData

### What is TempData?

TempData stores data that **survives a single redirect** and is automatically deleted after being read.

```csharp
// TempData is backed by session state by default
// Persists across requests until read
```

### Using TempData

```csharp
// File: Controllers/EmployeeController.cs

public class EmployeeController : Controller
{
    // ═══════════════════════════════════════════════════════════
    // POST action - Create employee
    // ═══════════════════════════════════════════════════════════
    
    [HttpPost]
    public IActionResult Create(Employee employee)
    {
        if (ModelState.IsValid)
        {
            _repository.Add(employee);
            
            // Line 1: Store success message in TempData
            TempData["SuccessMessage"] = $"Employee {employee.Name} created successfully!";
            
            // Line 2: Redirect to Index
            // TempData SURVIVES this redirect
            return RedirectToAction(nameof(Index));
        }
        
        // Line 3: Store error message
        TempData["ErrorMessage"] = "Please correct the errors and try again.";
        
        return View(employee);
    }
    
    // ═══════════════════════════════════════════════════════════
    // GET action - Index
    // ═══════════════════════════════════════════════════════════
    
    public IActionResult Index()
    {
        var employees = _repository.GetAllEmployee();
        
        // TempData["SuccessMessage"] is available here!
        // After reading, it will be automatically deleted
        
        return View(employees);
    }
    
    // ═══════════════════════════════════════════════════════════
    // Keeping TempData for another request
    // ═══════════════════════════════════════════════════════════
    
    public IActionResult SomeAction()
    {
        // Line 4: Read without deleting (using Peek)
        var message = TempData.Peek("Message");
        // TempData["Message"] still available for next request
        
        // Line 5: Keep for another request
        TempData.Keep("Message");
        // TempData["Message"] will survive one more request
        
        return View();
    }
}
```

```html
<!-- File: Views/Employee/Index.cshtml -->

<!-- ═══════════════════════════════════════════════════════════ -->
<!-- READING TempData in View -->
<!-- ═══════════════════════════════════════════════════════════ -->

<!-- Line 1: Check and display success message -->
@if (TempData["SuccessMessage"] != null)
{
    <div class="alert alert-success alert-dismissible fade show">
        <strong>Success!</strong> @TempData["SuccessMessage"]
        <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
    </div>
}

<!-- Line 2: Check and display error message -->
@if (TempData["ErrorMessage"] != null)
{
    <div class="alert alert-danger alert-dismissible fade show">
        <strong>Error!</strong> @TempData["ErrorMessage"]
        <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
    </div>
}
```

### TempData Lifecycle

```
┌─────────────────────────────────────────────────────────────────┐
│                    TempData LIFECYCLE                            │
└─────────────────────────────────────────────────────────────────┘

Request 1: POST /Employee/Create
────────────────────────────────────────
  TempData["Message"] = "Success!"
  return RedirectToAction("Index")
        │
        │ TempData stored in session
        ▼

Request 2: GET /Employee/Index (Redirect)
────────────────────────────────────────
  View reads: @TempData["Message"]
  Displays: "Success!"
        │
        │ TempData marked for deletion
        ▼

Request 3: GET /Employee/Details/5
────────────────────────────────────────
  TempData["Message"] = null
  (Already deleted after being read)
```

### TempData Methods

```csharp
// Line 1: Normal access (read and mark for deletion)
var value = TempData["Key"];

// Line 2: Peek (read without marking for deletion)
var value = TempData.Peek("Key");

// Line 3: Keep (prevent deletion for one more request)
TempData.Keep("Key");

// Line 4: Keep all
TempData.Keep();

// Line 5: Check if exists
if (TempData.ContainsKey("Key"))
{
    // Use TempData["Key"]
}
```

---

## Comparison

### Side-by-Side Comparison

```csharp
// ═══════════════════════════════════════════════════════════════
// VIEWDATA
// ═══════════════════════════════════════════════════════════════

// Controller
ViewData["Message"] = "Hello";
ViewData["Count"] = 10;

// View (requires casting)
var message = ViewData["Message"] as string;
var count = (int)ViewData["Count"];

// ═══════════════════════════════════════════════════════════════
// VIEWBAG
// ═══════════════════════════════════════════════════════════════

// Controller
ViewBag.Message = "Hello";
ViewBag.Count = 10;

// View (no casting)
var message = ViewBag.Message;
var count = ViewBag.Count;

// ═══════════════════════════════════════════════════════════════
// TEMPDATA
// ═══════════════════════════════════════════════════════════════

// Controller (Request 1)
TempData["Message"] = "Hello";
return RedirectToAction("Index");

// Controller (Request 2 - after redirect)
var message = TempData["Message"];  // Still available!
```

### Comparison Table

| Feature | ViewData | ViewBag | TempData |
|---------|----------|---------|----------|
| **Type** | Dictionary | Dynamic | Dictionary |
| **Syntax** | `["key"]` | `.Property` | `["key"]` |
| **Casting** | Required | Not required | Required |
| **Lifetime** | Current request | Current request | Across redirect |
| **IntelliSense** | No | No | No |
| **Null Safety** | Check required | Check required | Check required |
| **Performance** | Fast | Fast | Slightly slower |
| **Storage** | In-memory | In-memory | Session state |

---

## When to Use Each

### ViewData/ViewBag Use Cases

```csharp
// ✅ GOOD USE CASES:

// 1. Page title
ViewBag.Title = "Employee List";

// 2. Dropdown lists
ViewData["DepartmentId"] = new SelectList(departments, "Id", "Name");

// 3. Small metadata
ViewBag.ShowFilter = true;
ViewBag.PageNumber = 1;

// 4. CSS classes or UI flags
ViewData["ActiveTab"] = "employees";
```

### TempData Use Cases

```csharp
// ✅ GOOD USE CASES:

// 1. Success/Error messages after redirect
TempData["SuccessMessage"] = "Employee created successfully!";
return RedirectToAction("Index");

// 2. Passing data between actions
TempData["SearchCriteria"] = searchModel;
return RedirectToAction("Results");

// 3. Wizard-style multi-step forms
TempData["Step1Data"] = formData;
return RedirectToAction("Step2");

// 4. Confirmation dialogs
TempData["ConfirmDelete"] = employeeId;
return RedirectToAction("DeleteConfirmation");
```

### When NOT to Use

```csharp
// ❌ BAD: Use strongly-typed Model instead

// BAD - ViewBag for main data
ViewBag.Employee = employee;
ViewBag.Departments = departments;
return View();

// GOOD - Strongly-typed Model
return View(employee);

// GOOD - ViewModel for complex data
var viewModel = new EmployeeViewModel
{
    Employee = employee,
    Departments = departments
};
return View(viewModel);
```

---

## Code Examples

### Complete Example: CRUD with Messages

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
    // INDEX - Display list
    // ═══════════════════════════════════════════════════════════
    public IActionResult Index()
    {
        // ViewBag for metadata
        ViewBag.Title = "Employees";
        ViewBag.TotalCount = _service.GetCount();
        
        var employees = _service.GetAllEmployee();
        
        // TempData messages displayed here (from Create/Edit/Delete)
        return View(employees);
    }
    
    // ═══════════════════════════════════════════════════════════
    // CREATE - GET
    // ═══════════════════════════════════════════════════════════
    public IActionResult Create()
    {
        // ViewData for dropdown
        var departments = _service.GetAllDepartment();
        ViewData["DepartmentId"] = new SelectList(departments, "Id", "Name");
        
        ViewBag.Title = "Create Employee";
        return View();
    }
    
    // ═══════════════════════════════════════════════════════════
    // CREATE - POST
    // ═══════════════════════════════════════════════════════════
    [HttpPost]
    [ValidateAntiForgeryToken]
    public IActionResult Create(Employee employee)
    {
        if (ModelState.IsValid)
        {
            _service.Add(employee);
            
            // TempData survives redirect
            TempData["SuccessMessage"] = $"Employee '{employee.Name}' created successfully!";
            TempData["EmployeeId"] = employee.Id;
            
            return RedirectToAction(nameof(Index));
        }
        
        // Reload dropdown on validation failure
        var departments = _service.GetAllDepartment();
        ViewData["DepartmentId"] = new SelectList(departments, "Id", "Name");
        
        // TempData for error
        TempData["ErrorMessage"] = "Please correct the errors.";
        
        return View(employee);
    }
    
    // ═══════════════════════════════════════════════════════════
    // EDIT - POST
    // ═══════════════════════════════════════════════════════════
    [HttpPost]
    public IActionResult Edit(Employee employee)
    {
        if (ModelState.IsValid)
        {
            _service.Update(employee);
            
            TempData["SuccessMessage"] = "Employee updated successfully!";
            return RedirectToAction(nameof(Index));
        }
        
        TempData["ErrorMessage"] = "Update failed. Please try again.";
        return View(employee);
    }
    
    // ═══════════════════════════════════════════════════════════
    // DELETE - POST
    // ═══════════════════════════════════════════════════════════
    [HttpPost]
    public IActionResult Delete(int id)
    {
        try
        {
            var employee = _service.GetEmployee(id);
            _service.Delete(id);
            
            TempData["SuccessMessage"] = $"Employee '{employee.Name}' deleted successfully!";
        }
        catch (Exception ex)
        {
            TempData["ErrorMessage"] = $"Error deleting employee: {ex.Message}";
        }
        
        return RedirectToAction(nameof(Index));
    }
}
```

```html
<!-- File: Views/Shared/_Layout.cshtml -->

<!DOCTYPE html>
<html>
<head>
    <!-- Line 1: ViewBag for title -->
    <title>@ViewBag.Title - Employee Management</title>
</head>
<body>
    <main>
        <div class="container">
            <!-- Line 2: TempData messages (global) -->
            @if (TempData["SuccessMessage"] != null)
            {
                <div class="alert alert-success alert-dismissible fade show mt-3">
                    <i class="bi bi-check-circle"></i>
                    @TempData["SuccessMessage"]
                    <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
                </div>
            }
            
            @if (TempData["ErrorMessage"] != null)
            {
                <div class="alert alert-danger alert-dismissible fade show mt-3">
                    <i class="bi bi-exclamation-triangle"></i>
                    @TempData["ErrorMessage"]
                    <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
                </div>
            }
            
            @RenderBody()
        </div>
    </main>
</body>
</html>
```

---

## Best Practices

### 1. Prefer Strongly-Typed Models

```csharp
// ❌ BAD: Loosely-typed data
ViewBag.Employee = employee;
ViewBag.Departments = departments;

// ✅ GOOD: Strongly-typed ViewModel
public class EmployeeViewModel
{
    public Employee Employee { get; set; }
    public List<Department> Departments { get; set; }
}

return View(viewModel);
```

### 2. Always Check for Null

```html
<!-- ❌ BAD: No null check -->
<p>@ViewBag.Message</p>

<!-- ✅ GOOD: Null check -->
@if (ViewBag.Message != null)
{
    <p>@ViewBag.Message</p>
}
```

### 3. Use TempData for Post-Redirect-Get Pattern

```csharp
// ✅ GOOD: PRG pattern with TempData
[HttpPost]
public IActionResult Create(Employee emp)
{
    _service.Add(emp);
    TempData["Success"] = "Created!";
    return RedirectToAction("Index");  // Prevents double-submit
}
```

### 4. Use ViewData for Dropdowns

```csharp
// ✅ GOOD: SelectList in ViewData
ViewData["DepartmentId"] = new SelectList(departments, "Id", "Name");

// In View:
<select asp-for="DepartmentId" asp-items="ViewData["DepartmentId"] as SelectList">
</select>
```

---

## Interview Questions

### Q1: What is the difference between ViewData and ViewBag?
**Answer:** 
- **ViewData**: Dictionary-based, requires casting, uses `["key"]` syntax
- **ViewBag**: Dynamic wrapper around ViewData, no casting needed, uses `.Property` syntax
Both have the same lifetime (current request only).

### Q2: What is TempData and when should you use it?
**Answer:** TempData stores data that survives a redirect (one additional request). Use it for success/error messages after POST-Redirect-GET pattern, passing data between actions, or multi-step forms.

### Q3: How long does TempData persist?
**Answer:** TempData persists until it's read. After being read once, it's marked for deletion and will be removed after the current request completes.

### Q4: What is the difference between TempData.Peek() and TempData.Keep()?
**Answer:**
- `Peek("key")`: Read value without marking for deletion
- `Keep("key")`: Mark already-read value to persist for one more request

### Q5: Which is better: ViewBag or ViewData?
**Answer:** Neither is inherently better - they're the same underneath. ViewBag has cleaner syntax but no compile-time checking. However, **strongly-typed models are preferred** over both for main data.

### Q6: Where is TempData stored?
**Answer:** By default, TempData is stored in session state (server-side). In ASP.NET Core, you can configure alternative providers like cookies.

---

## Quick Reference

### ViewData

```csharp
// Set
ViewData["Key"] = value;

// Get (requires casting)
var value = ViewData["Key"] as Type;
var number = (int)ViewData["Number"];
```

### ViewBag

```csharp
// Set
ViewBag.Property = value;

// Get (no casting)
var value = ViewBag.Property;
```

### TempData

```csharp
// Set
TempData["Key"] = value;

// Get (deleted after read)
var value = TempData["Key"];

// Peek (not deleted)
var value = TempData.Peek("Key");

// Keep
TempData.Keep("Key");
```

### Typical Usage

```csharp
// Controller
ViewBag.Title = "Page Title";
ViewData["Dropdown"] = new SelectList(...);
TempData["Message"] = "Success!";
return View(model);  // Strongly-typed model preferred
```
