# Routing in ASP.NET Core MVC

## Table of Contents
1. [Introduction](#introduction)
2. [Types of Routing](#types-of-routing)
3. [Conventional Routing](#conventional-routing)
4. [Attribute Routing](#attribute-routing)
5. [Route Parameters](#route-parameters)
6. [Route Constraints](#route-constraints)
7. [Route Order and Priority](#route-order-and-priority)
8. [Token Replacement](#token-replacement)
9. [Area Routing](#area-routing)
10. [Code Examples](#code-examples)
11. [Best Practices](#best-practices)
12. [Interview Questions](#interview-questions)

---

## Introduction

### Real-World Analogy
Routing is like a **GPS navigation system**:

```
Destination (URL): /Employee/Details/5
                           │
                           ▼
┌──────────────────────────────────────────────────────────────┐
│                    GPS (ROUTING SYSTEM)                       │
│                                                              │
│   Parsing URL...                                             │
│   ┌────────────────┬──────────────┬─────────────┐           │
│   │  Controller    │    Action    │     ID      │           │
│   │   Employee     │   Details    │      5      │           │
│   └────────────────┴──────────────┴─────────────┘           │
│                                                              │
│   Route matched! Directing to EmployeeController.Details(5) │
└──────────────────────────────────────────────────────────────┘
```

---

## Types of Routing

### Routing Methods Comparison

| Type | Definition Location | Best For |
|------|-------------------|----------|
| **Conventional** | Program.cs | Standard MVC apps |
| **Attribute** | Controller/Actions | Web APIs, custom URLs |

```
┌─────────────────────────────────────────────────────────────────┐
│                    ROUTING IN ASP.NET CORE                       │
├────────────────────────────┬────────────────────────────────────┤
│     CONVENTIONAL ROUTING   │       ATTRIBUTE ROUTING            │
├────────────────────────────┼────────────────────────────────────┤
│                            │                                    │
│  Defined in Program.cs     │  Defined on Controllers/Actions   │
│                            │                                    │
│  app.MapControllerRoute(   │  [Route("api/[controller]")]      │
│    name: "default",        │  public class ProductController   │
│    pattern: "{controller}  │  {                                │
│             /{action}      │      [HttpGet("{id}")]            │
│             /{id?}"        │      public IActionResult Get()   │
│  );                        │  }                                │
│                            │                                    │
│  URL: /Products/Index      │  URL: /api/products/5             │
│                            │                                    │
└────────────────────────────┴────────────────────────────────────┘
```

---

## Conventional Routing

### Basic Setup

```csharp
// File: Program.cs

var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllersWithViews();

var app = builder.Build();

app.UseRouting();

// Line 1: Define default conventional route
app.MapControllerRoute(
    // Line 2: Route name (for URL generation)
    name: "default",
    
    // Line 3: URL pattern with placeholders
    // {controller} - matches controller name
    // {action} - matches action method name
    // {id?} - optional parameter (? makes it optional)
    pattern: "{controller=Home}/{action=Index}/{id?}"
);

// Breakdown:
// {controller=Home} - Default to HomeController if not specified
// {action=Index}    - Default to Index() action if not specified
// {id?}             - Optional route parameter

app.Run();
```

### How Pattern Matching Works

```
URL Pattern: {controller=Home}/{action=Index}/{id?}

Example URL Matches:
──────────────────────────────────────────────────────────────────
URL                    Controller    Action      id
──────────────────────────────────────────────────────────────────
/                      Home          Index       null
/Employee              Employee      Index       null
/Employee/Index        Employee      Index       null
/Employee/Details      Employee      Details     null
/Employee/Details/5    Employee      Details     5
/Employee/Edit/10      Employee      Edit        10
/Product/Search        Product       Search      null
──────────────────────────────────────────────────────────────────
```

### Multiple Conventional Routes

```csharp
// File: Program.cs

// Line 1: More specific routes first
app.MapControllerRoute(
    name: "blog",
    pattern: "blog/{year}/{month}/{slug}",
    defaults: new { controller = "Blog", action = "Post" }
);

// Line 2: Admin area route
app.MapControllerRoute(
    name: "admin",
    pattern: "admin/{controller=Dashboard}/{action=Index}/{id?}"
);

// Line 3: Default route (most general - LAST)
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}"
);

// URL /blog/2024/01/hello-world matches:
// Controller: Blog, Action: Post
// Parameters: year=2024, month=01, slug=hello-world
```

---

## Attribute Routing

### Basic Attribute Routing

```csharp
// File: Controllers/ProductController.cs

// Line 1: Route attribute on controller sets base path
[Route("[controller]")]  // URL starts with /Product
public class ProductController : Controller
{
    // Line 2: GET /Product or /Product/Index
    [Route("")]          // Matches /Product
    [Route("Index")]     // Also matches /Product/Index
    public IActionResult Index()
    {
        return View();
    }
    
    // Line 3: GET /Product/Details/5
    [Route("Details/{id}")]
    public IActionResult Details(int id)
    {
        return View();
    }
    
    // Line 4: Multiple routes for same action
    [Route("Edit/{id}")]
    [Route("Modify/{id}")]  // Alternative URL
    public IActionResult Edit(int id)
    {
        return View();
    }
}
```

### HTTP Method Attributes

```csharp
// File: Controllers/EmployeeController.cs

[Route("api/[controller]")]
public class EmployeeController : Controller
{
    // Line 1: GET /api/employee
    [HttpGet]
    public IActionResult GetAll()
    {
        return Ok(_employees);
    }
    
    // Line 2: GET /api/employee/5
    [HttpGet("{id}")]
    public IActionResult GetById(int id)
    {
        return Ok(_employees.Find(e => e.Id == id));
    }
    
    // Line 3: POST /api/employee
    [HttpPost]
    public IActionResult Create([FromBody] Employee emp)
    {
        return CreatedAtAction(nameof(GetById), new { id = emp.Id }, emp);
    }
    
    // Line 4: PUT /api/employee/5
    [HttpPut("{id}")]
    public IActionResult Update(int id, [FromBody] Employee emp)
    {
        return Ok(emp);
    }
    
    // Line 5: DELETE /api/employee/5
    [HttpDelete("{id}")]
    public IActionResult Delete(int id)
    {
        return NoContent();
    }
}
```

---

## Route Parameters

### Parameter Types

```csharp
// File: Controllers/OrderController.cs

[Route("[controller]")]
public class OrderController : Controller
{
    // ═══════════════════════════════════════════════════════════
    // REQUIRED PARAMETER
    // ═══════════════════════════════════════════════════════════
    
    // Line 1: Required parameter - must be provided
    [Route("Details/{id}")]  // /Order/Details/5 ✓
                              // /Order/Details   ✗ (404)
    public IActionResult Details(int id)
    {
        return View();
    }
    
    // ═══════════════════════════════════════════════════════════
    // OPTIONAL PARAMETER
    // ═══════════════════════════════════════════════════════════
    
    // Line 2: Optional parameter with ?
    [Route("Search/{category?}")]  // /Order/Search       ✓
                                    // /Order/Search/Books ✓
    public IActionResult Search(string? category)
    {
        return View();
    }
    
    // ═══════════════════════════════════════════════════════════
    // DEFAULT VALUE
    // ═══════════════════════════════════════════════════════════
    
    // Line 3: Parameter with default value
    [Route("List/{page=1}")]  // /Order/List   → page = 1
                               // /Order/List/5 → page = 5
    public IActionResult List(int page = 1)
    {
        return View();
    }
    
    // ═══════════════════════════════════════════════════════════
    // CATCH-ALL PARAMETER
    // ═══════════════════════════════════════════════════════════
    
    // Line 4: Catch-all with *
    [Route("Files/{*filePath}")]  // /Order/Files/docs/reports/2024/q1.pdf
                                   // filePath = "docs/reports/2024/q1.pdf"
    public IActionResult Files(string filePath)
    {
        return View();
    }
    
    // ═══════════════════════════════════════════════════════════
    // MULTIPLE PARAMETERS
    // ═══════════════════════════════════════════════════════════
    
    // Line 5: Multiple route parameters
    [Route("{year}/{month}/{day}")]  // /Order/2024/01/15
    public IActionResult ByDate(int year, int month, int day)
    {
        return View();
    }
}
```

---

## Route Constraints

### Built-in Constraints

```csharp
// File: Controllers/ProductController.cs

[Route("[controller]")]
public class ProductController : Controller
{
    // ═══════════════════════════════════════════════════════════
    // TYPE CONSTRAINTS
    // ═══════════════════════════════════════════════════════════
    
    // Line 1: Integer only
    [Route("Details/{id:int}")]  // /Product/Details/5   ✓
                                  // /Product/Details/abc ✗ (no match)
    public IActionResult Details(int id) => View();
    
    // Line 2: GUID only
    [Route("Order/{orderId:guid}")]
    public IActionResult Order(Guid orderId) => View();
    
    // Line 3: Boolean only
    [Route("Status/{active:bool}")]
    public IActionResult Status(bool active) => View();
    
    // Line 4: DateTime only
    [Route("Report/{date:datetime}")]
    public IActionResult Report(DateTime date) => View();
    
    // ═══════════════════════════════════════════════════════════
    // LENGTH CONSTRAINTS
    // ═══════════════════════════════════════════════════════════
    
    // Line 5: Minimum length
    [Route("Search/{term:minlength(3)}")]  // At least 3 characters
    public IActionResult Search(string term) => View();
    
    // Line 6: Maximum length
    [Route("Code/{code:maxlength(10)}")]  // At most 10 characters
    public IActionResult Code(string code) => View();
    
    // Line 7: Exact length
    [Route("SKU/{sku:length(8)}")]  // Exactly 8 characters
    public IActionResult SKU(string sku) => View();
    
    // Line 8: Range of lengths
    [Route("Username/{name:length(3,20)}")]  // 3-20 characters
    public IActionResult Username(string name) => View();
    
    // ═══════════════════════════════════════════════════════════
    // RANGE CONSTRAINTS
    // ═══════════════════════════════════════════════════════════
    
    // Line 9: Minimum value
    [Route("Page/{page:min(1)}")]  // Must be >= 1
    public IActionResult Page(int page) => View();
    
    // Line 10: Maximum value
    [Route("Rating/{rating:max(5)}")]  // Must be <= 5
    public IActionResult Rating(int rating) => View();
    
    // Line 11: Range
    [Route("Month/{month:range(1,12)}")]  // 1-12 inclusive
    public IActionResult Month(int month) => View();
    
    // ═══════════════════════════════════════════════════════════
    // PATTERN CONSTRAINTS
    // ═══════════════════════════════════════════════════════════
    
    // Line 12: Alpha only (letters)
    [Route("Category/{name:alpha}")]  // Only letters
    public IActionResult Category(string name) => View();
    
    // Line 13: Regex pattern
    [Route("Phone/{number:regex(^\\d{{3}}-\\d{{4}}$)}")]  // Format: 000-0000
    public IActionResult Phone(string number) => View();
    
    // ═══════════════════════════════════════════════════════════
    // MULTIPLE CONSTRAINTS
    // ═══════════════════════════════════════════════════════════
    
    // Line 14: Combine constraints with colon
    [Route("Employee/{id:int:min(1):max(1000)}")]
    public IActionResult Employee(int id) => View();
}
```

### Constraint Reference Table

| Constraint | Description | Example |
|------------|-------------|---------|
| `int` | Matches integer | `{id:int}` |
| `bool` | Matches boolean | `{active:bool}` |
| `datetime` | Matches DateTime | `{date:datetime}` |
| `decimal` | Matches decimal | `{price:decimal}` |
| `double` | Matches double | `{rate:double}` |
| `float` | Matches float | `{value:float}` |
| `guid` | Matches GUID | `{id:guid}` |
| `long` | Matches long | `{id:long}` |
| `minlength(n)` | Min string length | `{name:minlength(3)}` |
| `maxlength(n)` | Max string length | `{name:maxlength(50)}` |
| `length(n)` | Exact length | `{code:length(10)}` |
| `length(m,n)` | Length range | `{code:length(5,10)}` |
| `min(n)` | Min numeric value | `{page:min(1)}` |
| `max(n)` | Max numeric value | `{rating:max(5)}` |
| `range(m,n)` | Numeric range | `{month:range(1,12)}` |
| `alpha` | Letters only | `{name:alpha}` |
| `regex(expr)` | Regex pattern | `{code:regex(^[A-Z]{{3}}$)}` |
| `required` | Must have value | `{name:required}` |

---

## Route Order and Priority

### How Routes are Matched

```
┌─────────────────────────────────────────────────────────────────┐
│                   ROUTE MATCHING PRIORITY                        │
└─────────────────────────────────────────────────────────────────┘

Routes are matched in this order:

1. MORE SPECIFIC routes match first
2. Routes with MORE CONSTRAINTS win
3. Routes with MORE LITERAL SEGMENTS win
4. Routes with FEWER OPTIONAL PARAMETERS win

Example:
─────────────────────────────────────────────────────────────────
URL: /Product/Details/5

Route A: [Route("Product/Details/{id:int}")]     ← MATCHES (more specific)
Route B: [Route("Product/{action}/{id?}")]       ← Also matches (less specific)
Route C: [Route("{controller}/{action}/{id?}")]  ← Also matches (least specific)

Route A wins because it has literal segments "Product/Details"
```

### Order Attribute

```csharp
// Control matching order with Order property
[Route("Product/Special", Order = 1)]  // Checked first
public IActionResult Special() => View();

[Route("Product/{id}", Order = 2)]     // Checked second
public IActionResult Details(int id) => View();

// Lower Order values have higher priority
// Default Order is 0
```

---

## Token Replacement

### Controller and Action Tokens

```csharp
// [controller] and [action] are replaced at runtime

[Route("[controller]")]     // Replaced with controller name
public class ProductController : Controller
{
    // URL: /Product
    
    [Route("[action]")]     // Replaced with action name
    public IActionResult Index()
    {
        // URL: /Product/Index
        return View();
    }
    
    [Route("[action]/{id}")]
    public IActionResult Details(int id)
    {
        // URL: /Product/Details/5
        return View();
    }
}

// Combined tokens
[Route("[controller]/[action]")]
public class OrderController : Controller
{
    public IActionResult List()
    {
        // URL: /Order/List
        return View();
    }
}
```

---

## Area Routing

### Setting Up Areas

```csharp
// File: Areas/Admin/Controllers/DashboardController.cs

// Line 1: Specify the area
[Area("Admin")]
public class DashboardController : Controller
{
    // URL: /Admin/Dashboard/Index
    public IActionResult Index()
    {
        return View();
    }
}
```

### Area Route Configuration

```csharp
// File: Program.cs

app.UseRouting();

// Line 1: Area route (must come before default)
app.MapControllerRoute(
    name: "areas",
    pattern: "{area:exists}/{controller=Home}/{action=Index}/{id?}"
);

// Line 2: Default route
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}"
);
```

### Area Folder Structure

```
Areas/
├── Admin/
│   ├── Controllers/
│   │   └── DashboardController.cs
│   ├── Views/
│   │   ├── Dashboard/
│   │   │   └── Index.cshtml
│   │   ├── _ViewImports.cshtml
│   │   └── _ViewStart.cshtml
│   └── Models/
└── Customer/
    ├── Controllers/
    └── Views/
```

---

## Code Examples

### Complete Routing Example

```csharp
// File: Controllers/EmployeeController.cs

using Microsoft.AspNetCore.Mvc;
using MVCEmpDept.Models;
using MVCEmpDept.Service;

namespace MVCEmpDept.Controllers
{
    // Line 1: No Route attribute - uses conventional routing
    public class EmployeeController : Controller
    {
        private readonly IEmployeeService _employeeRepository;
        
        public EmployeeController(IEmployeeService employeeRepository)
        {
            _employeeRepository = employeeRepository;
        }
        
        // ═══════════════════════════════════════════════════════
        // URL: /Employee or /Employee/Index (conventional routing)
        // ═══════════════════════════════════════════════════════
        public ActionResult Index()
        {
            var model = _employeeRepository.GetAllEmployee();
            return View(model);
        }
        
        // ═══════════════════════════════════════════════════════
        // URL: /Employee/Details/5
        // The 'id' comes from route parameter {id}
        // ═══════════════════════════════════════════════════════
        public ActionResult Details(int id)
        {
            var model = _employeeRepository.GetEmployee(id);
            if (model == null)
            {
                return NotFound();
            }
            return View(model);
        }
        
        // ═══════════════════════════════════════════════════════
        // URL: /Employee/Create (GET - show form)
        // ═══════════════════════════════════════════════════════
        public ActionResult Create()
        {
            return View();
        }
        
        // ═══════════════════════════════════════════════════════
        // URL: /Employee/Create (POST - handle form submission)
        // ═══════════════════════════════════════════════════════
        [HttpPost]
        [ValidateAntiForgeryToken]
        public ActionResult Create(Employee employee)
        {
            if (ModelState.IsValid)
            {
                _employeeRepository.Add(employee);
                return RedirectToAction(nameof(Index));
            }
            return View(employee);
        }
        
        // ═══════════════════════════════════════════════════════
        // URL: /Employee/Edit/5
        // ═══════════════════════════════════════════════════════
        public ActionResult Edit(int id)
        {
            var model = _employeeRepository.GetEmployee(id);
            if (model == null)
            {
                return NotFound();
            }
            return View(model);
        }
        
        // ═══════════════════════════════════════════════════════
        // URL: /Employee/Edit/5 (POST)
        // ═══════════════════════════════════════════════════════
        [HttpPost]
        [ValidateAntiForgeryToken]
        public ActionResult Edit(Employee emp)
        {
            if (ModelState.IsValid)
            {
                _employeeRepository.Update(emp);
                return RedirectToAction(nameof(Index));
            }
            return View(emp);
        }
        
        // ═══════════════════════════════════════════════════════
        // URL: /Employee/Delete/5
        // ═══════════════════════════════════════════════════════
        public ActionResult Delete(int id)
        {
            var model = _employeeRepository.GetEmployee(id);
            if (model == null)
                return NotFound();
            return View(model);
        }
        
        // ═══════════════════════════════════════════════════════
        // POST action with different method name but same URL
        // [ActionName] makes it accessible at /Employee/Delete
        // ═══════════════════════════════════════════════════════
        [HttpPost]
        [ActionName("Delete")]
        [ValidateAntiForgeryToken]
        public ActionResult DeleteConfirmed(int id)
        {
            _employeeRepository.Delete(id);
            return RedirectToAction(nameof(Index));
        }
    }
}
```

### Generating URLs

```csharp
// In Controller
public IActionResult Example()
{
    // Generate URL to an action
    var url = Url.Action("Details", "Employee", new { id = 5 });
    // Result: /Employee/Details/5
    
    // Redirect to action
    return RedirectToAction("Index", "Home");
}
```

```html
<!-- In View -->
<!-- Using Tag Helpers -->
<a asp-controller="Employee" 
   asp-action="Details" 
   asp-route-id="@item.Id">Details</a>

<!-- Using Html Helpers -->
@Html.ActionLink("Details", "Details", "Employee", new { id = item.Id })

<!-- Generated HTML -->
<a href="/Employee/Details/5">Details</a>
```

---

## Best Practices

### 1. Use Attribute Routing for APIs

```csharp
// ✅ GOOD: Clear API routes with attributes
[Route("api/v1/[controller]")]
[ApiController]
public class ProductsController : ControllerBase
{
    [HttpGet]
    public IActionResult GetAll() => Ok();
    
    [HttpGet("{id}")]
    public IActionResult GetById(int id) => Ok();
}
```

### 2. Use Constraints for Validation

```csharp
// ✅ GOOD: Route-level validation
[HttpGet("{id:int:min(1)}")]  // Only positive integers
public IActionResult Get(int id) => Ok();

// ❌ BAD: Validation only in action
[HttpGet("{id}")]
public IActionResult Get(int id)
{
    if (id <= 0) return BadRequest();  // Too late!
}
```

### 3. Order Routes Properly

```csharp
// ✅ CORRECT: More specific routes FIRST
app.MapControllerRoute("special", "products/featured", ...);
app.MapControllerRoute("default", "{controller}/{action}/{id?}");

// ❌ WRONG: Default catches everything
app.MapControllerRoute("default", "{controller}/{action}/{id?}");
app.MapControllerRoute("special", "products/featured", ...);  // Never reached!
```

---

## Interview Questions

### Q1: What are the two types of routing in ASP.NET Core?
**Answer:** 
1. **Conventional Routing**: Defined in Program.cs using `MapControllerRoute`, uses pattern matching with placeholders like `{controller}/{action}/{id}`
2. **Attribute Routing**: Defined on controllers/actions using `[Route]`, `[HttpGet]`, etc. attributes

### Q2: What is the difference between {id} and {id?} in route patterns?
**Answer:** 
- `{id}` - Required parameter, must be provided
- `{id?}` - Optional parameter, can be omitted

### Q3: What are route constraints? Give examples.
**Answer:** Route constraints restrict what values a route parameter can match:
- `{id:int}` - Must be an integer
- `{name:minlength(3)}` - Minimum 3 characters
- `{page:range(1,100)}` - Between 1 and 100
- `{code:regex(^[A-Z]{3}$)}` - Three uppercase letters

### Q4: What is [ActionName] attribute?
**Answer:** `[ActionName]` allows you to specify a different name for an action method than its method name. Common use: having multiple methods respond to the same action name with different HTTP methods.

### Q5: Why should more specific routes be defined before general routes?
**Answer:** Routes are matched in order. If a general route (like `{controller}/{action}`) is defined first, it will match URLs that should go to more specific routes. More specific routes must be checked first to ensure correct matching.

---

## Quick Reference

### Route Pattern Syntax

```
Literal:     /products/list      (exact match)
Parameter:   /products/{id}      (captures segment)
Optional:    /products/{id?}     (optional parameter)
Default:     /products/{id=1}    (default value)
Catch-all:   /files/{*path}      (captures rest of URL)
Constraint:  /products/{id:int}  (type constraint)
```

### Common Attributes

| Attribute | Purpose |
|-----------|---------|
| `[Route("path")]` | Define route template |
| `[HttpGet]` | Handle GET requests |
| `[HttpPost]` | Handle POST requests |
| `[HttpPut]` | Handle PUT requests |
| `[HttpDelete]` | Handle DELETE requests |
| `[ActionName("name")]` | Override action name |
| `[Area("name")]` | Assign to area |

### URL Generation

```csharp
// In Controller
Url.Action("Action", "Controller", new { id = 5 })

// In View (Tag Helper)
<a asp-controller="X" asp-action="Y" asp-route-id="5">Link</a>
```
