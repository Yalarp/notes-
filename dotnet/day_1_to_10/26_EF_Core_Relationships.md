# 🔗 EF Core Relationships and Data Annotations

## 📚 Table of Contents
- [Relationship Types](#-relationship-types)
- [One-to-Many](#-one-to-many)
- [One-to-One](#-one-to-one)
- [Many-to-Many](#-many-to-many)
- [Data Annotations](#-data-annotations)
- [Interview Questions](#-interview-questions)

---

## 🎯 Relationship Types

```
┌─────────────────────────────────────────────────────────────────────┐
│                    EF CORE RELATIONSHIPS                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ONE-TO-MANY (Most Common)                                         │
│  Department ────────────< Employee                                 │
│  (1)                       (Many)                                  │
│  One department has many employees                                 │
│                                                                     │
│  ONE-TO-ONE                                                         │
│  Person ─────────────── Address                                    │
│  (1)                      (1)                                      │
│  One person has one address                                        │
│                                                                     │
│  MANY-TO-MANY                                                       │
│  Student >────────────< Course                                     │
│  (Many)                  (Many)                                    │
│  Students enroll in multiple courses, courses have many students   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔷 One-to-Many

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// ONE-TO-MANY: Department -> Employees
// ═══════════════════════════════════════════════════════════════════════════

// Principal entity (one side)
public class Department
{
    public int Id { get; set; }
    public string Name { get; set; }
    
    // Navigation property - collection
    public ICollection<Employee> Employees { get; set; }
}

// Dependent entity (many side)
public class Employee
{
    public int Id { get; set; }
    public string Name { get; set; }
    
    // Foreign key property (optional but recommended)
    public int DepartmentId { get; set; }
    
    // Navigation property - reference
    public Department Department { get; set; }
}

// ─────────────────────────────────────────────────────────────────────────
// FLUENT API CONFIGURATION
// ─────────────────────────────────────────────────────────────────────────

protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Employee>()
        .HasOne(e => e.Department)           // Employee has one Department
        .WithMany(d => d.Employees)          // Department has many Employees
        .HasForeignKey(e => e.DepartmentId)  // FK property
        .OnDelete(DeleteBehavior.Cascade);   // Delete employees when dept deleted
}

// ─────────────────────────────────────────────────────────────────────────
// USAGE
// ─────────────────────────────────────────────────────────────────────────

// Get department with employees
var dept = await _context.Departments
    .Include(d => d.Employees)
    .FirstOrDefaultAsync(d => d.Id == 1);

// Add employee to department
var employee = new Employee
{
    Name = "John",
    DepartmentId = 1  // Assign via FK
};
_context.Employees.Add(employee);

// Or via navigation
var dept = await _context.Departments.FindAsync(1);
dept.Employees.Add(new Employee { Name = "Jane" });
await _context.SaveChangesAsync();
```

---

## 🔶 One-to-One

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// ONE-TO-ONE: Person <-> Address
// ═══════════════════════════════════════════════════════════════════════════

public class Person
{
    public int Id { get; set; }
    public string Name { get; set; }
    
    // Navigation property
    public Address Address { get; set; }
}

public class Address
{
    public int Id { get; set; }
    public string Street { get; set; }
    public string City { get; set; }
    
    // Foreign key
    public int PersonId { get; set; }
    
    // Navigation property
    public Person Person { get; set; }
}

// ─────────────────────────────────────────────────────────────────────────
// FLUENT API CONFIGURATION
// ─────────────────────────────────────────────────────────────────────────

protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Person>()
        .HasOne(p => p.Address)        // Person has one Address
        .WithOne(a => a.Person)        // Address has one Person
        .HasForeignKey<Address>(a => a.PersonId);  // FK on Address side
}

// ─────────────────────────────────────────────────────────────────────────
// USAGE
// ─────────────────────────────────────────────────────────────────────────

// Create person with address
var person = new Person
{
    Name = "John",
    Address = new Address
    {
        Street = "123 Main St",
        City = "New York"
    }
};
_context.Persons.Add(person);
await _context.SaveChangesAsync();

// Load with Include
var personWithAddr = await _context.Persons
    .Include(p => p.Address)
    .FirstOrDefaultAsync(p => p.Id == 1);
```

---

## 🔵 Many-to-Many

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// MANY-TO-MANY: Student <-> Course (EF Core 5+ automatic join table)
// ═══════════════════════════════════════════════════════════════════════════

public class Student
{
    public int Id { get; set; }
    public string Name { get; set; }
    
    // Navigation - collection of courses
    public ICollection<Course> Courses { get; set; }
}

public class Course
{
    public int Id { get; set; }
    public string Title { get; set; }
    
    // Navigation - collection of students
    public ICollection<Student> Students { get; set; }
}

// EF Core automatically creates join table: StudentCourse

// ─────────────────────────────────────────────────────────────────────────
// EXPLICIT JOIN TABLE (for additional properties)
// ─────────────────────────────────────────────────────────────────────────

public class Student
{
    public int Id { get; set; }
    public string Name { get; set; }
    public ICollection<Enrollment> Enrollments { get; set; }
}

public class Course
{
    public int Id { get; set; }
    public string Title { get; set; }
    public ICollection<Enrollment> Enrollments { get; set; }
}

// Join table with additional data
public class Enrollment
{
    public int StudentId { get; set; }
    public Student Student { get; set; }
    
    public int CourseId { get; set; }
    public Course Course { get; set; }
    
    // Additional properties
    public DateTime EnrollmentDate { get; set; }
    public string Grade { get; set; }
}

// Fluent API
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Enrollment>()
        .HasKey(e => new { e.StudentId, e.CourseId });  // Composite key
    
    modelBuilder.Entity<Enrollment>()
        .HasOne(e => e.Student)
        .WithMany(s => s.Enrollments)
        .HasForeignKey(e => e.StudentId);
    
    modelBuilder.Entity<Enrollment>()
        .HasOne(e => e.Course)
        .WithMany(c => c.Enrollments)
        .HasForeignKey(e => e.CourseId);
}

// Usage
var student = await _context.Students
    .Include(s => s.Enrollments)
        .ThenInclude(e => e.Course)
    .FirstOrDefaultAsync(s => s.Id == 1);
```

---

## 🟢 Data Annotations

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// DATA ANNOTATIONS - ATTRIBUTE-BASED CONFIGURATION
// ═══════════════════════════════════════════════════════════════════════════

using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

[Table("Employees")]  // Explicit table name
public class Employee
{
    [Key]  // Primary key (usually convention-based)
    public int Id { get; set; }
    
    [Required]  // NOT NULL
    [StringLength(100)]  // Max length
    public string Name { get; set; }
    
    [Required]
    [EmailAddress]  // Validation
    [Column("EmailAddress")]  // Column name
    public string Email { get; set; }
    
    [Range(0, 1000000)]  // Value range
    [Column(TypeName = "decimal(18,2)")]  // SQL type
    public decimal Salary { get; set; }
    
    [MaxLength(500)]
    public string Notes { get; set; }
    
    [NotMapped]  // Exclude from database
    public string FullName => $"{Name} ({Email})";
    
    // Foreign key
    [ForeignKey("Department")]
    public int DepartmentId { get; set; }
    public Department Department { get; set; }
    
    // Timestamps
    [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
    public DateTime CreatedAt { get; set; }
    
    [Timestamp]  // For concurrency
    public byte[] RowVersion { get; set; }
}

// ─────────────────────────────────────────────────────────────────────────
// INDEX AND UNIQUE CONSTRAINTS
// ─────────────────────────────────────────────────────────────────────────

[Index(nameof(Email), IsUnique = true)]  // Unique index
[Index(nameof(Name), nameof(DepartmentId))]  // Composite index
public class Employee { }
```

### Common Data Annotations

| Annotation | Purpose |
|------------|---------|
| `[Key]` | Primary key |
| `[Required]` | NOT NULL |
| `[StringLength(n)]` | Max string length |
| `[MaxLength(n)]` | Max length (any type) |
| `[Column("name")]` | Custom column name |
| `[Table("name")]` | Custom table name |
| `[ForeignKey("Nav")]` | Foreign key property |
| `[NotMapped]` | Exclude from DB |
| `[Index]` | Create index |
| `[Timestamp]` | Concurrency token |

---

## 🎤 Interview Questions

**Q1: How to configure one-to-many?**
> Use `HasOne().WithMany().HasForeignKey()` in Fluent API, or navigation properties with FK.

**Q2: Data Annotations vs Fluent API?**
> Annotations: simpler, on entity. Fluent API: more powerful, in OnModelCreating.

**Q3: What is navigation property?**
> Property that references related entity (single reference or collection).

**Q4: How does many-to-many work in EF Core 5+?**
> Automatic join table if just navigation collections. Explicit join entity for additional properties.

**Q5: What is `[NotMapped]`?**
> Excludes property from database mapping. Used for computed properties.

---

## 📝 Summary

| Relationship | Configuration |
|--------------|---------------|
| **One-to-Many** | `HasOne().WithMany().HasForeignKey()` |
| **One-to-One** | `HasOne().WithOne().HasForeignKey<T>()` |
| **Many-to-Many** | Collections on both sides (auto join table) |
| **Custom Join** | Explicit join entity with composite key |
