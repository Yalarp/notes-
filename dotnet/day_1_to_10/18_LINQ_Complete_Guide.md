# 🔍 LINQ Complete Guide

## 📚 Table of Contents
- [Overview](#-overview)
- [Query Syntax vs Method Syntax](#-query-syntax-vs-method-syntax)
- [Filtering and Projection](#-filtering-and-projection)
- [Sorting and Grouping](#-sorting-and-grouping)
- [Joins](#-joins)
- [Aggregation](#-aggregation)
- [Deferred Execution](#-deferred-execution)
- [Interview Questions](#-interview-questions)

---

## 🎯 Overview

**LINQ (Language Integrated Query)** provides a unified way to query data from various sources.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    LINQ DATA SOURCES                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   LINQ Query ──────────────▶ Collections (LINQ to Objects)         │
│                |─────────────▶ Databases (LINQ to SQL/EF)          │
│                |─────────────▶ XML (LINQ to XML)                   │
│                └─────────────▶ Any IEnumerable<T>                  │
│                                                                     │
│   Same query syntax works for all data sources!                    │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔷 Query Syntax vs Method Syntax

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// TWO WAYS TO WRITE LINQ
// ═══════════════════════════════════════════════════════════════════════════

using System;
using System.Linq;
using System.Collections.Generic;

class LinqSyntaxDemo
{
    static void Main()
    {
        List<int> numbers = new List<int> { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
        
        // ─────────────────────────────────────────────────────────────────
        // QUERY SYNTAX (SQL-like)
        // ─────────────────────────────────────────────────────────────────
        
        var evenQuery = from n in numbers
                        where n % 2 == 0
                        select n;
        // Line 1: from = data source
        // Line 2: where = filter condition
        // Line 3: select = projection (what to return)
        
        // ─────────────────────────────────────────────────────────────────
        // METHOD SYNTAX (Extension methods)
        // ─────────────────────────────────────────────────────────────────
        
        var evenMethod = numbers.Where(n => n % 2 == 0);
        // Line 4: Where is extension method on IEnumerable<T>
        //         Lambda n => n % 2 == 0 is the predicate
        
        // Both produce same result: 2, 4, 6, 8, 10
        
        // ─────────────────────────────────────────────────────────────────
        // COMPLEX QUERY COMPARISON
        // ─────────────────────────────────────────────────────────────────
        
        List<Person> people = GetPeople();
        
        // Query syntax
        var adultNamesQuery = from p in people
                              where p.Age >= 18
                              orderby p.Name
                              select p.Name;
        
        // Method syntax (equivalent)
        var adultNamesMethod = people
            .Where(p => p.Age >= 18)
            .OrderBy(p => p.Name)
            .Select(p => p.Name);
    }
    
    static List<Person> GetPeople() => new List<Person>
    {
        new Person { Name = "Alice", Age = 25 },
        new Person { Name = "Bob", Age = 17 },
        new Person { Name = "Charlie", Age = 30 }
    };
}

class Person 
{ 
    public string Name { get; set; } 
    public int Age { get; set; } 
    public string Department { get; set; }
}
```

---

## 🔶 Filtering and Projection

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// WHERE, SELECT, SELECTMANY
// ═══════════════════════════════════════════════════════════════════════════

class FilterProjectDemo
{
    static void Main()
    {
        var people = new List<Person>
        {
            new Person { Name = "Alice", Age = 25, Department = "IT" },
            new Person { Name = "Bob", Age = 30, Department = "HR" },
            new Person { Name = "Charlie", Age = 35, Department = "IT" }
        };
        
        // ─────────────────────────────────────────────────────────────────
        // WHERE - Filtering
        // ─────────────────────────────────────────────────────────────────
        
        var itPeople = people.Where(p => p.Department == "IT");
        // Returns: Alice, Charlie
        
        // Multiple conditions
        var seniorIT = people.Where(p => p.Department == "IT" && p.Age > 28);
        // Returns: Charlie
        
        // With index
        var indexed = people.Where((p, index) => index % 2 == 0);
        // Returns every other person (index 0, 2, ...)
        
        // ─────────────────────────────────────────────────────────────────
        // SELECT - Projection (Transform)
        // ─────────────────────────────────────────────────────────────────
        
        // Select single property
        IEnumerable<string> names = people.Select(p => p.Name);
        // Returns: "Alice", "Bob", "Charlie"
        
        // Transform to new type
        var summaries = people.Select(p => new 
        {
            FullInfo = $"{p.Name} ({p.Age})",
            IsAdult = p.Age >= 18
        });
        // Returns anonymous type with FullInfo and IsAdult
        
        // With index
        var numbered = people.Select((p, i) => $"{i + 1}. {p.Name}");
        // Returns: "1. Alice", "2. Bob", "3. Charlie"
        
        // ─────────────────────────────────────────────────────────────────
        // SELECTMANY - Flatten nested collections
        // ─────────────────────────────────────────────────────────────────
        
        var departments = new List<Department>
        {
            new Department 
            { 
                Name = "IT", 
                Employees = new List<string> { "Alice", "Charlie" } 
            },
            new Department 
            { 
                Name = "HR", 
                Employees = new List<string> { "Bob" } 
            }
        };
        
        // Select returns: List<List<string>>
        var nested = departments.Select(d => d.Employees);
        
        // SelectMany flattens: List<string>
        var allEmployees = departments.SelectMany(d => d.Employees);
        // Returns: "Alice", "Charlie", "Bob" (flattened)
    }
}

class Department
{
    public string Name { get; set; }
    public List<string> Employees { get; set; }
}
```

---

## 🔵 Sorting and Grouping

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// ORDERBY, GROUPBY
// ═══════════════════════════════════════════════════════════════════════════

class SortGroupDemo
{
    static void Main()
    {
        var people = new List<Person>
        {
            new Person { Name = "Charlie", Age = 30, Department = "IT" },
            new Person { Name = "Alice", Age = 25, Department = "IT" },
            new Person { Name = "Bob", Age = 30, Department = "HR" },
            new Person { Name = "Diana", Age = 25, Department = "HR" }
        };
        
        // ─────────────────────────────────────────────────────────────────
        // ORDERING
        // ─────────────────────────────────────────────────────────────────
        
        // Ascending
        var byName = people.OrderBy(p => p.Name);
        // Alice, Bob, Charlie, Diana
        
        // Descending
        var byAgeDesc = people.OrderByDescending(p => p.Age);
        // Charlie(30), Bob(30), Alice(25), Diana(25)
        
        // Multiple sort keys
        var multiSort = people
            .OrderBy(p => p.Age)         // Primary: age ascending
            .ThenByDescending(p => p.Name); // Secondary: name descending
        // Diana(25), Alice(25), Charlie(30), Bob(30)
        
        // ─────────────────────────────────────────────────────────────────
        // GROUPING
        // ─────────────────────────────────────────────────────────────────
        
        // Group by single key
        var byDept = people.GroupBy(p => p.Department);
        // Returns IEnumerable<IGrouping<string, Person>>
        
        foreach (var group in byDept)
        {
            Console.WriteLine($"Department: {group.Key}");
            // group.Key = grouping key (IT or HR)
            
            foreach (var person in group)
            {
                Console.WriteLine($"  - {person.Name}");
            }
        }
        /* Output:
        Department: IT
          - Charlie
          - Alice
        Department: HR
          - Bob
          - Diana
        */
        
        // Group with projection
        var deptSummary = people
            .GroupBy(p => p.Department)
            .Select(g => new
            {
                Dept = g.Key,
                Count = g.Count(),
                AvgAge = g.Average(p => p.Age),
                Names = g.Select(p => p.Name).ToList()
            });
        
        // Group by multiple keys
        var byDeptAndAge = people.GroupBy(p => new { p.Department, p.Age });
    }
}
```

---

## 🟢 Joins

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// JOIN, GROUPJOIN
// ═══════════════════════════════════════════════════════════════════════════

class JoinDemo
{
    static void Main()
    {
        var employees = new List<Employee>
        {
            new Employee { Id = 1, Name = "Alice", DeptId = 100 },
            new Employee { Id = 2, Name = "Bob", DeptId = 200 },
            new Employee { Id = 3, Name = "Charlie", DeptId = 100 }
        };
        
        var departments = new List<Dept>
        {
            new Dept { Id = 100, Name = "IT" },
            new Dept { Id = 200, Name = "HR" },
            new Dept { Id = 300, Name = "Finance" }  // No employees
        };
        
        // ─────────────────────────────────────────────────────────────────
        // INNER JOIN
        // ─────────────────────────────────────────────────────────────────
        
        // Method syntax
        var innerJoin = employees.Join(
            departments,               // Inner collection
            e => e.DeptId,            // Outer key selector
            d => d.Id,                // Inner key selector
            (e, d) => new             // Result selector
            {
                Employee = e.Name,
                Department = d.Name
            });
        
        // Query syntax
        var innerJoinQuery = 
            from e in employees
            join d in departments on e.DeptId equals d.Id
            select new { Employee = e.Name, Department = d.Name };
        
        /* Result:
           { Alice, IT }
           { Bob, HR }
           { Charlie, IT }
        */
        
        // ─────────────────────────────────────────────────────────────────
        // LEFT OUTER JOIN (using GroupJoin + SelectMany)
        // ─────────────────────────────────────────────────────────────────
        
        var leftJoin = departments.GroupJoin(
            employees,
            d => d.Id,
            e => e.DeptId,
            (dept, emps) => new { Dept = dept, Employees = emps })
            .SelectMany(
                x => x.Employees.DefaultIfEmpty(),
                (x, emp) => new
                {
                    Department = x.Dept.Name,
                    Employee = emp?.Name ?? "No Employee"
                });
        
        /* Result:
           { IT, Alice }
           { IT, Charlie }
           { HR, Bob }
           { Finance, No Employee }
        */
    }
}

class Employee { public int Id; public string Name; public int DeptId; }
class Dept { public int Id; public string Name; }
```

---

## 🟣 Aggregation

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// AGGREGATION METHODS
// ═══════════════════════════════════════════════════════════════════════════

class AggregationDemo
{
    static void Main()
    {
        var numbers = new List<int> { 5, 3, 9, 1, 7, 2, 8 };
        var people = new List<Person>
        {
            new Person { Name = "Alice", Age = 25 },
            new Person { Name = "Bob", Age = 30 },
            new Person { Name = "Charlie", Age = 35 }
        };
        
        // ─────────────────────────────────────────────────────────────────
        // COUNT
        // ─────────────────────────────────────────────────────────────────
        
        int total = numbers.Count();               // 7
        int evenCount = numbers.Count(n => n % 2 == 0);  // 3
        
        // ─────────────────────────────────────────────────────────────────
        // SUM, AVERAGE
        // ─────────────────────────────────────────────────────────────────
        
        int sum = numbers.Sum();                   // 35
        double avg = numbers.Average();            // 5.0
        
        // With selector
        int totalAge = people.Sum(p => p.Age);     // 90
        double avgAge = people.Average(p => p.Age); // 30.0
        
        // ─────────────────────────────────────────────────────────────────
        // MIN, MAX
        // ─────────────────────────────────────────────────────────────────
        
        int min = numbers.Min();                   // 1
        int max = numbers.Max();                   // 9
        
        int youngest = people.Min(p => p.Age);     // 25
        int oldest = people.Max(p => p.Age);       // 35
        
        // MinBy, MaxBy (C# 10+) - returns element, not value
        Person youngestPerson = people.MinBy(p => p.Age);  // Alice
        
        // ─────────────────────────────────────────────────────────────────
        // AGGREGATE (Custom reduction)
        // ─────────────────────────────────────────────────────────────────
        
        // Sum using aggregate
        int sumAggregate = numbers.Aggregate((acc, n) => acc + n);
        // acc starts as first element
        // Iteration: 5+3=8, 8+9=17, 17+1=18, 18+7=25, 25+2=27, 27+8=35
        
        // With seed value
        string csv = numbers.Aggregate(
            "",  // Seed (starting value)
            (acc, n) => acc + (acc.Length > 0 ? "," : "") + n);
        // Result: "5,3,9,1,7,2,8"
        
        // ─────────────────────────────────────────────────────────────────
        // ALL, ANY
        // ─────────────────────────────────────────────────────────────────
        
        bool allPositive = numbers.All(n => n > 0);    // true
        bool anyEven = numbers.Any(n => n % 2 == 0);   // true
        bool anyAbove10 = numbers.Any(n => n > 10);    // false
        
        bool hasAnyElements = numbers.Any();  // Check if not empty
        
        // ─────────────────────────────────────────────────────────────────
        // FIRST, LAST, SINGLE
        // ─────────────────────────────────────────────────────────────────
        
        int first = numbers.First();           // 5
        int last = numbers.Last();             // 8
        
        int firstEven = numbers.First(n => n % 2 == 0);  // 2
        
        // Safe versions (no exception if not found)
        int firstOrDef = numbers.FirstOrDefault(n => n > 100);  // 0
        
        // Single - must be exactly one match
        var onlyAlice = people.Single(p => p.Name == "Alice");
        // Throws if 0 or 2+ matches
    }
}
```

---

## 🔴 Deferred Execution

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// DEFERRED VS IMMEDIATE EXECUTION
// ═══════════════════════════════════════════════════════════════════════════

class DeferredDemo
{
    static void Main()
    {
        var numbers = new List<int> { 1, 2, 3 };
        
        // ─────────────────────────────────────────────────────────────────
        // DEFERRED (Query NOT executed yet)
        // ─────────────────────────────────────────────────────────────────
        
        var query = numbers.Where(n => n > 1);
        // Line 1: Query is DEFINED, not EXECUTED
        
        numbers.Add(4);  // Modify source
        
        foreach (var n in query)
        {
            Console.WriteLine(n);
        }
        // Output: 2, 3, 4  (includes 4 added after query!)
        // Line 2: Query executes HERE on each iteration
        
        // ─────────────────────────────────────────────────────────────────
        // IMMEDIATE (Execute and cache results)
        // ─────────────────────────────────────────────────────────────────
        
        var immediate = numbers.Where(n => n > 1).ToList();
        // Line 3: ToList() executes query immediately
        //         Results cached in List
        
        numbers.Add(5);  // Won't affect 'immediate'
        
        // Methods that force immediate execution:
        // ToList(), ToArray(), ToDictionary()
        // Count(), Sum(), Max(), Min(), Average()
        // First(), Last(), Single()
        // Any(), All()
        
        // ─────────────────────────────────────────────────────────────────
        // MULTIPLE ENUMERATION WARNING
        // ─────────────────────────────────────────────────────────────────
        
        var expensive = GetData().Where(x => x > 0);
        // If GetData() hits database:
        
        int count = expensive.Count();    // Database query #1
        int sum = expensive.Sum();        // Database query #2 (executes again!)
        
        // Better: materialize once
        var cached = expensive.ToList();
        int count2 = cached.Count();      // Uses cached list
        int sum2 = cached.Sum();          // Uses cached list
    }
    
    static IEnumerable<int> GetData() { yield return 1; }
}
```

---

## 🎤 Interview Questions

**Q1: What is LINQ?**
> Language Integrated Query - unified query syntax for collections, databases, XML, etc.

**Q2: Query syntax vs Method syntax?**
> Query uses SQL-like keywords (from, where, select). Method uses extension methods with lambdas. Both compile to same code.

**Q3: What is deferred execution?**
> Query execution is delayed until results are actually needed (enumeration). Use ToList() for immediate execution.

**Q4: Difference between Select and SelectMany?**
> Select transforms each element. SelectMany flattens nested collections into single sequence.

**Q5: First vs Single?**
> First returns first match (exception if empty). Single returns only element (exception if 0 or 2+ matches).

---

## 📝 Summary

| Method | Purpose | Returns |
|--------|---------|---------|
| `Where` | Filter | IEnumerable<T> |
| `Select` | Transform | IEnumerable<TResult> |
| `OrderBy` | Sort ascending | IOrderedEnumerable |
| `GroupBy` | Group elements | IEnumerable<IGrouping> |
| `Join` | Combine collections | IEnumerable<TResult> |
| `First/Last` | Get element | T |
| `Count/Sum/Avg` | Aggregate | number |
| `ToList()` | Immediate execution | List<T> |
