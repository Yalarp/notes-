# 🔗 Collection Interfaces in C# - Deep Dive

## 📚 Table of Contents
- [Overview](#-overview)
- [Interface Hierarchy](#-interface-hierarchy)
- [IEnumerable and IEnumerator](#-ienumerable-and-ienumerator)
- [ICollection<T>](#-icollectiont)
- [IList<T>](#-ilistt)
- [IQueryable<T>](#-iqueryablet)
- [Interview Questions](#-interview-questions)

---

## 🎯 Overview

Collection interfaces define contracts for different levels of collection functionality.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    INTERFACE HIERARCHY                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   IEnumerable<T>          - Iteration only (foreach)               │
│        │                                                            │
│        ▼                                                            │
│   ICollection<T>          - Count, Add, Remove, Contains           │
│        │                                                            │
│        ▼                                                            │
│   IList<T>                - Index access, Insert, RemoveAt         │
│        │                                                            │
│        ▼                                                            │
│   List<T>                 - Concrete implementation                │
│                                                                     │
│   IQueryable<T>           - Deferred execution for databases       │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔷 IEnumerable and IEnumerator

### IEnumerable<T> - Iteration Support

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// IENUMERABLE<T> - ENABLES foreach ITERATION
// ═══════════════════════════════════════════════════════════════════════════

using System;
using System.Collections;
using System.Collections.Generic;

// Line 1: IEnumerable<T> has ONE method: GetEnumerator()
public interface IEnumerable<T> : IEnumerable
{
    IEnumerator<T> GetEnumerator();
}

// Line 2: IEnumerator<T> provides iteration mechanism
public interface IEnumerator<T> : IEnumerator, IDisposable
{
    T Current { get; }      // Current element
    bool MoveNext();        // Move to next, return false if end
    void Reset();           // Reset to before first element
}

// ═══════════════════════════════════════════════════════════════════════════
// HOW foreach WORKS INTERNALLY
// ═══════════════════════════════════════════════════════════════════════════

class ForEachDemo
{
    static void Main()
    {
        List<string> names = new List<string> { "Alice", "Bob", "Charlie" };
        
        // This foreach loop:
        foreach (string name in names)
        {
            Console.WriteLine(name);
        }
        
        // Is compiled to:
        IEnumerator<string> enumerator = names.GetEnumerator();
        try
        {
            while (enumerator.MoveNext())
            // Line 3: MoveNext advances to next element
            //         Returns true if element exists
            {
                string name = enumerator.Current;
                // Line 4: Current returns current element
                Console.WriteLine(name);
            }
        }
        finally
        {
            enumerator.Dispose();
            // Line 5: Dispose cleans up resources
        }
    }
}
```

### Implementing Custom IEnumerable

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// CUSTOM IENUMERABLE IMPLEMENTATION
// ═══════════════════════════════════════════════════════════════════════════

public class NumberRange : IEnumerable<int>
{
    private readonly int _start;
    private readonly int _end;
    
    public NumberRange(int start, int end)
    {
        _start = start;
        _end = end;
    }
    
    // Line 1: Required by IEnumerable<T>
    public IEnumerator<int> GetEnumerator()
    {
        // Line 2: yield return creates iterator automatically
        for (int i = _start; i <= _end; i++)
        {
            yield return i;
            // yield creates state machine for iteration
            // Pauses here, resumes on next MoveNext()
        }
    }
    
    // Line 3: Required by non-generic IEnumerable
    IEnumerator IEnumerable.GetEnumerator()
    {
        return GetEnumerator();
    }
}

class CustomEnumerableDemo
{
    static void Main()
    {
        NumberRange range = new NumberRange(1, 5);
        
        foreach (int num in range)
        {
            Console.WriteLine(num);  // 1, 2, 3, 4, 5
        }
        
        // Works with LINQ too!
        var doubled = range.Select(x => x * 2);  // 2, 4, 6, 8, 10
    }
}
```

### yield return - Iterator Pattern

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// YIELD RETURN - LAZY EVALUATION
// ═══════════════════════════════════════════════════════════════════════════

class YieldDemo
{
    // Line 1: Method returns IEnumerable using yield
    static IEnumerable<int> GetNumbers()
    {
        Console.WriteLine("Starting...");
        yield return 1;  // Pause, return 1
        Console.WriteLine("After 1");
        yield return 2;  // Pause, return 2
        Console.WriteLine("After 2");
        yield return 3;  // Pause, return 3
        Console.WriteLine("Done");
    }
    
    static void Main()
    {
        // Line 2: Method NOT executed yet!
        IEnumerable<int> numbers = GetNumbers();
        Console.WriteLine("Method called");
        
        // Line 3: Execution starts on first iteration
        foreach (int n in numbers)
        {
            Console.WriteLine($"Got: {n}");
        }
        
        /* Output:
        Method called          <- IEnumerable created, not executed
        Starting...            <- First MoveNext()
        Got: 1
        After 1                <- Second MoveNext()
        Got: 2
        After 2                <- Third MoveNext()
        Got: 3
        Done                   <- Iteration ends
        */
    }
    
    // Infinite sequence example
    static IEnumerable<int> InfiniteNumbers()
    {
        int i = 0;
        while (true)
        {
            yield return i++;
            // Line 4: Only generates values when requested
            //         No infinite loop because of lazy evaluation
        }
    }
    
    // Usage:
    // var first10 = InfiniteNumbers().Take(10);  // Only generates 10 numbers
}
```

---

## 🔶 ICollection<T>

Extends IEnumerable with Count, Add, Remove, Contains.

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// ICOLLECTION<T> INTERFACE
// ═══════════════════════════════════════════════════════════════════════════

public interface ICollection<T> : IEnumerable<T>
{
    int Count { get; }                    // Number of elements
    bool IsReadOnly { get; }              // Can modify?
    void Add(T item);                     // Add element
    void Clear();                         // Remove all
    bool Contains(T item);                // Check existence
    void CopyTo(T[] array, int index);    // Copy to array
    bool Remove(T item);                  // Remove element
}

// When to use ICollection<T>:
// - Need to know collection size
// - Need to add/remove elements
// - Don't need index-based access

void ProcessCollection(ICollection<string> items)
{
    Console.WriteLine($"Count: {items.Count}");
    
    if (!items.IsReadOnly)
    {
        items.Add("New Item");
    }
    
    if (items.Contains("Target"))
    {
        items.Remove("Target");
    }
}
```

---

## 🔵 IList<T>

Adds index-based access to ICollection.

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// ILIST<T> INTERFACE
// ═══════════════════════════════════════════════════════════════════════════

public interface IList<T> : ICollection<T>
{
    T this[int index] { get; set; }    // Indexer - access by position
    int IndexOf(T item);                // Find index of element
    void Insert(int index, T item);     // Insert at position
    void RemoveAt(int index);           // Remove at position
}

class ListInterfaceDemo
{
    // Method accepts any IList<T> implementation
    static void ProcessList(IList<string> list)
    {
        // Index access
        string first = list[0];
        list[0] = "Modified";
        
        // Find index
        int index = list.IndexOf("Target");
        
        // Insert at position
        list.Insert(1, "Inserted");
        
        // Remove at position
        list.RemoveAt(list.Count - 1);
    }
    
    static void Main()
    {
        // All these implement IList<T>
        List<string> list = new List<string> { "A", "B", "C" };
        string[] array = { "X", "Y", "Z" };
        
        ProcessList(list);   // Works!
        ProcessList(array);  // Arrays implement IList<T> too!
    }
}
```

---

## 🟢 IQueryable<T>

For deferred query execution, especially with databases.

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// IQueryable vs IEnumerable
// ═══════════════════════════════════════════════════════════════════════════

// IEnumerable<T>: Executes in memory
// IQueryable<T>: Translates to SQL, executes on database

class QueryableDemo
{
    static void Main()
    {
        // Assume: dbContext.Employees is IQueryable<Employee>
        
        // ─────────────────────────────────────────────────────────────────
        // IQueryable - Query built as expression tree
        // ─────────────────────────────────────────────────────────────────
        
        // This is NOT executed yet - just builds expression
        IQueryable<Employee> query = dbContext.Employees
            .Where(e => e.Department == "IT")
            .OrderBy(e => e.Name);
        
        // SQL generated: SELECT * FROM Employees 
        //                WHERE Department = 'IT' 
        //                ORDER BY Name
        
        // Executed when iterated
        foreach (var emp in query) { ... }
        // Or: query.ToList()
        
        // ─────────────────────────────────────────────────────────────────
        // IEnumerable - Loads all, filters in memory
        // ─────────────────────────────────────────────────────────────────
        
        IEnumerable<Employee> inMemory = dbContext.Employees
            .AsEnumerable()  // Forces IEnumerable
            .Where(e => e.Department == "IT");
        
        // SQL: SELECT * FROM Employees (loads ALL)
        // Then filters in C# memory - SLOWER!
    }
}
```

### IQueryable vs IEnumerable Comparison

| Aspect | IEnumerable<T> | IQueryable<T> |
|--------|----------------|---------------|
| **Execution** | In-memory | Deferred to provider |
| **Best for** | Collections | Databases |
| **LINQ** | LINQ to Objects | LINQ to SQL/EF |
| **Filtering** | After loading all | At data source |
| **Namespace** | System.Collections | System.Linq |

---

## 🔶 Choosing the Right Interface

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// PARAMETER TYPE GUIDELINES
// ═══════════════════════════════════════════════════════════════════════════

class InterfaceChoiceDemo
{
    // Use IEnumerable<T> for read-only iteration
    void ProcessItems(IEnumerable<string> items)
    {
        foreach (var item in items) { /* read only */ }
    }
    
    // Use ICollection<T> if you need Count or add/remove
    void ManageItems(ICollection<string> items)
    {
        if (items.Count > 0) { items.Clear(); }
    }
    
    // Use IList<T> if you need index access
    void IndexedAccess(IList<string> items)
    {
        string first = items[0];
        items.Insert(0, "New First");
    }
    
    // Use most GENERAL type possible
    // ✅ GOOD: Accepts List, Array, HashSet, etc.
    void Good(IEnumerable<string> items) { }
    
    // ❌ BAD: Only accepts List<string>
    void Bad(List<string> items) { }
}
```

---

## 🎤 Interview Questions

**Q1: What is IEnumerable<T>?**
> Interface that enables iteration via foreach. Has single method `GetEnumerator()` that returns an `IEnumerator<T>`.

**Q2: What does yield return do?**
> Creates a state machine for lazy iteration. Execution pauses at yield, resumes on next MoveNext().

**Q3: Difference between IEnumerable and IQueryable?**
> IEnumerable executes in memory (LINQ to Objects). IQueryable builds expression tree for deferred execution (LINQ to SQL/EF).

**Q4: When to use IList vs ICollection vs IEnumerable?**
> IEnumerable for read-only iteration, ICollection if need Count/Add/Remove, IList if need index access.

**Q5: Why prefer IEnumerable<T> as parameter?**
> Most flexible - accepts any collection type. Follows "accept the most general type, return the most specific" principle.

---

## 📝 Summary

| Interface | Key Features | When to Use |
|-----------|--------------|-------------|
| `IEnumerable<T>` | foreach, yield | Read-only iteration |
| `ICollection<T>` | Count, Add, Remove | Need modification |
| `IList<T>` | Indexer, Insert, RemoveAt | Need index access |
| `IQueryable<T>` | Expression trees | Database queries |
