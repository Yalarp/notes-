# MVC Architecture Pattern in ASP.NET Core

## Table of Contents
1. [Introduction](#introduction)
2. [What is MVC?](#what-is-mvc)
3. [MVC Components](#mvc-components)
4. [Request Flow in MVC](#request-flow-in-mvc)
5. [Model in Detail](#model-in-detail)
6. [View in Detail](#view-in-detail)
7. [Controller in Detail](#controller-in-detail)
8. [Complete Code Examples](#complete-code-examples)
9. [Separation of Concerns](#separation-of-concerns)
10. [Best Practices](#best-practices)
11. [Interview Questions](#interview-questions)

---

## Introduction

### Real-World Analogy
Think of MVC like a **restaurant**:
- **Model** = The kitchen and ingredients (data and business logic)
- **View** = The plate presentation and menu (what customer sees)
- **Controller** = The waiter (handles requests, coordinates between kitchen and customer)

```
Customer (User)
    │
    ▼
┌─────────┐     "I want pasta"     ┌─────────────┐
│ Waiter  │ ◄──────────────────────│   Menu      │
│(Control)│                        │   (View)    │
└────┬────┘                        └─────────────┘
     │ "Make pasta order"
     ▼
┌─────────────┐
│   Kitchen   │
│   (Model)   │
│ - Recipes   │
│ - Cooking   │
└─────────────┘
```

### Why MVC Matters
- **Separation of Concerns**: Each component has a single responsibility
- **Testability**: Components can be tested independently
- **Maintainability**: Easy to modify one part without affecting others
- **Parallel Development**: Frontend and backend teams can work simultaneously
- **Reusability**: Models and views can be reused across different parts

---

## What is MVC?

**MVC (Model-View-Controller)** is an architectural pattern that divides an application into three interconnected components:

```
┌─────────────────────────────────────────────────────────────────┐
│                        MVC PATTERN                               │
├──────────────────┬──────────────────┬──────────────────────────┤
│      MODEL       │       VIEW       │       CONTROLLER          │
│                  │                  │                           │
│ • Data           │ • UI/Display     │ • Handles Requests        │
│ • Business Logic │ • HTML/Razor     │ • Coordinates M & V       │
│ • Validation     │ • User Interface │ • Returns Responses       │
│ • Database       │ • Presentation   │ • Application Flow        │
└──────────────────┴──────────────────┴──────────────────────────┘
```

### MVC Flow Diagram

```
┌──────────────────────────────────────────────────────────────────┐
│                      USER REQUEST                                 │
│                    (HTTP GET /Employee/Index)                     │
└───────────────────────────────┬──────────────────────────────────┘
                                │
                                ▼
┌──────────────────────────────────────────────────────────────────┐
│                      CONTROLLER                                   │
│                 (EmployeeController.Index())                      │
│                                                                   │
│  1. Receives request                                              │
│  2. Calls Model to get data                                       │
│  3. Selects View to display                                       │
│  4. Passes data to View                                           │
└───────────────────────────────┬──────────────────────────────────┘
                                │
              ┌─────────────────┴─────────────────┐
              │                                   │
              ▼                                   ▼
┌─────────────────────────┐         ┌─────────────────────────────┐
│         MODEL           │         │           VIEW              │
│   (Employee.cs)         │         │     (Index.cshtml)          │
│                         │         │                             │
│ • Employee data         │         │ • Renders HTML              │
│ • GetAllEmployees()     │────────▶│ • Displays employee list    │
│ • Business rules        │  Data   │ • Uses Razor syntax         │
└─────────────────────────┘         └──────────────┬──────────────┘
                                                   │
                                                   ▼
                                    ┌─────────────────────────────┐
                                    │      HTTP RESPONSE          │
                                    │      (HTML Page)            │
                                    └─────────────────────────────┘
```

---

## MVC Components

### Component Responsibilities

| Component | Responsibility | Example Files |
|-----------|---------------|---------------|
| **Model** | Data + Business Logic | `Employee.cs`, `Department.cs` |
| **View** | UI Presentation | `Index.cshtml`, `Create.cshtml` |
| **Controller** | Request Handling | `EmployeeController.cs` |

### Detailed Breakdown

```
ASP.NET Core MVC Application
│
├── Models/
│   ├── Employee.cs          ← Data model (properties)
│   ├── Department.cs        ← Related entity
│   ├── EmployeeViewModel.cs ← View-specific model
│   └── ErrorViewModel.cs    ← Error handling model
│
├── Views/
│   ├── Employee/
│   │   ├── Index.cshtml     ← List view
│   │   ├── Details.cshtml   ← Single item view
│   │   ├── Create.cshtml    ← Create form
│   │   ├── Edit.cshtml      ← Edit form
│   │   └── Delete.cshtml    ← Delete confirmation
│   ├── Shared/
│   │   ├── _Layout.cshtml   ← Master page
│   │   └── Error.cshtml     ← Error page
│   └── _ViewImports.cshtml  ← Common imports
│
└── Controllers/
    ├── EmployeeController.cs ← Handles /Employee/* requests
    └── HomeController.cs     ← Handles /Home/* requests
```

---

## Model in Detail

### What is a Model?

The Model represents:
- **Data structure** (entity properties)
- **Business logic** (validation, calculations)
- **Data access** (through repositories/services)

### Model Example with Line-by-Line Explanation

```csharp
// File: Models/Employee.cs

using System.ComponentModel.DataAnnotations;           // Line 1: Import validation attributes
using System.ComponentModel.DataAnnotations.Schema;    // Line 2: Import database schema attributes

namespace MVCEmpDept.Models                            // Line 3: Define namespace
{
    public class Employee                               // Line 4: Define Employee class (entity)
    {
        // Line 5: Primary Key - uniquely identifies each employee
        // [Key] attribute marks this as the primary key in database
        [Key]
        public int Id { get; set; }

        // Line 6: Required field - cannot be null
        // [Required] ensures this field must have a value
        // ErrorMessage is shown when validation fails
        [Required(ErrorMessage = "Name is required")]
        // Line 7: String length constraint
        // MinimumLength: at least 2 characters
        // MaximumLength: at most 50 characters
        [StringLength(50, MinimumLength = 2, ErrorMessage = "Name must be 2-50 characters")]
        // Line 8: Display name for UI labels
        [Display(Name = "Employee Name")]
        public string Name { get; set; }

        // Line 9: Email validation
        // [EmailAddress] validates email format automatically
        [Required(ErrorMessage = "Email is required")]
        [EmailAddress(ErrorMessage = "Invalid email format")]
        public string Email { get; set; }

        // Line 10: Foreign key to Department table
        // This creates a relationship between Employee and Department
        [Required(ErrorMessage = "Department is required")]
        [Display(Name = "Department")]
        public int DepartmentId { get; set; }

        // Line 11: Navigation property
        // [ForeignKey] links this property to DepartmentId
        // This allows accessing the full Department object
        [ForeignKey("DepartmentId")]
        public Department? Department { get; set; }

        // Line 12: Salary with range validation
        // [Range] ensures value is between 10000 and 1000000
        [Range(10000, 1000000, ErrorMessage = "Salary must be between 10,000 and 10,00,000")]
        [Column(TypeName = "decimal(18,2)")]  // Database column type
        public decimal Salary { get; set; }
    }
}
```

### Model Validation Attributes Summary

| Attribute | Purpose | Example |
|-----------|---------|---------|
| `[Key]` | Primary key | `public int Id { get; set; }` |
| `[Required]` | Field must have value | `[Required(ErrorMessage = "...")]` |
| `[StringLength]` | Max/min string length | `[StringLength(50, MinimumLength = 2)]` |
| `[Range]` | Numeric range | `[Range(1, 100)]` |
| `[EmailAddress]` | Email format validation | `[EmailAddress]` |
| `[Phone]` | Phone number format | `[Phone]` |
| `[Display]` | UI label text | `[Display(Name = "Employee Name")]` |
| `[ForeignKey]` | Relationship definition | `[ForeignKey("DepartmentId")]` |

---

## View in Detail

### What is a View?

The View is responsible for:
- **Displaying data** to the user
- **Rendering HTML** content
- **Handling user input** (forms)
- **Presenting information** in a user-friendly format

### Razor View Syntax

```html
@* File: Views/Employee/Index.cshtml *@

@* Line 1: Model declaration - specifies the data type this view expects *@
@model IEnumerable<MVCEmpDept.Models.Employee>

@* Line 2: Set the page title in ViewData dictionary *@
@{
    ViewData["Title"] = "Employee List";
}

@* Line 3: Main heading using the ViewData title *@
<h1>@ViewData["Title"]</h1>

@* Line 4: Link to Create action - asp-action is a Tag Helper *@
<p>
    <a asp-action="Create" class="btn btn-primary">Create New Employee</a>
</p>

@* Line 5: Table structure for displaying employees *@
<table class="table table-striped table-bordered">
    <thead class="table-dark">
        <tr>
            @* Line 6: DisplayNameFor gets the [Display] attribute value *@
            <th>@Html.DisplayNameFor(model => model.Name)</th>
            <th>@Html.DisplayNameFor(model => model.Email)</th>
            <th>@Html.DisplayNameFor(model => model.Department)</th>
            <th>@Html.DisplayNameFor(model => model.Salary)</th>
            <th>Actions</th>
        </tr>
    </thead>
    <tbody>
        @* Line 7: foreach loop to iterate through all employees *@
        @foreach (var item in Model)
        {
            <tr>
                @* Line 8: DisplayFor renders the property value *@
                <td>@Html.DisplayFor(modelItem => item.Name)</td>
                <td>@Html.DisplayFor(modelItem => item.Email)</td>
                @* Line 9: Accessing navigation property *@
                <td>@Html.DisplayFor(modelItem => item.Department.Name)</td>
                @* Line 10: Currency formatting *@
                <td>@item.Salary.ToString("C")</td>
                <td>
                    @* Line 11: Action links with route values *@
                    @* asp-route-id passes the employee id to the action *@
                    <a asp-action="Edit" asp-route-id="@item.Id" 
                       class="btn btn-sm btn-warning">Edit</a>
                    <a asp-action="Details" asp-route-id="@item.Id" 
                       class="btn btn-sm btn-info">Details</a>
                    <a asp-action="Delete" asp-route-id="@item.Id" 
                       class="btn btn-sm btn-danger">Delete</a>
                </td>
            </tr>
        }
    </tbody>
</table>
```

### Razor Syntax Reference

| Syntax | Purpose | Example |
|--------|---------|---------|
| `@` | Output C# expression | `@Model.Name` |
| `@{ }` | C# code block | `@{ var x = 5; }` |
| `@if` | Conditional rendering | `@if (Model != null) { }` |
| `@foreach` | Loop through collection | `@foreach (var item in Model) { }` |
| `@model` | Declare model type | `@model Employee` |
| `@Html.DisplayFor()` | Display property value | `@Html.DisplayFor(m => m.Name)` |
| `@Html.DisplayNameFor()` | Display property label | `@Html.DisplayNameFor(m => m.Name)` |
| `@*  *@` | Razor comment | `@* This is a comment *@` |

---

## Controller in Detail

### What is a Controller?

The Controller:
- **Receives HTTP requests** from users
- **Processes input** (validates, transforms)
- **Calls Model/Services** to get/update data
- **Selects View** to render
- **Returns HTTP response**

### Complete Controller Example with Line-by-Line Explanation

```csharp
// File: Controllers/EmployeeController.cs

// Line 1-6: Import necessary namespaces
using Microsoft.AspNetCore.Mvc;              // MVC base classes
using Microsoft.AspNetCore.Mvc.Rendering;    // For SelectList
using MVCEmpDept.Models;                     // Our model classes
using MVCEmpDept.Service;                    // Service interfaces
using System.Diagnostics;                    // For Activity

namespace MVCEmpDept.Controllers
{
    // Line 7: Controller class inherits from Controller base class
    // Controller base class provides View(), RedirectToAction(), etc.
    public class EmployeeController : Controller
    {
        // Line 8: Private field to hold the injected service
        // readonly ensures it can only be set in constructor
        private IEmployeeService _employeeRepository;
        
        // Line 9: Logger for debugging and monitoring
        private readonly ILogger<EmployeeController> _logger;

        // Line 10-14: Constructor with Dependency Injection
        // ASP.NET Core automatically injects these dependencies
        public EmployeeController(
            IEmployeeService employeeRepository,      // Service for employee operations
            ILogger<EmployeeController> _logger)      // Logger instance
        {
            _employeeRepository = employeeRepository; // Store injected service
            this._logger = _logger;                   // Store injected logger
        }

        // ==================== READ OPERATIONS ====================

        // Line 15: GET: /Employee/Index
        // ActionResult is the return type for controller actions
        public ActionResult Index()
        {
            // Line 16: Log information for debugging
            _logger.LogInformation("Getting emp details");

            // Line 17: Call service to get all employees
            var model = _employeeRepository.GetAllEmployee();
            
            // Line 18: Return View with model data
            // View() looks for Views/Employee/Index.cshtml
            return View(model);
        }

        // Line 19: GET: /Employee/Details/5
        // 'id' parameter comes from URL route
        public ActionResult Details(int id)
        {
            // Line 20: Get single employee by ID
            var model = _employeeRepository.GetEmployee(id);
            
            // Line 21-27: Handle case when employee not found
            if (model == null)
            {
                // Set HTTP status code to 404
                Response.StatusCode = 404;
                
                // Return error view with error details
                return View("Error", new ErrorViewModel
                {
                    RequestId = Activity.Current?.Id ?? HttpContext.TraceIdentifier
                });
            }

            // Line 28: Return Details view with employee data
            return View(model);
        }

        // ==================== CREATE OPERATIONS ====================

        // Line 29: GET: /Employee/Create
        // This shows the empty create form
        public ActionResult Create()
        {
            // Line 30: Get all departments for dropdown
            IEnumerable<Department> DepartmentName = 
                _employeeRepository.GetAllDepartment();
            
            // Line 31: Create SelectList for dropdown
            // SelectList parameters: (items, valueField, textField)
            // ViewData["xyz"] passes data to the view
            ViewData["xyz"] = new SelectList(DepartmentName, "Id", "Name");
            
            // Line 32: Return empty Create view
            return View();
        }

        // Line 33: POST: /Employee/Create
        // [HttpPost] - only responds to POST requests
        // [ValidateAntiForgeryToken] - prevents CSRF attacks
        [HttpPost]
        [ValidateAntiForgeryToken]
        public ActionResult Create(Employee employee)
        {
            // Line 34: Get departments again for dropdown (in case of validation error)
            IEnumerable<Department> DepartmentName = 
                _employeeRepository.GetAllDepartment();

            // Line 35-40: Check if model validation passed
            if (ModelState.IsValid)
            {
                // Line 36: Add new employee to database
                var model = _employeeRepository.Add(employee);
                
                // Line 37: Redirect to Index action after successful create
                // nameof(Index) is safer than hardcoding "Index"
                return RedirectToAction(nameof(Index));
            }

            // Line 41: If validation failed, reload dropdown and show form again
            ViewData["xyz"] = new SelectList(DepartmentName, "Id", "Name");
            return View();
        }

        // ==================== UPDATE OPERATIONS ====================

        // Line 42: GET: /Employee/Edit/5
        public ActionResult Edit(int id)
        {
            // Line 43: Get employee to edit
            var model = _employeeRepository.GetEmployee(id);

            // Line 44-47: Handle not found
            if (model == null)
            {
                return View("Errorpage", id);
            }

            // Line 48: Get departments for dropdown
            IEnumerable<Department> DepartmentName = 
                _employeeRepository.GetAllDepartment();
            
            // Line 49: Create SelectList with current value selected
            // 4th parameter: model.DepartmentId is the selected value
            ViewData["DepartmentId"] = new SelectList(
                DepartmentName, "Id", "Name", model.DepartmentId);
            
            // Line 50: Return Edit view with current employee data
            return View(model);
        }

        // Line 51: POST: /Employee/Edit/5
        [HttpPost]
        [ValidateAntiForgeryToken]
        public ActionResult Edit(Employee emp)
        {
            // Line 52-58: Validate and update
            if (ModelState.IsValid)
            {
                try
                {
                    // Line 53: Update employee in database
                    var model = _employeeRepository.Update(emp);
                    return RedirectToAction(nameof(Index));
                }
                catch
                {
                    // Line 54: Return form with error
                    return View(emp);
                }
            }

            // Line 59: Reload dropdown if validation failed
            IEnumerable<Department> DepartmentName = 
                _employeeRepository.GetAllDepartment();
            ViewData["DepartmentId"] = new SelectList(
                DepartmentName, "Id", "Name", emp.DepartmentId);
            return View(emp);
        }

        // ==================== DELETE OPERATIONS ====================

        // Line 60: GET: /Employee/Delete/5
        // Shows delete confirmation page
        public ActionResult Delete(int id)
        {
            // Line 61: Get employee to delete
            var model = _employeeRepository.GetEmployee(id);
            
            // Line 62: Return NotFound if doesn't exist
            if (model == null)
                return NotFound();
            
            // Line 63: Return Delete confirmation view
            return View(model);
        }

        // Line 64: POST: /Employee/Delete/5
        [HttpPost]
        [ActionName("Delete")]           // Maps to Delete URL even with different method name
        [ValidateAntiForgeryToken]
        public ActionResult Deletedata(int id)
        {
            try
            {
                // Line 65: Delete employee from database
                var model = _employeeRepository.Delete(id);
                return RedirectToAction(nameof(Index));
            }
            catch
            {
                return View();
            }
        }

        // ==================== SPECIAL EXAMPLES ====================

        // Line 66: Dependency Injection in Action Method
        // [FromServices] injects the service directly into the method
        public IActionResult Display([FromServices] IEmployeeService es)
        {
            var model = es.GetAllEmployee();
            // Json() returns data as JSON instead of HTML view
            return Json(model);
        }

        // Line 67: View for Dependency Injection demonstration
        public ActionResult Dispview()
        {
            return View();
        }
    }
}
```

### Controller Action Flow

```
HTTP Request: GET /Employee/Details/5
                    │
                    ▼
┌─────────────────────────────────────────┐
│         ROUTING SYSTEM                  │
│  Controller = Employee                  │
│  Action = Details                       │
│  id = 5                                 │
└────────────────────┬────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────┐
│     EmployeeController Constructor      │
│  - Inject IEmployeeService              │
│  - Inject ILogger                       │
└────────────────────┬────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────┐
│      Details(int id) Method             │
│  1. Call _employeeRepository.Get(5)     │
│  2. Check if employee exists            │
│  3. Return View(employee)               │
└────────────────────┬────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────┐
│       VIEW RENDERING                    │
│  Views/Employee/Details.cshtml          │
│  - Receives Employee model              │
│  - Renders HTML                         │
└────────────────────┬────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────┐
│       HTTP RESPONSE                     │
│  - Status 200 OK                        │
│  - HTML content body                    │
└─────────────────────────────────────────┘
```

---

## Complete Code Examples

### Complete Service Interface and Implementation

```csharp
// File: Service/IEmployeeService.cs

namespace MVCEmpDept.Service
{
    // Line 1: Interface defines the contract for employee operations
    // Interfaces allow swapping implementations without changing controllers
    public interface IEmployeeService
    {
        // Line 2: Get all employees
        IEnumerable<Employee> GetAllEmployee();

        // Line 3: Get single employee by ID
        Employee? GetEmployee(int id);

        // Line 4: Add new employee
        Employee Add(Employee employee);

        // Line 5: Update existing employee
        Employee Update(Employee employee);

        // Line 6: Delete employee by ID
        Employee? Delete(int id);

        // Line 7: Get all departments for dropdowns
        IEnumerable<Department> GetAllDepartment();
    }
}
```

```csharp
// File: Service/SqlEmployeeService.cs

using Microsoft.EntityFrameworkCore;
using MVCEmpDept.Models;
using MVCEmpDept.Repository;

namespace MVCEmpDept.Service
{
    // Line 1: Concrete implementation of IEmployeeService
    // Uses Entity Framework Core for database operations
    public class SqlEmployeeService : IEmployeeService
    {
        // Line 2: DbContext for database access
        private readonly AppdbContextRepository _context;

        // Line 3: Constructor receives DbContext through DI
        public SqlEmployeeService(AppdbContextRepository context)
        {
            _context = context;
        }

        // Line 4: Get all employees with their departments
        public IEnumerable<Employee> GetAllEmployee()
        {
            // Include() performs eager loading of related Department
            return _context.Employees.Include(e => e.Department).ToList();
        }

        // Line 5: Get single employee by ID
        public Employee? GetEmployee(int id)
        {
            return _context.Employees
                .Include(e => e.Department)
                .FirstOrDefault(e => e.Id == id);
        }

        // Line 6: Add new employee
        public Employee Add(Employee employee)
        {
            _context.Employees.Add(employee);
            _context.SaveChanges();    // Commit to database
            return employee;
        }

        // Line 7: Update existing employee
        public Employee Update(Employee employee)
        {
            // Attach and mark as modified
            var emp = _context.Employees.Attach(employee);
            emp.State = EntityState.Modified;
            _context.SaveChanges();
            return employee;
        }

        // Line 8: Delete employee
        public Employee? Delete(int id)
        {
            var employee = _context.Employees.Find(id);
            if (employee != null)
            {
                _context.Employees.Remove(employee);
                _context.SaveChanges();
            }
            return employee;
        }

        // Line 9: Get all departments
        public IEnumerable<Department> GetAllDepartment()
        {
            return _context.Departments.ToList();
        }
    }
}
```

---

## Separation of Concerns

### Why Separation Matters

```
WITHOUT Separation of Concerns          WITH Separation of Concerns
┌───────────────────────────┐           ┌─────────┐ ┌─────────┐ ┌──────────┐
│     Everything Mixed      │           │  Model  │ │  View   │ │Controller│
│  - Database code          │           │         │ │         │ │          │
│  - HTML generation        │    vs     │ Data    │ │ Display │ │ Logic    │
│  - Business logic         │           │ Logic   │ │ Only    │ │ Only     │
│  - Request handling       │           │ Only    │ │         │ │          │
└───────────────────────────┘           └─────────┘ └─────────┘ └──────────┘
         HARD TO                                EASY TO
     - Test                                  - Test each part
     - Maintain                              - Modify one part
     - Scale                                 - Work in parallel
     - Debug                                 - Reuse components
```

### Benefits of MVC Pattern

| Benefit | Description |
|---------|-------------|
| **Testability** | Controllers can be unit tested without views |
| **Maintainability** | Change UI without touching business logic |
| **Parallel Development** | Frontend and backend work simultaneously |
| **Reusability** | Same model can be used by different views |
| **Flexibility** | Easy to swap implementations |

---

## Best Practices

### 1. Keep Controllers Thin

```csharp
// ❌ BAD: Fat Controller - too much logic
public ActionResult Create(Employee emp)
{
    // Validation logic here
    if (string.IsNullOrEmpty(emp.Name)) 
        ModelState.AddModelError("Name", "Required");
    // Business logic here
    emp.CreatedDate = DateTime.Now;
    emp.CreatedBy = User.Identity.Name;
    // Database logic here
    _context.Employees.Add(emp);
    _context.SaveChanges();
    return RedirectToAction("Index");
}

// ✅ GOOD: Thin Controller - delegates to service
public ActionResult Create(Employee emp)
{
    if (ModelState.IsValid)
    {
        _employeeService.Add(emp);  // All logic in service
        return RedirectToAction("Index");
    }
    return View(emp);
}
```

### 2. Use ViewModels for Complex Views

```csharp
// ✅ GOOD: Create specific ViewModel
public class EmployeeCreateViewModel
{
    public Employee Employee { get; set; }
    public SelectList Departments { get; set; }
    public SelectList Managers { get; set; }
}
```

### 3. Always Validate ModelState

```csharp
[HttpPost]
public ActionResult Create(Employee emp)
{
    // ✅ Always check ModelState before processing
    if (!ModelState.IsValid)
    {
        return View(emp);  // Return form with errors
    }
    
    // Process valid data
    _service.Add(emp);
    return RedirectToAction("Index");
}
```

### 4. Use Async for Database Operations

```csharp
// ✅ GOOD: Async for better scalability
public async Task<ActionResult> Index()
{
    var employees = await _service.GetAllAsync();
    return View(employees);
}
```

---

## Interview Questions

### Q1: What is MVC?
**Answer:** MVC (Model-View-Controller) is an architectural pattern that separates an application into three components:
- **Model**: Data and business logic
- **View**: User interface presentation
- **Controller**: Handles user requests and coordinates Model and View

### Q2: What is the role of Controller in MVC?
**Answer:** The Controller:
- Receives HTTP requests from users
- Validates and processes input data
- Calls Models/Services to perform business operations
- Selects the appropriate View to render
- Passes data to the View
- Returns HTTP responses

### Q3: What is the difference between View() and RedirectToAction()?
**Answer:**
- `View()`: Renders a view and returns HTML response (same URL)
- `RedirectToAction()`: Sends HTTP 302 redirect to another action (changes URL)

```csharp
return View();                      // Renders view, stays at /Employee/Create
return RedirectToAction("Index");   // Redirects to /Employee/Index
```

### Q4: What is [ValidateAntiForgeryToken]?
**Answer:** It's a security attribute that prevents Cross-Site Request Forgery (CSRF) attacks. It verifies that the request came from a form generated by your application by checking a token.

### Q5: What is the difference between ViewData, ViewBag, and TempData?
**Answer:**
| Feature | ViewData | ViewBag | TempData |
|---------|----------|---------|----------|
| Type | Dictionary | Dynamic | Dictionary |
| Lifetime | Current request | Current request | Until read (survives redirect) |
| Type Casting | Required | Not required | Required |

### Q6: How does model binding work?
**Answer:** Model binding automatically maps HTTP request data (form fields, query strings, route data) to action method parameters. ASP.NET Core matches parameter names with request data keys.

### Q7: What is the difference between `Controller` and `ControllerBase`?
**Answer:**
- `Controller`: Full MVC controller with View support
- `ControllerBase`: Lightweight base for Web API (no View methods)

---

## Quick Reference

### Controller Action Return Types
| Return Type | Description | Example |
|-------------|-------------|---------|
| `View()` | Render Razor view | `return View(model);` |
| `PartialView()` | Render partial view | `return PartialView("_List", model);` |
| `Json()` | Return JSON data | `return Json(data);` |
| `RedirectToAction()` | Redirect to action | `return RedirectToAction("Index");` |
| `Redirect()` | Redirect to URL | `return Redirect("/home");` |
| `NotFound()` | Return 404 | `return NotFound();` |
| `BadRequest()` | Return 400 | `return BadRequest("Error");` |
| `Ok()` | Return 200 with data | `return Ok(data);` |

### Common Attributes
| Attribute | Purpose |
|-----------|---------|
| `[HttpGet]` | Handle GET requests |
| `[HttpPost]` | Handle POST requests |
| `[HttpPut]` | Handle PUT requests |
| `[HttpDelete]` | Handle DELETE requests |
| `[Route("path")]` | Define custom route |
| `[ActionName("name")]` | Override action name |
| `[ValidateAntiForgeryToken]` | CSRF protection |
| `[FromServices]` | Method-level DI |
| `[FromBody]` | Bind from request body |
| `[FromQuery]` | Bind from query string |
