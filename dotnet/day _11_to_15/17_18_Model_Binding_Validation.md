# Model Binding and Validation in ASP.NET Core

## Table of Contents
1. [Introduction](#introduction)
2. [What is Model Binding?](#what-is-model-binding)
3. [Binding Sources](#binding-sources)
4. [Complex Type Binding](#complex-type-binding)
5. [Model Validation](#model-validation)
6. [Data Annotations](#data-annotations)
7. [Custom Validation](#custom-validation)
8. [Client-Side Validation](#client-side-validation)
9. [ModelState](#modelstate)
10. [Code Examples](#code-examples)
11. [Best Practices](#best-practices)
12. [Interview Questions](#interview-questions)

---

## Introduction

### Real-World Analogy
Model Binding is like a **smart mail sorting system**:

```
Incoming Mail (HTTP Request)
        │
        │ Contains: Form data, URLs, Headers, Body
        ▼
┌──────────────────────────────────────────────────────────────┐
│               MAIL SORTING SYSTEM                            │
│                  (Model Binder)                              │
│                                                              │
│   📧 "Name=John"      → Employee.Name = "John"              │
│   📧 "Email=j@x.com"  → Employee.Email = "j@x.com"          │
│   📧 "Age=30"         → Employee.Age = 30                   │
│   📧 "DeptId=5"       → Employee.DepartmentId = 5           │
│                                                              │
│   Automatically organizes into correct employee object!     │
└──────────────────────────────────────────────────────────────┘
        │
        ▼
   Employee Object (Ready to use in Controller)
```

---

## What is Model Binding?

Model Binding automatically maps HTTP request data to action method parameters and model objects.

### How It Works

```
HTTP Request:
POST /Employee/Create
Content-Type: application/x-www-form-urlencoded

name=John&email=john@company.com&departmentId=5
        │
        │  Model Binding
        ▼
public ActionResult Create(Employee employee)
{
    // employee.Name = "John"
    // employee.Email = "john@company.com"  
    // employee.DepartmentId = 5
    
    // Automatically populated!
}
```

### Binding Flow Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                    MODEL BINDING PROCESS                         │
└─────────────────────────────────────────────────────────────────┘

    HTTP Request
         │
         ▼
┌─────────────────────────────────────────────────────────────────┐
│  1. IDENTIFY BINDING SOURCE                                      │
│     - Route data (/Employee/Edit/5 → id=5)                      │
│     - Query string (?page=2 → page=2)                            │
│     - Form data (POST body)                                      │
│     - Request body (JSON)                                        │
│     - Headers                                                    │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│  2. MATCH PARAMETER NAMES                                        │
│     Request: name=John, email=j@x.com, departmentId=5           │
│     Model:   Name,      Email,        DepartmentId              │
│              ↓          ↓             ↓                          │
│              Match      Match         Match                      │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│  3. TYPE CONVERSION                                              │
│     "30" (string) → 30 (int)                                    │
│     "true" (string) → true (bool)                                │
│     "2024-01-15" (string) → DateTime                             │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│  4. VALIDATE MODEL                                               │
│     Check [Required], [StringLength], [Range], etc.             │
│     Populate ModelState with errors                              │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
              Action Method Receives Populated Model
```

---

## Binding Sources

### Default Binding Sources (Priority Order)

| Priority | Source | Example | Attribute |
|----------|--------|---------|-----------|
| 1 | Form values | POST form data | `[FromForm]` |
| 2 | Route data | `/Employee/Edit/{id}` | `[FromRoute]` |
| 3 | Query string | `?page=2&sort=name` | `[FromQuery]` |
| 4 | Request body | JSON body | `[FromBody]` |
| 5 | Headers | Authorization header | `[FromHeader]` |
| 6 | Services | DI container | `[FromServices]` |

### Explicit Binding Source Attributes

```csharp
// File: Controllers/ProductController.cs

[ApiController]
[Route("api/[controller]")]
public class ProductController : ControllerBase
{
    // ═══════════════════════════════════════════════════════════
    // [FromRoute] - Bind from URL route
    // ═══════════════════════════════════════════════════════════
    
    // GET /api/product/5
    [HttpGet("{id}")]
    public IActionResult Get([FromRoute] int id)
    {
        // id comes from URL path
        return Ok($"Product ID: {id}");
    }
    
    // ═══════════════════════════════════════════════════════════
    // [FromQuery] - Bind from query string
    // ═══════════════════════════════════════════════════════════
    
    // GET /api/product?category=electronics&page=2
    [HttpGet]
    public IActionResult Search(
        [FromQuery] string category,
        [FromQuery] int page = 1)
    {
        return Ok($"Category: {category}, Page: {page}");
    }
    
    // ═══════════════════════════════════════════════════════════
    // [FromBody] - Bind from request body (JSON)
    // ═══════════════════════════════════════════════════════════
    
    // POST /api/product
    // Body: {"name":"Laptop","price":999.99}
    [HttpPost]
    public IActionResult Create([FromBody] Product product)
    {
        return CreatedAtAction(nameof(Get), new { id = product.Id }, product);
    }
    
    // ═══════════════════════════════════════════════════════════
    // [FromHeader] - Bind from HTTP header
    // ═══════════════════════════════════════════════════════════
    
    [HttpGet("check")]
    public IActionResult Check([FromHeader(Name = "X-Api-Key")] string apiKey)
    {
        return Ok($"API Key: {apiKey}");
    }
    
    // ═══════════════════════════════════════════════════════════
    // [FromForm] - Bind from form data
    // ═══════════════════════════════════════════════════════════
    
    [HttpPost("upload")]
    public IActionResult Upload([FromForm] IFormFile file, [FromForm] string description)
    {
        return Ok($"File: {file.FileName}, Description: {description}");
    }
    
    // ═══════════════════════════════════════════════════════════
    // [FromServices] - Inject service from DI
    // ═══════════════════════════════════════════════════════════
    
    [HttpGet("di-example")]
    public IActionResult DIExample([FromServices] IProductService service)
    {
        // service is injected from DI container
        return Ok(service.GetAll());
    }
}
```

---

## Complex Type Binding

### Binding to Complex Objects

```csharp
// Model
public class Employee
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
    public int DepartmentId { get; set; }
    public Address Address { get; set; }  // Nested object
}

public class Address
{
    public string Street { get; set; }
    public string City { get; set; }
    public string ZipCode { get; set; }
}
```

```html
<!-- Form with nested object binding -->
<form asp-action="Create" method="post">
    <!-- Simple properties -->
    <input name="Name" value="" />
    <input name="Email" value="" />
    <input name="DepartmentId" value="" />
    
    <!-- Nested object: Use dot notation -->
    <input name="Address.Street" value="" />
    <input name="Address.City" value="" />
    <input name="Address.ZipCode" value="" />
    
    <button type="submit">Create</button>
</form>
```

### Binding Collections

```html
<!-- List binding: Use index notation -->
<form asp-action="CreateMultiple" method="post">
    <!-- First employee -->
    <input name="employees[0].Name" value="John" />
    <input name="employees[0].Email" value="john@x.com" />
    
    <!-- Second employee -->
    <input name="employees[1].Name" value="Jane" />
    <input name="employees[1].Email" value="jane@x.com" />
    
    <button type="submit">Create All</button>
</form>
```

```csharp
// Controller action
[HttpPost]
public IActionResult CreateMultiple(List<Employee> employees)
{
    // employees[0] = {Name: "John", Email: "john@x.com"}
    // employees[1] = {Name: "Jane", Email: "jane@x.com"}
    return RedirectToAction("Index");
}
```

---

## Model Validation

### Validation Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                    VALIDATION PROCESS                            │
└─────────────────────────────────────────────────────────────────┘

                    Model Binding Complete
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│  1. VALIDATION ATTRIBUTES CHECKED                                │
│     [Required] - Is value present?                              │
│     [StringLength] - Is length valid?                           │
│     [Range] - Is value in range?                                │
│     [EmailAddress] - Is format valid?                           │
└────────────────────────────┬────────────────────────────────────┘
                             │
              ┌──────────────┴──────────────┐
              │                             │
              ▼                             ▼
┌─────────────────────┐        ┌─────────────────────┐
│  ALL VALID          │        │  VALIDATION ERRORS  │
│                     │        │                     │
│  ModelState.IsValid │        │  ModelState.IsValid │
│      = true         │        │      = false        │
│                     │        │                     │
│  Continue with      │        │  Return View with   │
│  business logic     │        │  error messages     │
└─────────────────────┘        └─────────────────────┘
```

---

## Data Annotations

### Common Validation Attributes

```csharp
// File: Models/Employee.cs

using System.ComponentModel.DataAnnotations;

public class Employee
{
    // ═══════════════════════════════════════════════════════════
    // [Key] - Primary key marker
    // ═══════════════════════════════════════════════════════════
    [Key]
    public int Id { get; set; }
    
    // ═══════════════════════════════════════════════════════════
    // [Required] - Value must be provided
    // ═══════════════════════════════════════════════════════════
    [Required(ErrorMessage = "Name is required")]
    [Display(Name = "Employee Name")]
    public string Name { get; set; }
    
    // ═══════════════════════════════════════════════════════════
    // [StringLength] - Min and max string length
    // ═══════════════════════════════════════════════════════════
    [StringLength(100, MinimumLength = 2, 
        ErrorMessage = "Name must be 2-100 characters")]
    public string FullName { get; set; }
    
    // ═══════════════════════════════════════════════════════════
    // [EmailAddress] - Email format validation
    // ═══════════════════════════════════════════════════════════
    [Required(ErrorMessage = "Email is required")]
    [EmailAddress(ErrorMessage = "Invalid email format")]
    public string Email { get; set; }
    
    // ═══════════════════════════════════════════════════════════
    // [Phone] - Phone number format
    // ═══════════════════════════════════════════════════════════
    [Phone(ErrorMessage = "Invalid phone number")]
    public string PhoneNumber { get; set; }
    
    // ═══════════════════════════════════════════════════════════
    // [Range] - Numeric range validation
    // ═══════════════════════════════════════════════════════════
    [Range(18, 65, ErrorMessage = "Age must be between 18 and 65")]
    public int Age { get; set; }
    
    // ═══════════════════════════════════════════════════════════
    // [Range] for decimal/money
    // ═══════════════════════════════════════════════════════════
    [Range(typeof(decimal), "10000", "10000000", 
        ErrorMessage = "Salary must be between 10,000 and 1,00,00,000")]
    [DataType(DataType.Currency)]
    public decimal Salary { get; set; }
    
    // ═══════════════════════════════════════════════════════════
    // [Compare] - Compare with another property
    // ═══════════════════════════════════════════════════════════
    [DataType(DataType.Password)]
    public string Password { get; set; }
    
    [Compare("Password", ErrorMessage = "Passwords don't match")]
    [DataType(DataType.Password)]
    public string ConfirmPassword { get; set; }
    
    // ═══════════════════════════════════════════════════════════
    // [RegularExpression] - Custom regex pattern
    // ═══════════════════════════════════════════════════════════
    [RegularExpression(@"^[A-Z]{3}\d{4}$", 
        ErrorMessage = "Code must be 3 uppercase letters followed by 4 digits")]
    public string EmployeeCode { get; set; }
    
    // ═══════════════════════════════════════════════════════════
    // [Url] - Valid URL format
    // ═══════════════════════════════════════════════════════════
    [Url(ErrorMessage = "Invalid URL format")]
    public string Website { get; set; }
    
    // ═══════════════════════════════════════════════════════════
    // [CreditCard] - Credit card number format
    // ═══════════════════════════════════════════════════════════
    [CreditCard(ErrorMessage = "Invalid credit card number")]
    public string CardNumber { get; set; }
    
    // ═══════════════════════════════════════════════════════════
    // [Display] - UI label and order
    // ═══════════════════════════════════════════════════════════
    [Display(Name = "Department", Order = 5)]
    [Required(ErrorMessage = "Department is required")]
    public int DepartmentId { get; set; }
}
```

### Validation Attribute Reference Table

| Attribute | Purpose | Example |
|-----------|---------|---------|
| `[Required]` | Field must have value | `[Required(ErrorMessage = "...")]` |
| `[StringLength]` | String length limits | `[StringLength(50, MinimumLength = 2)]` |
| `[Range]` | Numeric range | `[Range(1, 100)]` |
| `[EmailAddress]` | Email format | `[EmailAddress]` |
| `[Phone]` | Phone format | `[Phone]` |
| `[Url]` | URL format | `[Url]` |
| `[CreditCard]` | Credit card format | `[CreditCard]` |
| `[Compare]` | Match another property | `[Compare("Password")]` |
| `[RegularExpression]` | Custom regex | `[RegularExpression(@"^[A-Z]+$")]` |
| `[DataType]` | Data type hint | `[DataType(DataType.Date)]` |
| `[Display]` | UI display name | `[Display(Name = "Full Name")]` |
| `[MaxLength]` | Max collection/string length | `[MaxLength(100)]` |
| `[MinLength]` | Min collection/string length | `[MinLength(5)]` |

---

## Custom Validation

### Creating Custom Validation Attribute

```csharp
// File: Validation/MinimumAgeAttribute.cs

using System.ComponentModel.DataAnnotations;

// Line 1: Custom validation attribute
public class MinimumAgeAttribute : ValidationAttribute
{
    private readonly int _minimumAge;
    
    // Line 2: Constructor accepts minimum age
    public MinimumAgeAttribute(int minimumAge)
    {
        _minimumAge = minimumAge;
        ErrorMessage = $"You must be at least {minimumAge} years old.";
    }
    
    // Line 3: Override IsValid for validation logic
    protected override ValidationResult? IsValid(
        object? value, 
        ValidationContext validationContext)
    {
        if (value is DateTime dateOfBirth)
        {
            // Line 4: Calculate age
            var today = DateTime.Today;
            var age = today.Year - dateOfBirth.Year;
            
            if (dateOfBirth.Date > today.AddYears(-age))
                age--;
            
            // Line 5: Check minimum age
            if (age >= _minimumAge)
            {
                return ValidationResult.Success;
            }
        }
        
        // Line 6: Return error if invalid
        return new ValidationResult(ErrorMessage);
    }
}
```

```csharp
// Usage in Model
public class Employee
{
    [Required]
    [MinimumAge(18)]  // Custom attribute
    [DataType(DataType.Date)]
    public DateTime DateOfBirth { get; set; }
}
```

### IValidatableObject Interface

```csharp
// File: Models/Order.cs

using System.ComponentModel.DataAnnotations;

// Line 1: Implement IValidatableObject for complex validation
public class Order : IValidatableObject
{
    public int Id { get; set; }
    
    [Required]
    public DateTime OrderDate { get; set; }
    
    [Required]
    public DateTime ShipDate { get; set; }
    
    [Required]
    public string ProductName { get; set; }
    
    public int Quantity { get; set; }
    public decimal UnitPrice { get; set; }
    public decimal Discount { get; set; }
    
    // Line 2: Validate method for cross-property validation
    public IEnumerable<ValidationResult> Validate(ValidationContext validationContext)
    {
        // Line 3: ShipDate must be after OrderDate
        if (ShipDate < OrderDate)
        {
            yield return new ValidationResult(
                "Ship date must be after order date",
                new[] { nameof(ShipDate) });
        }
        
        // Line 4: Discount cannot exceed total price
        var total = Quantity * UnitPrice;
        if (Discount > total)
        {
            yield return new ValidationResult(
                "Discount cannot exceed total order value",
                new[] { nameof(Discount) });
        }
        
        // Line 5: Quantity must be positive
        if (Quantity <= 0)
        {
            yield return new ValidationResult(
                "Quantity must be greater than zero",
                new[] { nameof(Quantity) });
        }
    }
}
```

---

## Client-Side Validation

### Enabling Client-Side Validation

```html
<!-- File: Views/Shared/_Layout.cshtml -->
<!DOCTYPE html>
<html>
<head>
    <title>@ViewData["Title"]</title>
</head>
<body>
    @RenderBody()
    
    <!-- jQuery (required for validation) -->
    <script src="~/lib/jquery/dist/jquery.min.js"></script>
    
    <!-- jQuery Validation -->
    <script src="~/lib/jquery-validation/dist/jquery.validate.min.js"></script>
    
    <!-- Unobtrusive Validation (ASP.NET Core specific) -->
    <script src="~/lib/jquery-validation-unobtrusive/jquery.validate.unobtrusive.min.js"></script>
    
    @await RenderSectionAsync("Scripts", required: false)
</body>
</html>
```

### Validation in Views

```html
<!-- File: Views/Employee/Create.cshtml -->
@model Employee

<form asp-action="Create" method="post">
    <!-- Validation Summary - shows all errors -->
    <div asp-validation-summary="All" class="text-danger"></div>
    
    <div class="form-group">
        <label asp-for="Name"></label>
        <!-- Input with validation attributes -->
        <input asp-for="Name" class="form-control" />
        <!-- Individual field validation message -->
        <span asp-validation-for="Name" class="text-danger"></span>
    </div>
    
    <div class="form-group">
        <label asp-for="Email"></label>
        <input asp-for="Email" class="form-control" />
        <span asp-validation-for="Email" class="text-danger"></span>
    </div>
    
    <div class="form-group">
        <label asp-for="Age"></label>
        <input asp-for="Age" class="form-control" />
        <span asp-validation-for="Age" class="text-danger"></span>
    </div>
    
    <button type="submit" class="btn btn-primary">Create</button>
</form>

@section Scripts {
    @{await Html.RenderPartialAsync("_ValidationScriptsPartial");}
}
```

### Generated HTML with Validation Attributes

```html
<!-- The tag helper generates data-val-* attributes for client-side validation -->
<input 
    class="form-control" 
    type="text" 
    id="Name" 
    name="Name"
    data-val="true"
    data-val-required="Name is required"
    data-val-length="Name must be 2-100 characters"
    data-val-length-max="100"
    data-val-length-min="2"
    value="" />
```

---

## ModelState

### Checking ModelState

```csharp
// File: Controllers/EmployeeController.cs

[HttpPost]
[ValidateAntiForgeryToken]
public ActionResult Create(Employee employee)
{
    // Line 1: Check if all validations passed
    if (ModelState.IsValid)
    {
        // All validations passed - save to database
        _repository.Add(employee);
        return RedirectToAction(nameof(Index));
    }
    
    // Line 2: Validation failed - return form with errors
    // Reload any dropdown data
    ViewData["Departments"] = new SelectList(_repository.GetDepartments(), "Id", "Name");
    
    // Return view with the model (preserves user input)
    return View(employee);
}
```

### Adding Custom Errors

```csharp
[HttpPost]
public ActionResult Create(Employee employee)
{
    // Line 1: Check for duplicate email (custom validation)
    if (_repository.EmailExists(employee.Email))
    {
        // Add error for specific property
        ModelState.AddModelError("Email", "This email is already registered");
    }
    
    // Line 2: Check for business rule
    if (employee.Salary > 1000000 && employee.Age < 25)
    {
        // Add error not tied to specific property
        ModelState.AddModelError(string.Empty, 
            "Employees under 25 cannot have salary over 10 lakhs");
    }
    
    if (ModelState.IsValid)
    {
        _repository.Add(employee);
        return RedirectToAction(nameof(Index));
    }
    
    return View(employee);
}
```

### ModelState Properties and Methods

| Member | Description |
|--------|-------------|
| `IsValid` | True if no errors |
| `ErrorCount` | Number of errors |
| `Values` | All value entries |
| `Keys` | All property keys |
| `AddModelError(key, message)` | Add custom error |
| `Clear()` | Remove all entries |
| `Remove(key)` | Remove specific entry |

---

## Code Examples

### Complete Controller with Validation

```csharp
// File: Controllers/EmployeeController.cs

using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.Rendering;
using MVCEmpDept.Models;
using MVCEmpDept.Service;

public class EmployeeController : Controller
{
    private readonly IEmployeeService _employeeRepository;
    private readonly ILogger<EmployeeController> _logger;

    public EmployeeController(
        IEmployeeService employeeRepository, 
        ILogger<EmployeeController> logger)
    {
        _employeeRepository = employeeRepository;
        _logger = logger;
    }

    // ═══════════════════════════════════════════════════════════
    // GET: /Employee/Create
    // ═══════════════════════════════════════════════════════════
    public ActionResult Create()
    {
        // Line 1: Populate dropdown for departments
        var departments = _employeeRepository.GetAllDepartment();
        ViewData["DepartmentId"] = new SelectList(departments, "Id", "Name");
        
        return View();
    }

    // ═══════════════════════════════════════════════════════════
    // POST: /Employee/Create
    // ═══════════════════════════════════════════════════════════
    [HttpPost]
    [ValidateAntiForgeryToken]
    public ActionResult Create(Employee employee)
    {
        // Line 2: Get departments for dropdown (needed if validation fails)
        var departments = _employeeRepository.GetAllDepartment();

        // Line 3: Check ModelState
        if (ModelState.IsValid)
        {
            // Line 4: All validations passed
            try
            {
                _employeeRepository.Add(employee);
                _logger.LogInformation($"Employee {employee.Name} created successfully");
                
                // Line 5: Redirect to Index after successful create
                return RedirectToAction(nameof(Index));
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error creating employee");
                ModelState.AddModelError(string.Empty, "An error occurred while saving");
            }
        }
        
        // Line 6: Validation failed - reload dropdown and return view
        ViewData["DepartmentId"] = new SelectList(departments, "Id", "Name");
        return View(employee);
    }

    // ═══════════════════════════════════════════════════════════
    // GET: /Employee/Edit/5
    // ═══════════════════════════════════════════════════════════
    public ActionResult Edit(int id)
    {
        var employee = _employeeRepository.GetEmployee(id);
        
        if (employee == null)
        {
            return NotFound();
        }
        
        // Populate dropdown with current value selected
        var departments = _employeeRepository.GetAllDepartment();
        ViewData["DepartmentId"] = new SelectList(
            departments, "Id", "Name", employee.DepartmentId);
        
        return View(employee);
    }

    // ═══════════════════════════════════════════════════════════
    // POST: /Employee/Edit/5
    // ═══════════════════════════════════════════════════════════
    [HttpPost]
    [ValidateAntiForgeryToken]
    public ActionResult Edit(Employee emp)
    {
        if (ModelState.IsValid)
        {
            try
            {
                _employeeRepository.Update(emp);
                return RedirectToAction(nameof(Index));
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error updating employee");
                ModelState.AddModelError(string.Empty, "Error updating employee");
            }
        }
        
        // Reload dropdown on validation failure
        var departments = _employeeRepository.GetAllDepartment();
        ViewData["DepartmentId"] = new SelectList(
            departments, "Id", "Name", emp.DepartmentId);
        
        return View(emp);
    }
}
```

---

## Best Practices

### 1. Always Check ModelState

```csharp
// ✅ GOOD: Always check before processing
[HttpPost]
public ActionResult Create(Employee emp)
{
    if (!ModelState.IsValid)
    {
        return View(emp);  // Return with errors
    }
    // Process valid data
}

// ❌ BAD: Skipping validation check
[HttpPost]
public ActionResult Create(Employee emp)
{
    _repository.Add(emp);  // Saving potentially invalid data!
}
```

### 2. Use Appropriate Validation Attributes

```csharp
// ✅ GOOD: Specific validation for each field
[Required, EmailAddress]
public string Email { get; set; }

[Required, Phone]
public string Phone { get; set; }

[Range(0, 100)]
public int Percentage { get; set; }

// ❌ BAD: Only checking Required
[Required]  // Missing format validation
public string Email { get; set; }
```

### 3. Provide Meaningful Error Messages

```csharp
// ✅ GOOD: Helpful error messages
[Required(ErrorMessage = "Please enter your email address")]
[EmailAddress(ErrorMessage = "Please enter a valid email format (e.g., user@domain.com)")]
public string Email { get; set; }

// ❌ BAD: Generic or missing messages
[Required]
[EmailAddress]
public string Email { get; set; }
// Default message: "The Email field is not a valid e-mail address."
```

---

## Interview Questions

### Q1: What is Model Binding in ASP.NET Core?
**Answer:** Model Binding is the process of mapping HTTP request data (form values, route data, query strings, request body) to action method parameters and model objects. It automatically converts string values to appropriate types and populates complex objects.

### Q2: What is the difference between [FromBody] and [FromForm]?
**Answer:**
- `[FromBody]`: Binds data from the request body, typically JSON. Used with APIs.
- `[FromForm]`: Binds data from form fields (multipart/form-data or application/x-www-form-urlencoded). Used with HTML forms.

### Q3: How do you validate a model in ASP.NET Core?
**Answer:** Using Data Annotations (validation attributes) on model properties and checking `ModelState.IsValid` in the controller action. Common attributes include `[Required]`, `[StringLength]`, `[Range]`, `[EmailAddress]`.

### Q4: What is ModelState?
**Answer:** ModelState is a dictionary that contains the state of model binding and validation. It tracks values, errors, and validation states for each property. `ModelState.IsValid` returns true only if all validations pass.

### Q5: How do you create custom validation?
**Answer:** Two ways:
1. Create a class inheriting from `ValidationAttribute` and override `IsValid()`
2. Implement `IValidatableObject` interface on the model for cross-property validation

### Q6: What is client-side validation?
**Answer:** Client-side validation runs in the browser before form submission using JavaScript. ASP.NET Core uses jQuery Validation and Unobtrusive Validation libraries that read `data-val-*` attributes generated by tag helpers.

---

## Quick Reference

### Binding Source Attributes

| Attribute | Source |
|-----------|--------|
| `[FromRoute]` | URL route values |
| `[FromQuery]` | Query string |
| `[FromForm]` | Form data |
| `[FromBody]` | Request body (JSON) |
| `[FromHeader]` | HTTP headers |
| `[FromServices]` | DI container |

### Common Validation Attributes

```csharp
[Required]
[StringLength(100, MinimumLength = 2)]
[Range(1, 100)]
[EmailAddress]
[Phone]
[Url]
[Compare("Password")]
[RegularExpression(@"^[A-Z]+$")]
```

### ModelState Methods

```csharp
ModelState.IsValid           // Check if valid
ModelState.AddModelError()   // Add custom error
ModelState.Clear()           // Clear all
ModelState.Remove("key")     // Remove entry
ModelState.ErrorCount        // Error count
```
