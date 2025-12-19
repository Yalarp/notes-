# 🗃️ Entity Framework Core Introduction

## 📚 Table of Contents
- [Overview](#-overview)
- [Setting Up EF Core](#-setting-up-ef-core)
- [DbContext](#-dbcontext)
- [Basic CRUD Operations](#-basic-crud-operations)
- [Migrations](#-migrations)
- [Interview Questions](#-interview-questions)

---

## 🎯 Overview

**Entity Framework Core (EF Core)** is an ORM (Object-Relational Mapper) that enables .NET developers to work with databases using .NET objects.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    EF CORE ARCHITECTURE                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   C# Application                                                    │
│        │                                                            │
│        ▼                                                            │
│   ┌──────────────┐                                                  │
│   │   DbContext  │  ◄── Unit of Work + Repository                  │
│   └──────────────┘                                                  │
│        │                                                            │
│        ▼                                                            │
│   ┌──────────────┐                                                  │
│   │  DbSet<T>    │  ◄── Collection of entities                     │
│   └──────────────┘                                                  │
│        │                                                            │
│        ▼                                                            │
│   ┌──────────────┐                                                  │
│   │   Entity     │  ◄── POCO class mapped to table                 │
│   └──────────────┘                                                  │
│        │                                                            │
│        ▼                                                            │
│   [Database: SQL Server / SQLite / PostgreSQL / MySQL]             │
│                                                                     │
│   Benefits:                                                         │
│   • Write C# instead of SQL                                        │
│   • Type-safe queries with LINQ                                    │
│   • Automatic change tracking                                      │
│   • Database migrations                                            │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔷 Setting Up EF Core

### Install NuGet Packages

```bash
# For SQL Server
dotnet add package Microsoft.EntityFrameworkCore.SqlServer

# For SQLite
dotnet add package Microsoft.EntityFrameworkCore.Sqlite

# Tools for migrations
dotnet add package Microsoft.EntityFrameworkCore.Tools
dotnet add package Microsoft.EntityFrameworkCore.Design
```

### Create Entity Classes

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// ENTITY CLASSES - POCO (Plain Old CLR Objects)
// ═══════════════════════════════════════════════════════════════════════════

public class Employee
{
    public int Id { get; set; }           // Primary Key (convention: Id or EmployeeId)
    public string Name { get; set; }
    public string Email { get; set; }
    public decimal Salary { get; set; }
    public int DepartmentId { get; set; } // Foreign Key
    
    // Navigation property
    public Department Department { get; set; }
}

public class Department
{
    public int Id { get; set; }
    public string Name { get; set; }
    
    // Navigation property - collection
    public ICollection<Employee> Employees { get; set; }
}
```

---

## 🔶 DbContext

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// DBCONTEXT - DATABASE SESSION
// ═══════════════════════════════════════════════════════════════════════════

using Microsoft.EntityFrameworkCore;

public class AppDbContext : DbContext
{
    // DbSet = Table
    public DbSet<Employee> Employees { get; set; }
    public DbSet<Department> Departments { get; set; }
    
    // Constructor for DI
    public AppDbContext(DbContextOptions<AppDbContext> options)
        : base(options)
    {
    }
    
    // ─────────────────────────────────────────────────────────────────────
    // FLUENT API CONFIGURATION
    // ─────────────────────────────────────────────────────────────────────
    
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // Configure Employee entity
        modelBuilder.Entity<Employee>(entity =>
        {
            entity.HasKey(e => e.Id);
            
            entity.Property(e => e.Name)
                .IsRequired()
                .HasMaxLength(100);
            
            entity.Property(e => e.Salary)
                .HasColumnType("decimal(18,2)");
            
            // Relationship
            entity.HasOne(e => e.Department)
                .WithMany(d => d.Employees)
                .HasForeignKey(e => e.DepartmentId);
        });
        
        // Seed data
        modelBuilder.Entity<Department>().HasData(
            new Department { Id = 1, Name = "IT" },
            new Department { Id = 2, Name = "HR" }
        );
    }
}

// ─────────────────────────────────────────────────────────────────────────
// REGISTER IN PROGRAM.CS
// ─────────────────────────────────────────────────────────────────────────

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("Default")));

// appsettings.json
{
  "ConnectionStrings": {
    "Default": "Server=.;Database=MyApp;Trusted_Connection=True;"
  }
}
```

---

## 🔵 Basic CRUD Operations

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// CRUD WITH EF CORE
// ═══════════════════════════════════════════════════════════════════════════

public class EmployeeService
{
    private readonly AppDbContext _context;
    
    public EmployeeService(AppDbContext context)
    {
        _context = context;
    }
    
    // ═══════════════════════════════════════════════════════════════════
    // CREATE
    // ═══════════════════════════════════════════════════════════════════
    
    public async Task<Employee> CreateAsync(Employee employee)
    {
        _context.Employees.Add(employee);
        // Line 1: Add to DbSet (marks as Added)
        
        await _context.SaveChangesAsync();
        // Line 2: Persist to database
        //         Employee.Id is auto-populated
        
        return employee;
    }
    
    // Add multiple
    public async Task AddRangeAsync(IEnumerable<Employee> employees)
    {
        _context.Employees.AddRange(employees);
        await _context.SaveChangesAsync();
    }
    
    // ═══════════════════════════════════════════════════════════════════
    // READ
    // ═══════════════════════════════════════════════════════════════════
    
    // Get by ID
    public async Task<Employee> GetByIdAsync(int id)
    {
        return await _context.Employees.FindAsync(id);
        // Line 3: FindAsync checks cache first, then DB
    }
    
    // Get all
    public async Task<List<Employee>> GetAllAsync()
    {
        return await _context.Employees.ToListAsync();
        // Line 4: ToListAsync executes query
    }
    
    // With filter
    public async Task<List<Employee>> GetByDepartmentAsync(int deptId)
    {
        return await _context.Employees
            .Where(e => e.DepartmentId == deptId)
            .ToListAsync();
    }
    
    // Include related data (eager loading)
    public async Task<Employee> GetWithDepartmentAsync(int id)
    {
        return await _context.Employees
            .Include(e => e.Department)
            // Line 5: Include loads related Department
            .FirstOrDefaultAsync(e => e.Id == id);
    }
    
    // ═══════════════════════════════════════════════════════════════════
    // UPDATE
    // ═══════════════════════════════════════════════════════════════════
    
    public async Task UpdateAsync(Employee employee)
    {
        // Option 1: Track changes (if already tracked)
        var existing = await _context.Employees.FindAsync(employee.Id);
        if (existing != null)
        {
            existing.Name = employee.Name;
            existing.Salary = employee.Salary;
            // Line 6: Changes auto-tracked
        }
        
        await _context.SaveChangesAsync();
    }
    
    // Option 2: Attach and mark modified
    public async Task UpdateExplicitAsync(Employee employee)
    {
        _context.Employees.Update(employee);
        // Line 7: Marks all properties as Modified
        
        await _context.SaveChangesAsync();
    }
    
    // ═══════════════════════════════════════════════════════════════════
    // DELETE
    // ═══════════════════════════════════════════════════════════════════
    
    public async Task DeleteAsync(int id)
    {
        var employee = await _context.Employees.FindAsync(id);
        if (employee != null)
        {
            _context.Employees.Remove(employee);
            // Line 8: Mark for deletion
            
            await _context.SaveChangesAsync();
        }
    }
}
```

---

## 🟢 Migrations

```bash
# ═══════════════════════════════════════════════════════════════════════════
# DATABASE MIGRATIONS
# ═══════════════════════════════════════════════════════════════════════════

# Create initial migration
dotnet ef migrations add InitialCreate

# Apply migrations to database
dotnet ef database update

# Remove last migration (if not applied)
dotnet ef migrations remove

# Generate SQL script instead of applying
dotnet ef migrations script

# Revert to specific migration
dotnet ef database update MigrationName
```

### Migration Files

```csharp
// Generated migration (Migrations/20231201_InitialCreate.cs)
public partial class InitialCreate : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.CreateTable(
            name: "Departments",
            columns: table => new
            {
                Id = table.Column<int>(nullable: false)
                    .Annotation("SqlServer:Identity", "1, 1"),
                Name = table.Column<string>(maxLength: 100, nullable: false)
            },
            constraints: table =>
            {
                table.PrimaryKey("PK_Departments", x => x.Id);
            });
        
        // More tables...
    }
    
    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.DropTable(name: "Employees");
        migrationBuilder.DropTable(name: "Departments");
    }
}
```

---

## 🎤 Interview Questions

**Q1: What is EF Core?**
> Object-Relational Mapper (ORM) that lets you work with databases using C# objects instead of SQL.

**Q2: What is DbContext?**
> Represents a session with the database. Contains DbSet properties for each entity table.

**Q3: What are migrations?**
> Code-based way to version and update database schema. Creates/modifies tables based on entity changes.

**Q4: FindAsync vs FirstOrDefaultAsync?**
> FindAsync checks cache first, works only with primary key. FirstOrDefaultAsync always queries database, supports any condition.

**Q5: What is change tracking?**
> EF Core automatically tracks property changes on entities. SaveChanges generates SQL based on tracked changes.

---

## 📝 Summary

| Concept | Description |
|---------|-------------|
| **Entity** | POCO class mapped to table |
| **DbContext** | Database session |
| **DbSet<T>** | Collection/Table of entities |
| **Migration** | Database schema versioning |
| **SaveChanges** | Persist tracked changes |
| **Include** | Eager load related data |
