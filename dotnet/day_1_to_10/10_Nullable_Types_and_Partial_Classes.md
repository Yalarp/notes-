# ❓ Nullable Types and Partial Classes in C#

## 📚 Table of Contents
- [Nullable Types](#-nullable-types)
- [Null-Coalescing Operators](#-null-coalescing-operators)
- [Partial Classes](#-partial-classes)
- [Partial Methods](#-partial-methods)
- [Interview Questions](#-interview-questions)

---

## 🎯 Nullable Types

### Why Nullable Types?

Value types (int, double, bool) cannot be null by default. Nullable types allow value types to represent "no value" or "undefined".

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// NULLABLE VALUE TYPES
// ═══════════════════════════════════════════════════════════════════════════

using System;

class NullableDemo
{
    static void Main()
    {
        // Regular value type - cannot be null
        int regularInt = 10;
        // regularInt = null;  // ❌ Compile error!
        
        // ─────────────────────────────────────────────────────────────────
        // DECLARING NULLABLE TYPES
        // ─────────────────────────────────────────────────────────────────
        
        // Method 1: Using ? syntax (preferred)
        int? nullableInt = null;
        // Line 1: int? is shorthand for Nullable<int>
        //         Can hold int value OR null
        
        // Method 2: Using Nullable<T> generic
        Nullable<int> anotherNullable = null;
        // Line 2: Explicit generic syntax
        //         Equivalent to int?
        
        // Assign value
        nullableInt = 42;
        Console.WriteLine($"Value: {nullableInt}");  // Output: Value: 42
        
        nullableInt = null;
        Console.WriteLine($"Value: {nullableInt}");  // Output: Value: (nothing)
        
        // ─────────────────────────────────────────────────────────────────
        // CHECKING FOR NULL
        // ─────────────────────────────────────────────────────────────────
        
        int? age = GetAge();  // May return null
        
        // Method 1: HasValue property
        if (age.HasValue)
        // Line 3: HasValue is true if variable has a value
        {
            Console.WriteLine($"Age: {age.Value}");
            // Line 4: .Value extracts the underlying value
            //         Throws InvalidOperationException if null!
        }
        
        // Method 2: Compare to null
        if (age != null)
        {
            Console.WriteLine($"Age: {age}");  // Implicit .Value
        }
        
        // Method 3: Pattern matching (C# 7+)
        if (age is int actualAge)
        // Line 5: Pattern matching extracts value if not null
        {
            Console.WriteLine($"Age: {actualAge}");
        }
        
        // ─────────────────────────────────────────────────────────────────
        // GETTING VALUE SAFELY
        // ─────────────────────────────────────────────────────────────────
        
        // GetValueOrDefault - returns default if null
        int safeAge = age.GetValueOrDefault();
        // Line 6: Returns 0 (default for int) if null
        
        int safeAge2 = age.GetValueOrDefault(18);
        // Line 7: Returns 18 if null
        
        // ─────────────────────────────────────────────────────────────────
        // NULLABLE WITH DIFFERENT VALUE TYPES
        // ─────────────────────────────────────────────────────────────────
        
        bool? isActive = null;      // Can be true, false, or null
        double? salary = 50000.50;
        DateTime? birthDate = null;
        
        // Enum can be nullable too
        DayOfWeek? meetingDay = null;
        meetingDay = DayOfWeek.Monday;
    }
    
    static int? GetAge()
    {
        // Simulate database that might return null
        return null;
    }
}
```

---

## 🔷 Null-Coalescing Operators

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// NULL-COALESCING OPERATORS
// ═══════════════════════════════════════════════════════════════════════════

using System;

class NullCoalescingDemo
{
    static void Main()
    {
        // ─────────────────────────────────────────────────────────────────
        // ?? OPERATOR (Null-Coalescing)
        // ─────────────────────────────────────────────────────────────────
        
        int? nullableValue = null;
        
        // If nullableValue is null, use default value
        int result = nullableValue ?? 0;
        // Line 1: ?? returns left side if not null, else right side
        //         result = 0 (because nullableValue is null)
        
        nullableValue = 42;
        result = nullableValue ?? 0;
        // result = 42 (left side is not null)
        
        // Chaining multiple ??
        int? a = null;
        int? b = null;
        int? c = 100;
        int value = a ?? b ?? c ?? 0;
        // Line 2: Evaluates left to right, returns first non-null
        // value = 100
        
        // Works with reference types too
        string name = null;
        string displayName = name ?? "Anonymous";
        // displayName = "Anonymous"
        
        // ─────────────────────────────────────────────────────────────────
        // ??= OPERATOR (Null-Coalescing Assignment) - C# 8.0+
        // ─────────────────────────────────────────────────────────────────
        
        string text = null;
        text ??= "Default Text";
        // Line 3: Assigns right side ONLY if left is null
        // text = "Default Text"
        
        text ??= "Another Value";
        // text = "Default Text" (unchanged, because not null)
        
        // Equivalent to:
        // if (text == null) text = "Default Text";
        
        // ─────────────────────────────────────────────────────────────────
        // ?. OPERATOR (Null-Conditional / Safe Navigation)
        // ─────────────────────────────────────────────────────────────────
        
        Person person = null;
        
        // Without ?. - would throw NullReferenceException
        // string city = person.Address.City;  // ❌ Exception!
        
        // With ?. - safe navigation
        string city = person?.Address?.City;
        // Line 4: ?. returns null if left side is null
        //         Doesn't access Address if person is null
        //         Doesn't access City if Address is null
        // city = null (no exception)
        
        // Combining ?. with ??
        string safeCity = person?.Address?.City ?? "Unknown";
        // Line 5: Get city or "Unknown" if any part is null
        // safeCity = "Unknown"
        
        // ?. with methods
        int? length = person?.Name?.Length;
        // Returns null if person or Name is null
        
        // ?. with indexers
        int[] numbers = null;
        int? firstNum = numbers?[0];
        // Line 6: Returns null instead of exception
        
        // ─────────────────────────────────────────────────────────────────
        // ?. WITH EVENTS AND DELEGATES
        // ─────────────────────────────────────────────────────────────────
        
        Action callback = null;
        
        // Old way (verbose)
        if (callback != null)
        {
            callback();
        }
        
        // New way with ?.
        callback?.Invoke();
        // Line 7: Only invokes if not null
        
        // With events
        // OnDataReceived?.Invoke(this, args);
    }
}

class Person
{
    public string Name { get; set; }
    public Address Address { get; set; }
}

class Address
{
    public string City { get; set; }
}
```

### Null Operators Summary

| Operator | Name | Example | Description |
|----------|------|---------|-------------|
| `?` | Nullable type | `int?` | Allows null for value types |
| `??` | Null-coalescing | `a ?? b` | Returns a if not null, else b |
| `??=` | Null-coalescing assignment | `a ??= b` | Assigns b to a only if a is null |
| `?.` | Null-conditional | `a?.b` | Returns null if a is null |
| `?[]` | Null-conditional index | `a?[0]` | Returns null if a is null |

---

## 🔶 Partial Classes

Partial classes allow splitting a class definition across multiple files.

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// PARTIAL CLASSES - Split class across files
// ═══════════════════════════════════════════════════════════════════════════

// ─────────────────────────────────────────────────────────────────────
// FILE 1: Employee.Properties.cs
// ─────────────────────────────────────────────────────────────────────

namespace Company
{
    // Line 1: 'partial' keyword allows class split across files
    public partial class Employee
    {
        // Properties defined here
        public int Id { get; set; }
        public string Name { get; set; }
        public string Department { get; set; }
    }
}

// ─────────────────────────────────────────────────────────────────────
// FILE 2: Employee.Methods.cs
// ─────────────────────────────────────────────────────────────────────

namespace Company
{
    // Line 2: Same class name with 'partial' keyword
    //         Compiler merges all parts into single class
    public partial class Employee
    {
        // Methods defined here
        public void PrintInfo()
        {
            Console.WriteLine($"{Id}: {Name} - {Department}");
        }
        
        public decimal CalculateBonus(decimal baseSalary)
        {
            return baseSalary * 0.1m;
        }
    }
}

// ─────────────────────────────────────────────────────────────────────
// FILE 3: Employee.Validation.cs (auto-generated or separate concern)
// ─────────────────────────────────────────────────────────────────────

namespace Company
{
    public partial class Employee
    {
        public bool IsValid()
        {
            return !string.IsNullOrEmpty(Name) && Id > 0;
        }
    }
}

// ─────────────────────────────────────────────────────────────────────
// USAGE - All parts work as single class
// ─────────────────────────────────────────────────────────────────────

class Program
{
    static void Main()
    {
        // Line 3: Use as single class - all members accessible
        Employee emp = new Employee
        {
            Id = 1,
            Name = "John",
            Department = "IT"
        };
        
        emp.PrintInfo();           // From Employee.Methods.cs
        bool valid = emp.IsValid(); // From Employee.Validation.cs
    }
}
```

### Why Use Partial Classes?

```
┌─────────────────────────────────────────────────────────────────────┐
│                   USE CASES FOR PARTIAL CLASSES                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  1. CODE GENERATORS                                                 │
│     ├── Tool generates: Employee.generated.cs (auto-generated)     │
│     └── You write:      Employee.custom.cs (your custom code)      │
│     Benefit: Tool can regenerate without losing your changes       │
│                                                                     │
│  2. LARGE CLASSES                                                   │
│     ├── MyForm.Designer.cs  (Visual Studio designer)               │
│     └── MyForm.cs           (your event handlers)                  │
│                                                                     │
│  3. SEPARATION OF CONCERNS                                          │
│     ├── Customer.Data.cs        (data properties)                  │
│     ├── Customer.Validation.cs  (validation logic)                 │
│     └── Customer.Serialization.cs (serialization helpers)          │
│                                                                     │
│  4. TEAM COLLABORATION                                              │
│     Multiple developers work on different parts of same class      │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Partial Class Rules

```csharp
// All parts must:
// 1. Use 'partial' keyword
// 2. Have same accessibility (public, internal, etc.)
// 3. Be in same namespace
// 4. Be in same assembly

// ✅ VALID: Different files, same namespace
// File1.cs
public partial class Data { public int X { get; set; } }
// File2.cs
public partial class Data { public int Y { get; set; } }

// ✅ Can inherit in one part, applies to all
public partial class Data : IDisposable { }  // File1
public partial class Data { public void Dispose() { } }  // File2

// ✅ Can have attributes - they're merged
[Serializable]
public partial class Data { }  // File1
[Description("Data class")]
public partial class Data { }  // File2
// Result: both attributes apply
```

---

## 🔷 Partial Methods

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// PARTIAL METHODS - Optional method implementations
// ═══════════════════════════════════════════════════════════════════════════

// ─────────────────────────────────────────────────────────────────────
// FILE 1: Entity.generated.cs (Generated by tool)
// ─────────────────────────────────────────────────────────────────────

public partial class Entity
{
    private string _name;
    
    public string Name
    {
        get => _name;
        set
        {
            OnNameChanging(value);  // Call partial method
            _name = value;
            OnNameChanged();        // Call partial method
        }
    }
    
    // Line 1: Partial method declaration (no body)
    //         Must be private, return void (in older C#)
    //         If no implementation exists, calls are removed by compiler
    partial void OnNameChanging(string newValue);
    partial void OnNameChanged();
}

// ─────────────────────────────────────────────────────────────────────
// FILE 2: Entity.custom.cs (Your custom code - OPTIONAL)
// ─────────────────────────────────────────────────────────────────────

public partial class Entity
{
    // Line 2: Partial method implementation
    //         OPTIONAL - if not provided, compiler removes call sites
    partial void OnNameChanging(string newValue)
    {
        Console.WriteLine($"Name changing to: {newValue}");
    }
    
    partial void OnNameChanged()
    {
        Console.WriteLine("Name has changed!");
    }
}

// ─────────────────────────────────────────────────────────────────────
// C# 9+ EXTENDED PARTIAL METHODS
// ─────────────────────────────────────────────────────────────────────

public partial class ModernEntity
{
    // Line 3: C# 9+ allows return types, out parameters, accessibility
    //         BUT implementation is now REQUIRED
    public partial string GetDisplayName();
    public partial bool TryParse(string input, out int result);
}

public partial class ModernEntity
{
    public partial string GetDisplayName() => "Entity Name";
    
    public partial bool TryParse(string input, out int result)
    {
        return int.TryParse(input, out result);
    }
}
```

---

## 🎤 Interview Questions

**Q1: What is a nullable type?**
> A nullable type allows value types (int, bool, etc.) to represent null. Declared using `?` syntax: `int? x = null;`

**Q2: What's the difference between `??` and `?.` operators?**
> `??` (null-coalescing) provides default value if null: `x ?? 0`. `?.` (null-conditional) safely accesses members: `obj?.Property` returns null instead of exception.

**Q3: What is a partial class?**
> A class split across multiple files using `partial` keyword. All parts are merged by compiler into single class.

**Q4: When would you use partial classes?**
> Code generation (tool generates one file, you customize another), large classes, separation of concerns, designer files (WinForms/WPF).

**Q5: What happens if partial method has no implementation?**
> The compiler removes all call sites - no runtime overhead.

**Q6: How is `int?` different from `int`?**
> `int?` (Nullable<int>) can hold int values OR null. `int` can only hold integer values, not null.

---

## 📝 Summary

| Concept | Syntax | Purpose |
|---------|--------|---------|
| **Nullable** | `int?` | Allow null for value types |
| **HasValue** | `x.HasValue` | Check if has value |
| **Value** | `x.Value` | Extract underlying value |
| **??** | `a ?? b` | Default if null |
| **??=** | `a ??= b` | Assign if null |
| **?.** | `a?.b` | Safe member access |
| **partial class** | `partial class X` | Split class across files |
| **partial method** | `partial void M()` | Optional method hook |
