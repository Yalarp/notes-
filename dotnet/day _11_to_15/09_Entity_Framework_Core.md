# Entity Framework Core in ASP.NET Core

## Table of Contents
1. [Introduction](#introduction)
2. [What is Entity Framework Core?](#what-is-entity-framework-core)
3. [DbContext](#dbcontext)
4. [Models and Entities](#models-and-entities)
5. [Database Operations (CRUD)](#database-operations-crud)
6. [LINQ Queries](#linq-queries)
7. [Migrations](#migrations)
8. [Relationships](#relationships)
9. [Code Examples](#code-examples)
10. [Best Practices](#best-practices)
11. [Interview Questions](#interview-questions)

---

## Introduction

### Real-World Analogy
Entity Framework Core is like a **translator between your code and database**:

```
Without EF Core:
    Developer → Writes SQL manually → Database
    ✗ Tedious, error-prone, database-specific

With EF Core:
    Developer → Uses C# objects → EF Core translates → SQL → Database
    ✓ Type-safe, IntelliSense, database-agnostic
```

---

## What is Entity Framework Core?

Entity Framework Core (EF Core) is an Object-Relational Mapper (ORM) that allows .NET developers to work with databases using C# objects.

### EF Core Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    EF CORE ARCHITECTURE                          │
└─────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────┐
│                    Your Application                           │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐   │
│  │                      Models                           │   │
│  │    Employee.cs, Department.cs (C# classes)            │   │
│  └──────────────────────────────────────────────────────┘   │
│                           │                                  │
│                           ▼                                  │
│  ┌──────────────────────────────────────────────────────┐   │
│  │                     DbContext                         │   │
│  │    AppDbContext : DbContext                           │   │
│  │    DbSet<Employee>, DbSet<Department>                 │   │
│  └──────────────────────────────────────────────────────┘   │
│                           │                                  │
│                           ▼                                  │
│  ┌──────────────────────────────────────────────────────┐   │
│  │                  EF Core Engine                       │   │
│  │    Change Tracking, Query Translation, Migrations    │   │
│  └──────────────────────────────────────────────────────┘   │
│                           │                                  │
│                           ▼                                  │
│  ┌──────────────────────────────────────────────────────┐   │
│  │                Database Provider                      │   │
│  │    SQL Server, PostgreSQL, SQLite, MySQL              │   │
│  └──────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────┘
                            │
                            ▼
                     ┌─────────────┐
                     │  Database   │
                     │  (SQL)      │
                     └─────────────┘
```

### Key Features

| Feature | Description |
|---------|-------------|
| **LINQ Support** | Query database using C# LINQ syntax |
| **Change Tracking** | Automatically tracks entity changes |
| **Migrations** | Version control for database schema |
| **Lazy/Eager Loading** | Control when related data loads |
| **Cross-Platform** | Works on Windows, Linux, macOS |

---

## DbContext

### What is DbContext?

DbContext is the primary class for interacting with the database. It represents a session with the database.

### Creating DbContext

```csharp
// File: Repository/AppdbContextRepository.cs

using Microsoft.EntityFrameworkCore;

public class AppdbContextRepository : DbContext
{
    // ═══════════════════════════════════════════════════════════
    // CONSTRUCTOR
    // ═══════════════════════════════════════════════════════════
    
    // Line 1: Constructor receives options from DI container
    public AppdbContextRepository(DbContextOptions<AppdbContextRepository> options)
        : base(options)
    {
    }
    
    // ═══════════════════════════════════════════════════════════
    // DBSETS - Each represents a table
    // ═══════════════════════════════════════════════════════════
    
    // Line 2: DbSet<Employee> maps to Employees table
    public DbSet<Employee> Employees { get; set; }
    
    // Line 3: DbSet<Department> maps to Departments table
    public DbSet<Department> Departments { get; set; }
    
    // ═══════════════════════════════════════════════════════════
    // FLUENT API CONFIGURATION
    // ═══════════════════════════════════════════════════════════
    
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // Line 4: Configure Employee entity
        modelBuilder.Entity<Employee>(entity =>
        {
            entity.HasKey(e => e.Id);
            entity.Property(e => e.Name)
                .IsRequired()
                .HasMaxLength(100);
            entity.Property(e => e.Email)
                .HasMaxLength(200);
        });
        
        // Line 5: Configure relationship
        modelBuilder.Entity<Employee>()
            .HasOne(e => e.Department)
            .WithMany(d => d.Employees)
            .HasForeignKey(e => e.DepartmentId);
            
        // Line 6: Seed initial data
        modelBuilder.Entity<Department>().HasData(
            new Department { Id = 1, Name = "IT" },
            new Department { Id = 2, Name = "HR" }
        );
    }
}
```

### Registering DbContext

```csharp
// File: Program.cs

var builder = WebApplication.CreateBuilder(args);

// ═══════════════════════════════════════════════════════════════
// REGISTER DBCONTEXT
// ═══════════════════════════════════════════════════════════════

// Line 1: Get connection string from appsettings.json
var connectionString = builder.Configuration
    .GetConnectionString("EmployeeDBConnection");

// Line 2: Register DbContext with connection pooling
builder.Services.AddDbContextPool<AppdbContextRepository>(options =>
    options.UseSqlServer(connectionString));

// appsettings.json:
// "ConnectionStrings": {
//   "EmployeeDBConnection": "Server=.;Database=EmployeeDB;Trusted_Connection=True;"
// }
```

---

## Models and Entities

### Creating Entity Classes

```csharp
// File: Models/Employee.cs

using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

public class Employee
{
    // ═══════════════════════════════════════════════════════════
    // PRIMARY KEY
    // ═══════════════════════════════════════════════════════════
    
    [Key]  // Primary key (optional if named 'Id')
    public int Id { get; set; }
    
    // ═══════════════════════════════════════════════════════════
    // REQUIRED PROPERTIES
    // ═══════════════════════════════════════════════════════════
    
    [Required]
    [StringLength(100)]
    public string Name { get; set; }
    
    [Required]
    [EmailAddress]
    public string Email { get; set; }
    
    // ═══════════════════════════════════════════════════════════
    // OPTIONAL PROPERTIES
    // ═══════════════════════════════════════════════════════════
    
    [Column(TypeName = "decimal(18,2)")]
    public decimal Salary { get; set; }
    
    public DateTime HireDate { get; set; }
    
    // ═══════════════════════════════════════════════════════════
    // FOREIGN KEY
    // ═══════════════════════════════════════════════════════════
    
    [ForeignKey("Department")]
    public int DepartmentId { get; set; }
    
    // ═══════════════════════════════════════════════════════════
    // NAVIGATION PROPERTY
    // ═══════════════════════════════════════════════════════════
    
    public Department Department { get; set; }
}
```

```csharp
// File: Models/Department.cs

public class Department
{
    [Key]
    public int Id { get; set; }
    
    [Required]
    [StringLength(50)]
    public string Name { get; set; }
    
    // Navigation property - collection of employees
    public ICollection<Employee> Employees { get; set; }
}
```

### Data Annotations

| Annotation | Description |
|------------|-------------|
| `[Key]` | Primary key |
| `[Required]` | Non-nullable field |
| `[StringLength(n)]` | Maximum string length |
| `[Column("Name")]` | Column name in database |
| `[Table("TableName")]` | Table name |
| `[ForeignKey("Property")]` | Foreign key relationship |
| `[NotMapped]` | Property not in database |

---

## Database Operations (CRUD)

### Create (Insert)

```csharp
// ═══════════════════════════════════════════════════════════════
// ADD SINGLE ENTITY
// ═══════════════════════════════════════════════════════════════

public async Task<Employee> AddAsync(Employee employee)
{
    // Line 1: Add to DbSet (tracked as Added)
    await _context.Employees.AddAsync(employee);
    
    // Line 2: Save changes to database (generates INSERT)
    await _context.SaveChangesAsync();
    
    return employee;  // Now has Id assigned
}

// ═══════════════════════════════════════════════════════════════
// ADD MULTIPLE ENTITIES
// ═══════════════════════════════════════════════════════════════

public async Task AddRangeAsync(IEnumerable<Employee> employees)
{
    await _context.Employees.AddRangeAsync(employees);
    await _context.SaveChangesAsync();
}
```

### Read (Select)

```csharp
// ═══════════════════════════════════════════════════════════════
// GET BY ID
// ═══════════════════════════════════════════════════════════════

public async Task<Employee?> GetByIdAsync(int id)
{
    // FindAsync uses primary key - efficient
    return await _context.Employees.FindAsync(id);
}

// ═══════════════════════════════════════════════════════════════
// GET ALL
// ═══════════════════════════════════════════════════════════════

public async Task<List<Employee>> GetAllAsync()
{
    return await _context.Employees.ToListAsync();
}

// ═══════════════════════════════════════════════════════════════
// GET WITH RELATED DATA (EAGER LOADING)
// ═══════════════════════════════════════════════════════════════

public async Task<List<Employee>> GetAllWithDepartmentAsync()
{
    return await _context.Employees
        .Include(e => e.Department)  // Load related Department
        .ToListAsync();
}

// ═══════════════════════════════════════════════════════════════
// GET WITH FILTER
// ═══════════════════════════════════════════════════════════════

public async Task<List<Employee>> GetByDepartmentAsync(int deptId)
{
    return await _context.Employees
        .Where(e => e.DepartmentId == deptId)
        .OrderBy(e => e.Name)
        .ToListAsync();
}

// ═══════════════════════════════════════════════════════════════
// GET FIRST OR DEFAULT
// ═══════════════════════════════════════════════════════════════

public async Task<Employee?> GetByEmailAsync(string email)
{
    return await _context.Employees
        .FirstOrDefaultAsync(e => e.Email == email);
}
```

### Update

```csharp
// ═══════════════════════════════════════════════════════════════
// UPDATE - Method 1: Modify tracked entity
// ═══════════════════════════════════════════════════════════════

public async Task<Employee?> UpdateAsync(int id, Employee updated)
{
    // Line 1: Get existing entity (tracked)
    var employee = await _context.Employees.FindAsync(id);
    
    if (employee == null)
        return null;
    
    // Line 2: Update properties
    employee.Name = updated.Name;
    employee.Email = updated.Email;
    employee.Salary = updated.Salary;
    employee.DepartmentId = updated.DepartmentId;
    
    // Line 3: Save changes (generates UPDATE)
    await _context.SaveChangesAsync();
    
    return employee;
}

// ═══════════════════════════════════════════════════════════════
// UPDATE - Method 2: Attach and mark as modified
// ═══════════════════════════════════════════════════════════════

public async Task UpdateAttachedAsync(Employee employee)
{
    _context.Employees.Update(employee);  // Mark entire entity modified
    await _context.SaveChangesAsync();
}
```

### Delete

```csharp
// ═══════════════════════════════════════════════════════════════
// DELETE BY ID
// ═══════════════════════════════════════════════════════════════

public async Task<bool> DeleteAsync(int id)
{
    // Line 1: Find entity
    var employee = await _context.Employees.FindAsync(id);
    
    if (employee == null)
        return false;
    
    // Line 2: Remove from DbSet (mark as Deleted)
    _context.Employees.Remove(employee);
    
    // Line 3: Save changes (generates DELETE)
    await _context.SaveChangesAsync();
    
    return true;
}

// ═══════════════════════════════════════════════════════════════
// DELETE MULTIPLE
// ═══════════════════════════════════════════════════════════════

public async Task DeleteByDepartmentAsync(int deptId)
{
    var employees = await _context.Employees
        .Where(e => e.DepartmentId == deptId)
        .ToListAsync();
    
    _context.Employees.RemoveRange(employees);
    await _context.SaveChangesAsync();
}
```

---

## LINQ Queries

### Common Query Operations

```csharp
// ═══════════════════════════════════════════════════════════════
// FILTERING (WHERE)
// ═══════════════════════════════════════════════════════════════

var activeEmployees = await _context.Employees
    .Where(e => e.IsActive && e.Salary > 50000)
    .ToListAsync();

// ═══════════════════════════════════════════════════════════════
// SORTING (ORDER BY)
// ═══════════════════════════════════════════════════════════════

var sorted = await _context.Employees
    .OrderBy(e => e.Department.Name)
    .ThenByDescending(e => e.Salary)
    .ToListAsync();

// ═══════════════════════════════════════════════════════════════
// PROJECTION (SELECT)
// ═══════════════════════════════════════════════════════════════

var names = await _context.Employees
    .Select(e => new { e.Id, e.Name, DeptName = e.Department.Name })
    .ToListAsync();

// ═══════════════════════════════════════════════════════════════
// PAGING (SKIP/TAKE)
// ═══════════════════════════════════════════════════════════════

int pageSize = 10;
int pageNumber = 2;

var pagedEmployees = await _context.Employees
    .Skip((pageNumber - 1) * pageSize)
    .Take(pageSize)
    .ToListAsync();

// ═══════════════════════════════════════════════════════════════
// AGGREGATIONS
// ═══════════════════════════════════════════════════════════════

var count = await _context.Employees.CountAsync();
var maxSalary = await _context.Employees.MaxAsync(e => e.Salary);
var avgSalary = await _context.Employees.AverageAsync(e => e.Salary);
var totalSalary = await _context.Employees.SumAsync(e => e.Salary);

// ═══════════════════════════════════════════════════════════════
// GROUPING
// ═══════════════════════════════════════════════════════════════

var byDepartment = await _context.Employees
    .GroupBy(e => e.DepartmentId)
    .Select(g => new
    {
        DepartmentId = g.Key,
        EmployeeCount = g.Count(),
        AvgSalary = g.Average(e => e.Salary)
    })
    .ToListAsync();

// ═══════════════════════════════════════════════════════════════
// CHECKING EXISTENCE
// ═══════════════════════════════════════════════════════════════

var exists = await _context.Employees
    .AnyAsync(e => e.Email == "test@example.com");
```

---

## Migrations

### Creating and Applying Migrations

```bash
# ═══════════════════════════════════════════════════════════════
# PACKAGE MANAGER CONSOLE COMMANDS
# ═══════════════════════════════════════════════════════════════

# Create initial migration
Add-Migration InitialCreate

# Create named migration
Add-Migration AddEmployeeTable

# Apply migrations to database
Update-Database

# Remove last migration (not applied)
Remove-Migration

# Script migrations (for production)
Script-Migration

# ═══════════════════════════════════════════════════════════════
# .NET CLI COMMANDS
# ═══════════════════════════════════════════════════════════════

# Create migration
dotnet ef migrations add InitialCreate

# Apply migrations
dotnet ef database update

# Remove last migration
dotnet ef migrations remove

# Generate SQL script
dotnet ef migrations script
```

### Migration File Structure

```csharp
// File: Migrations/20241219_AddEmployeeTable.cs

public partial class AddEmployeeTable : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        // Creates the table
        migrationBuilder.CreateTable(
            name: "Employees",
            columns: table => new
            {
                Id = table.Column<int>(nullable: false)
                    .Annotation("SqlServer:Identity", "1, 1"),
                Name = table.Column<string>(maxLength: 100, nullable: false),
                Email = table.Column<string>(maxLength: 200, nullable: false),
                Salary = table.Column<decimal>(type: "decimal(18,2)", nullable: false),
                DepartmentId = table.Column<int>(nullable: false)
            },
            constraints: table =>
            {
                table.PrimaryKey("PK_Employees", x => x.Id);
                table.ForeignKey(
                    name: "FK_Employees_Departments_DepartmentId",
                    column: x => x.DepartmentId,
                    principalTable: "Departments",
                    principalColumn: "Id",
                    onDelete: ReferentialAction.Cascade);
            });
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        // Reverts the change
        migrationBuilder.DropTable(name: "Employees");
    }
}
```

---

## Relationships

### One-to-Many

```csharp
// Department has many Employees
public class Department
{
    public int Id { get; set; }
    public string Name { get; set; }
    
    // Navigation property
    public ICollection<Employee> Employees { get; set; }
}

public class Employee
{
    public int Id { get; set; }
    public string Name { get; set; }
    
    // Foreign key
    public int DepartmentId { get; set; }
    
    // Navigation property
    public Department Department { get; set; }
}
```

### Loading Related Data

```csharp
// ═══════════════════════════════════════════════════════════════
// EAGER LOADING (Include)
// ═══════════════════════════════════════════════════════════════

var employees = await _context.Employees
    .Include(e => e.Department)  // Load Department with Employee
    .ToListAsync();

// Multiple levels
var departments = await _context.Departments
    .Include(d => d.Employees)
        .ThenInclude(e => e.Projects)  // Employee's projects
    .ToListAsync();

// ═══════════════════════════════════════════════════════════════
// EXPLICIT LOADING
// ═══════════════════════════════════════════════════════════════

var employee = await _context.Employees.FindAsync(1);

// Load related data explicitly
await _context.Entry(employee)
    .Reference(e => e.Department)
    .LoadAsync();

// Load collection
await _context.Entry(employee)
    .Collection(e => e.Projects)
    .LoadAsync();

// ═══════════════════════════════════════════════════════════════
// LAZY LOADING (requires proxies package)
// ═══════════════════════════════════════════════════════════════

// Configure in Program.cs
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseLazyLoadingProxies()
           .UseSqlServer(connectionString));

// Properties must be virtual
public class Employee
{
    public virtual Department Department { get; set; }  // Lazy loaded
}
```

---

## Code Examples

### Complete Repository Implementation

```csharp
// File: Services/SqlEmployeeService.cs

public class SqlEmployeeService : IEmployeeService
{
    private readonly AppdbContextRepository _context;
    private readonly ILogger<SqlEmployeeService> _logger;

    public SqlEmployeeService(
        AppdbContextRepository context,
        ILogger<SqlEmployeeService> logger)
    {
        _context = context;
        _logger = logger;
    }

    public List<Employee> GetAllEmployee()
    {
        _logger.LogInformation("Getting all employees with departments");
        
        return _context.Employees
            .Include(e => e.Department)
            .ToList();
    }

    public Employee GetEmployee(int id)
    {
        return _context.Employees
            .Include(e => e.Department)
            .FirstOrDefault(e => e.Id == id);
    }

    public Employee Add(Employee employee)
    {
        _context.Employees.Add(employee);
        _context.SaveChanges();
        
        _logger.LogInformation("Added employee {Name} with ID {Id}", 
            employee.Name, employee.Id);
        
        return employee;
    }

    public Employee Update(Employee updated)
    {
        var employee = _context.Employees.Find(updated.Id);
        
        if (employee != null)
        {
            employee.Name = updated.Name;
            employee.Email = updated.Email;
            employee.Salary = updated.Salary;
            employee.DepartmentId = updated.DepartmentId;
            
            _context.SaveChanges();
            
            _logger.LogInformation("Updated employee {Id}", updated.Id);
        }
        
        return employee;
    }

    public void Delete(int id)
    {
        var employee = _context.Employees.Find(id);
        
        if (employee != null)
        {
            _context.Employees.Remove(employee);
            _context.SaveChanges();
            
            _logger.LogInformation("Deleted employee {Id}", id);
        }
    }

    public List<Department> GetAllDepartment()
    {
        return _context.Departments.ToList();
    }
}
```

---

## Best Practices

### 1. Use AddDbContextPool for Performance

```csharp
// ✅ GOOD: Connection pooling
builder.Services.AddDbContextPool<AppDbContext>(options =>
    options.UseSqlServer(connectionString));

// ❌ Less efficient for high-traffic apps
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(connectionString));
```

### 2. Use AsNoTracking for Read-Only Queries

```csharp
// ✅ GOOD: Better performance for read-only
var employees = await _context.Employees
    .AsNoTracking()
    .ToListAsync();

// ❌ Unnecessary tracking overhead
var employees = await _context.Employees
    .ToListAsync();
```

### 3. Project Only Needed Fields

```csharp
// ✅ GOOD: Select only needed columns
var names = await _context.Employees
    .Select(e => new { e.Id, e.Name })
    .ToListAsync();

// ❌ BAD: Loading entire entity when not needed
var employees = await _context.Employees.ToListAsync();
var names = employees.Select(e => e.Name);
```

### 4. Avoid N+1 Query Problem

```csharp
// ❌ BAD: N+1 queries (1 for employees + N for departments)
var employees = await _context.Employees.ToListAsync();
foreach (var emp in employees)
{
    Console.WriteLine(emp.Department.Name);  // Lazy load each!
}

// ✅ GOOD: Single query with Include
var employees = await _context.Employees
    .Include(e => e.Department)
    .ToListAsync();
```

---

## Interview Questions

### Q1: What is the difference between DbContext and DbSet?
**Answer:**
- `DbContext`: Represents a session with the database, manages connections and transactions
- `DbSet<T>`: Represents a table in the database, used for CRUD operations on that entity

### Q2: What are the different loading strategies in EF Core?
**Answer:**
- **Eager Loading**: Use `.Include()` to load related data with main query
- **Lazy Loading**: Related data loaded automatically when accessed (requires proxies)
- **Explicit Loading**: Manually load related data using `.Load()`

### Q3: What is the N+1 query problem?
**Answer:** When iterating over a collection and accessing related data causes one additional query for each item (N items = N+1 queries). Solved using Eager Loading with Include().

### Q4: What is AsNoTracking?
**Answer:** Disables change tracking for a query, improving performance for read-only scenarios. The returned entities cannot be updated.

### Q5: What is the difference between Add and AddAsync?
**Answer:**
- `Add`: Synchronous, uses memory-only operations
- `AddAsync`: Useful when using value generators that need database access (rare)

For most cases, `Add` is sufficient since actual database I/O happens in `SaveChangesAsync`.

### Q6: What is a migration in EF Core?
**Answer:** A migration is a code representation of database schema changes. It allows version control of database schema and enables updating/reverting database structure.

---

## Quick Reference

### Common Methods

```csharp
// Create
_context.Entities.Add(entity);
_context.Entities.AddRange(entities);
await _context.SaveChangesAsync();

// Read
await _context.Entities.FindAsync(id);
await _context.Entities.FirstOrDefaultAsync(e => e.Id == id);
await _context.Entities.ToListAsync();
await _context.Entities.Include(e => e.Related).ToListAsync();

// Update
var entity = await _context.Entities.FindAsync(id);
entity.Property = newValue;
await _context.SaveChangesAsync();

// Delete
_context.Entities.Remove(entity);
await _context.SaveChangesAsync();
```

### LINQ Extensions

```csharp
.Where(predicate)          // Filter
.Select(projection)        // Project
.OrderBy()/OrderByDescending()  // Sort
.Skip(n).Take(n)          // Paging
.Include(navigation)      // Eager load
.AsNoTracking()           // Read-only
.FirstOrDefaultAsync()    // First or null
.AnyAsync(predicate)      // Exists check
.CountAsync()             // Count
```
