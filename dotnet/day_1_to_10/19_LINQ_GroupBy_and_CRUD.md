# 📊 LINQ GroupBy and CRUD Operations

## 📚 Table of Contents
- [GroupBy Deep Dive](#-groupby-deep-dive)
- [LINQ CRUD Operations](#-linq-crud-operations)
- [Practical Examples](#-practical-examples)
- [Interview Questions](#-interview-questions)

---

## 🎯 GroupBy Deep Dive

### Basic GroupBy

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// GROUPBY FUNDAMENTALS
// ═══════════════════════════════════════════════════════════════════════════

using System;
using System.Linq;
using System.Collections.Generic;

class GroupByDemo
{
    static void Main()
    {
        var employees = new List<Employee>
        {
            new Employee { Id = 1, Name = "Alice", Department = "IT", Salary = 60000 },
            new Employee { Id = 2, Name = "Bob", Department = "HR", Salary = 50000 },
            new Employee { Id = 3, Name = "Charlie", Department = "IT", Salary = 65000 },
            new Employee { Id = 4, Name = "Diana", Department = "HR", Salary = 55000 },
            new Employee { Id = 5, Name = "Eve", Department = "IT", Salary = 70000 }
        };
        
        // ─────────────────────────────────────────────────────────────────
        // SIMPLE GROUPBY
        // ─────────────────────────────────────────────────────────────────
        
        var byDepartment = employees.GroupBy(e => e.Department);
        // Line 1: Groups employees by Department
        // Returns: IEnumerable<IGrouping<string, Employee>>
        //          Key = Department name
        //          Elements = Employees in that department
        
        foreach (var group in byDepartment)
        {
            Console.WriteLine($"\n📁 {group.Key} Department:");
            // Line 2: group.Key = grouping key (IT or HR)
            
            foreach (var emp in group)
            {
                Console.WriteLine($"   • {emp.Name} - ${emp.Salary:N0}");
            }
        }
        /* Output:
        📁 IT Department:
           • Alice - $60,000
           • Charlie - $65,000
           • Eve - $70,000

        📁 HR Department:
           • Bob - $50,000
           • Diana - $55,000
        */
        
        // ─────────────────────────────────────────────────────────────────
        // GROUPBY WITH AGGREGATION
        // ─────────────────────────────────────────────────────────────────
        
        var deptStats = employees
            .GroupBy(e => e.Department)
            .Select(g => new
            {
                Department = g.Key,
                Count = g.Count(),
                TotalSalary = g.Sum(e => e.Salary),
                AvgSalary = g.Average(e => e.Salary),
                MaxSalary = g.Max(e => e.Salary),
                MinSalary = g.Min(e => e.Salary)
            });
        
        foreach (var dept in deptStats)
        {
            Console.WriteLine($"{dept.Department}: {dept.Count} employees, " +
                            $"Avg: ${dept.AvgSalary:N0}");
        }
        /* Output:
        IT: 3 employees, Avg: $65,000
        HR: 2 employees, Avg: $52,500
        */
        
        // ─────────────────────────────────────────────────────────────────
        // GROUPBY WITH ELEMENT SELECTOR
        // ─────────────────────────────────────────────────────────────────
        
        var namesByDept = employees.GroupBy(
            e => e.Department,     // Key selector
            e => e.Name            // Element selector (transform element)
        );
        // Line 3: Groups contain names only, not full Employee objects
        
        foreach (var group in namesByDept)
        {
            Console.WriteLine($"{group.Key}: {string.Join(", ", group)}");
        }
        // IT: Alice, Charlie, Eve
        // HR: Bob, Diana
        
        // ─────────────────────────────────────────────────────────────────
        // GROUPBY WITH RESULT SELECTOR
        // ─────────────────────────────────────────────────────────────────
        
        var summary = employees.GroupBy(
            e => e.Department,     // Key selector
            e => e,                // Element selector
            (key, emps) => new     // Result selector
            {
                Dept = key,
                Employees = emps.OrderBy(e => e.Name).ToList(),
                HeadCount = emps.Count()
            }
        );
        
        // ─────────────────────────────────────────────────────────────────
        // GROUPBY MULTIPLE KEYS
        // ─────────────────────────────────────────────────────────────────
        
        var products = new List<Product>
        {
            new Product { Category = "Electronics", Year = 2023, Sales = 1000 },
            new Product { Category = "Electronics", Year = 2024, Sales = 1500 },
            new Product { Category = "Clothing", Year = 2023, Sales = 800 },
            new Product { Category = "Clothing", Year = 2024, Sales = 900 }
        };
        
        var byCategoryYear = products.GroupBy(p => new { p.Category, p.Year });
        // Line 4: Anonymous type as composite key
        
        foreach (var group in byCategoryYear)
        {
            Console.WriteLine($"{group.Key.Category} - {group.Key.Year}: " +
                            $"${group.Sum(p => p.Sales)}");
        }
    }
}

class Employee
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Department { get; set; }
    public decimal Salary { get; set; }
}

class Product
{
    public string Category { get; set; }
    public int Year { get; set; }
    public decimal Sales { get; set; }
}
```

### Query Syntax for GroupBy

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// QUERY SYNTAX GROUPBY
// ═══════════════════════════════════════════════════════════════════════════

var employees = GetEmployees();

// Basic group
var grouped = from e in employees
              group e by e.Department;

// Group into variable for further processing
var withCount = from e in employees
                group e by e.Department into deptGroup
                select new
                {
                    Dept = deptGroup.Key,
                    Count = deptGroup.Count(),
                    Avg = deptGroup.Average(x => x.Salary)
                };

// Filter after grouping
var largeDepts = from e in employees
                 group e by e.Department into deptGroup
                 where deptGroup.Count() > 2
                 select deptGroup;

// Order after grouping
var orderedGroups = from e in employees
                    group e by e.Department into deptGroup
                    orderby deptGroup.Key
                    select deptGroup;
```

---

## 🔶 LINQ CRUD Operations

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// CRUD OPERATIONS WITH LINQ
// ═══════════════════════════════════════════════════════════════════════════

using System;
using System.Linq;
using System.Collections.Generic;

class LinqCrudDemo
{
    private static List<Employee> _employees = new List<Employee>
    {
        new Employee { Id = 1, Name = "Alice", Department = "IT", Salary = 60000 },
        new Employee { Id = 2, Name = "Bob", Department = "HR", Salary = 50000 },
        new Employee { Id = 3, Name = "Charlie", Department = "IT", Salary = 65000 }
    };
    
    static void Main()
    {
        // ═══════════════════════════════════════════════════════════════
        // READ OPERATIONS
        // ═══════════════════════════════════════════════════════════════
        
        // Get all
        var allEmployees = _employees.ToList();
        
        // Get by ID
        var emp = _employees.FirstOrDefault(e => e.Id == 1);
        // Returns Alice or null if not found
        
        // Get by ID (throws if not found)
        var empRequired = _employees.First(e => e.Id == 1);
        // Throws InvalidOperationException if not found
        
        // Single (exactly one match expected)
        var unique = _employees.SingleOrDefault(e => e.Id == 1);
        
        // Filter with Where
        var itEmployees = _employees.Where(e => e.Department == "IT");
        
        // Search by name (contains)
        var searchResults = _employees
            .Where(e => e.Name.Contains("li", StringComparison.OrdinalIgnoreCase));
        
        // Pagination
        int page = 1, pageSize = 10;
        var paged = _employees
            .Skip((page - 1) * pageSize)
            .Take(pageSize);
        
        // ═══════════════════════════════════════════════════════════════
        // CREATE OPERATION
        // ═══════════════════════════════════════════════════════════════
        
        void AddEmployee(Employee newEmp)
        {
            // Generate new ID
            newEmp.Id = _employees.Any() ? _employees.Max(e => e.Id) + 1 : 1;
            _employees.Add(newEmp);
        }
        
        AddEmployee(new Employee { Name = "Diana", Department = "HR", Salary = 55000 });
        
        // ═══════════════════════════════════════════════════════════════
        // UPDATE OPERATION
        // ═══════════════════════════════════════════════════════════════
        
        void UpdateEmployee(int id, string name, decimal salary)
        {
            var employee = _employees.FirstOrDefault(e => e.Id == id);
            if (employee != null)
            {
                employee.Name = name;
                employee.Salary = salary;
                // Object is reference type, so changes in list automatically
            }
        }
        
        UpdateEmployee(1, "Alicia", 65000);
        
        // Alternative: Find index and replace
        void ReplaceEmployee(Employee updated)
        {
            int index = _employees.FindIndex(e => e.Id == updated.Id);
            if (index >= 0)
            {
                _employees[index] = updated;
            }
        }
        
        // ═══════════════════════════════════════════════════════════════
        // DELETE OPERATION
        // ═══════════════════════════════════════════════════════════════
        
        void DeleteEmployee(int id)
        {
            var employee = _employees.FirstOrDefault(e => e.Id == id);
            if (employee != null)
            {
                _employees.Remove(employee);
            }
        }
        
        DeleteEmployee(2);  // Remove Bob
        
        // Alternative: RemoveAll
        int removedCount = _employees.RemoveAll(e => e.Department == "Temp");
        
        // ═══════════════════════════════════════════════════════════════
        // COMPLEX QUERIES
        // ═══════════════════════════════════════════════════════════════
        
        // Top N by salary
        var topEarners = _employees
            .OrderByDescending(e => e.Salary)
            .Take(3);
        
        // Distinct departments
        var departments = _employees.Select(e => e.Department).Distinct();
        
        // Check existence
        bool hasIT = _employees.Any(e => e.Department == "IT");
        bool allHighPaid = _employees.All(e => e.Salary > 40000);
        
        // Lookup (grouped dictionary)
        ILookup<string, Employee> byDept = _employees.ToLookup(e => e.Department);
        var itPeople = byDept["IT"];  // Get all IT employees
    }
}
```

---

## 🔵 Practical Examples

### Employee Report Generation

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// PRACTICAL: EMPLOYEE REPORTS
// ═══════════════════════════════════════════════════════════════════════════

class ReportGenerator
{
    public static void GenerateReports(List<Employee> employees)
    {
        // Department Summary Report
        var deptSummary = employees
            .GroupBy(e => e.Department)
            .Select(g => new
            {
                Department = g.Key,
                Headcount = g.Count(),
                TotalPayroll = g.Sum(e => e.Salary),
                AvgSalary = g.Average(e => e.Salary),
                HighestPaid = g.OrderByDescending(e => e.Salary).First().Name
            })
            .OrderByDescending(d => d.TotalPayroll);
        
        Console.WriteLine("=== DEPARTMENT SUMMARY ===");
        foreach (var dept in deptSummary)
        {
            Console.WriteLine($@"
{dept.Department}
  Employees: {dept.Headcount}
  Payroll:   ${dept.TotalPayroll:N0}
  Average:   ${dept.AvgSalary:N0}
  Top Paid:  {dept.HighestPaid}");
        }
        
        // Salary Distribution
        var salaryRanges = employees
            .GroupBy(e => e.Salary switch
            {
                < 50000 => "< $50K",
                < 70000 => "$50K - $70K",
                _ => "> $70K"
            })
            .Select(g => new { Range = g.Key, Count = g.Count() });
        
        Console.WriteLine("\n=== SALARY DISTRIBUTION ===");
        foreach (var range in salaryRanges)
        {
            Console.WriteLine($"{range.Range}: {range.Count} employees");
        }
    }
}
```

---

## 🎤 Interview Questions

**Q1: What does GroupBy return?**
> Returns `IEnumerable<IGrouping<TKey, TElement>>`. Each IGrouping has a Key property and contains elements for that key.

**Q2: How to aggregate after GroupBy?**
> Chain `.Select()` after GroupBy and use aggregate functions like `Count()`, `Sum()`, `Average()` on each group.

**Q3: GroupBy vs ToLookup?**
> GroupBy is deferred execution. ToLookup executes immediately and returns ILookup (like Dictionary but allows multiple values per key).

**Q4: How to group by multiple columns?**
> Use anonymous type: `.GroupBy(x => new { x.Col1, x.Col2 })`

---

## 📝 Summary

| Operation | LINQ Method |
|-----------|-------------|
| Group by key | `GroupBy(x => x.Key)` |
| Group with transform | `GroupBy(keySelector, elementSelector)` |
| Multiple keys | `GroupBy(x => new { x.A, x.B })` |
| Read by ID | `FirstOrDefault(x => x.Id == id)` |
| Filter | `Where(predicate)` |
| Update | Find + modify reference |
| Delete | `Remove()` or `RemoveAll()` |
