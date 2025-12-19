# ✨ C# 7, 8, 9, 10 New Features

## 📚 Table of Contents
- [C# 7 Features](#-c-7-features)
- [C# 8 Features](#-c-8-features)
- [C# 9 Features](#-c-9-features)
- [C# 10 Features](#-c-10-features)
- [Interview Questions](#-interview-questions)

---

## 🔷 C# 7 Features

### Tuples and Deconstruction

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// TUPLES (C# 7.0)
// ═══════════════════════════════════════════════════════════════════════════

// Old: Tuple<int, string> - Item1, Item2 (unclear names)
// New: Named tuples
(int Id, string Name) GetPerson() => (1, "John");

var person = GetPerson();
Console.WriteLine(person.Id);    // Named access!
Console.WriteLine(person.Name);

// Inline declaration
(string First, string Last) name = ("John", "Doe");

// Deconstruction
var (id, name2) = GetPerson();  // Unpack into separate variables

// Deconstruction in class
public class Point
{
    public int X { get; set; }
    public int Y { get; set; }
    
    public void Deconstruct(out int x, out int y)
    {
        x = X;
        y = Y;
    }
}

var point = new Point { X = 10, Y = 20 };
var (x, y) = point;  // Calls Deconstruct
```

### Out Variables and Discards

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// OUT VARIABLES (C# 7.0)
// ═══════════════════════════════════════════════════════════════════════════

// Old: Must declare before
int result;
if (int.TryParse("123", out result)) { }

// New: Inline declaration
if (int.TryParse("123", out int value))
{
    Console.WriteLine(value);  // value in scope
}

// With var
if (int.TryParse("123", out var number))
{
    Console.WriteLine(number);
}

// Discards - ignore values you don't need
if (int.TryParse("123", out _))  // Don't need the value
{
    Console.WriteLine("Valid number");
}

var (name, _) = GetPerson();  // Ignore second value
```

### Pattern Matching

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// PATTERN MATCHING (C# 7.0)
// ═══════════════════════════════════════════════════════════════════════════

// Type pattern in is
object obj = "Hello";
if (obj is string s)  // Check type AND cast
{
    Console.WriteLine(s.Length);
}

// Switch with patterns
object data = 42;
switch (data)
{
    case int i when i > 0:
        Console.WriteLine($"Positive int: {i}");
        break;
    case int i:
        Console.WriteLine($"Non-positive int: {i}");
        break;
    case string s:
        Console.WriteLine($"String: {s}");
        break;
    case null:
        Console.WriteLine("Null");
        break;
    default:
        Console.WriteLine("Unknown");
        break;
}
```

### Local Functions

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// LOCAL FUNCTIONS (C# 7.0)
// ═══════════════════════════════════════════════════════════════════════════

public int Calculate(int n)
{
    // Local function - only visible in this method
    int Factorial(int x)
    {
        if (x <= 1) return 1;
        return x * Factorial(x - 1);
    }
    
    return Factorial(n);
}

// Benefits over lambda:
// - Can use ref/out parameters
// - Can be recursive
// - No allocation for delegate
```

---

## 🔶 C# 8 Features

### Nullable Reference Types

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// NULLABLE REFERENCE TYPES (C# 8.0)
// ═══════════════════════════════════════════════════════════════════════════

// Enable in project: <Nullable>enable</Nullable>

string name = null;      // Warning: Possible null
string? nullableName = null;  // OK - explicitly nullable

void Process(string name)
{
    Console.WriteLine(name.Length);  // Warning if name could be null
}

// Null-forgiving operator
string? maybe = GetValue();
string definite = maybe!;  // ! tells compiler "trust me, not null"
```

### Switch Expressions

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// SWITCH EXPRESSIONS (C# 8.0)
// ═══════════════════════════════════════════════════════════════════════════

// Old switch statement
string GetDayType(DayOfWeek day)
{
    switch (day)
    {
        case DayOfWeek.Saturday:
        case DayOfWeek.Sunday:
            return "Weekend";
        default:
            return "Weekday";
    }
}

// New switch expression
string GetDayType(DayOfWeek day) => day switch
{
    DayOfWeek.Saturday or DayOfWeek.Sunday => "Weekend",
    _ => "Weekday"  // _ is default
};

// With patterns
string Classify(object obj) => obj switch
{
    int i when i > 0 => "Positive int",
    int i => "Non-positive int",
    string s => $"String of length {s.Length}",
    null => "Null",
    _ => "Unknown"
};
```

### Indices and Ranges

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// INDICES AND RANGES (C# 8.0)
// ═══════════════════════════════════════════════════════════════════════════

int[] numbers = { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 };

// Index from end with ^
int last = numbers[^1];      // 9 (last element)
int secondLast = numbers[^2]; // 8

// Range with ..
int[] slice1 = numbers[2..5];   // {2, 3, 4} - indices 2,3,4
int[] slice2 = numbers[..3];    // {0, 1, 2} - first 3
int[] slice3 = numbers[7..];    // {7, 8, 9} - from 7 to end
int[] slice4 = numbers[^3..];   // {7, 8, 9} - last 3
int[] copy = numbers[..];       // Copy all

// Works with strings too
string text = "Hello World";
string sub = text[0..5];  // "Hello"
```

### Using Declarations

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// USING DECLARATIONS (C# 8.0)
// ═══════════════════════════════════════════════════════════════════════════

// Old: using statement with braces
using (var file = new StreamReader("file.txt"))
{
    // use file
}  // Disposed here

// New: using declaration (no braces)
using var file = new StreamReader("file.txt");
// use file
// Disposed at end of scope (method end)
```

### Default Interface Methods

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// DEFAULT INTERFACE METHODS (C# 8.0)
// ═══════════════════════════════════════════════════════════════════════════

public interface ILogger
{
    void Log(string message);
    
    // Default implementation - classes get it for free
    void LogError(string message) => Log($"ERROR: {message}");
    void LogWarning(string message) => Log($"WARNING: {message}");
}

public class ConsoleLogger : ILogger
{
    public void Log(string message) => Console.WriteLine(message);
    // LogError and LogWarning provided by interface
}
```

---

## 🔵 C# 9 Features

### Record Types

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// RECORDS (C# 9.0) - Immutable reference types with value equality
// ═══════════════════════════════════════════════════════════════════════════

// Positional record (shortest syntax)
public record Person(string Name, int Age);

// Equivalent to class with:
// - Init-only properties
// - Value-based equality
// - ToString() override
// - Deconstruct method
// - With-expression support

var person1 = new Person("John", 30);
var person2 = new Person("John", 30);

Console.WriteLine(person1 == person2);  // True (value equality!)
Console.WriteLine(person1);  // Person { Name = John, Age = 30 }

// With-expression for immutable "copy with changes"
var person3 = person1 with { Age = 31 };  // New object, Age changed

// Standard record syntax
public record Employee
{
    public string Name { get; init; }
    public string Department { get; init; }
}
```

### Init-Only Setters

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// INIT-ONLY SETTERS (C# 9.0)
// ═══════════════════════════════════════════════════════════════════════════

public class Person
{
    public string Name { get; init; }  // Can only set during initialization
    public int Age { get; init; }
}

var person = new Person 
{ 
    Name = "John",  // OK
    Age = 30        // OK
};

// person.Name = "Jane";  // ERROR! Can't change after init
```

### Top-Level Statements

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// TOP-LEVEL STATEMENTS (C# 9.0)
// ═══════════════════════════════════════════════════════════════════════════

// Old: Required class and Main method
class Program
{
    static void Main(string[] args)
    {
        Console.WriteLine("Hello");
    }
}

// New: Just write code directly (one file per project)
Console.WriteLine("Hello");
await DoSomethingAsync();
return 0;  // Optional return code
```

### Pattern Matching Enhancements

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// PATTERN ENHANCEMENTS (C# 9.0)
// ═══════════════════════════════════════════════════════════════════════════

// Relational patterns
string Classify(int n) => n switch
{
    < 0 => "Negative",
    0 => "Zero",
    > 0 and < 10 => "Small positive",
    >= 10 => "Large positive"
};

// Logical patterns: and, or, not
if (obj is not null) { }
if (n is > 0 and < 100) { }
if (day is DayOfWeek.Saturday or DayOfWeek.Sunday) { }

// Type pattern simplification
if (obj is string) { }  // No need for variable if not used
```

---

## 🟢 C# 10 Features

### Global Usings

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// GLOBAL USINGS (C# 10)
// ═══════════════════════════════════════════════════════════════════════════

// In GlobalUsings.cs or any file
global using System;
global using System.Collections.Generic;
global using System.Linq;

// Now these namespaces available in ALL files without using statement
```

### File-Scoped Namespaces

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// FILE-SCOPED NAMESPACES (C# 10)
// ═══════════════════════════════════════════════════════════════════════════

// Old: Braces add indentation
namespace MyApp.Services
{
    public class UserService
    {
        // ...
    }
}

// New: No braces, entire file in namespace
namespace MyApp.Services;

public class UserService
{
    // ...
}
```

### Record Structs

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// RECORD STRUCTS (C# 10)
// ═══════════════════════════════════════════════════════════════════════════

// Record class (reference type) - C# 9
public record class Person(string Name, int Age);

// Record struct (value type) - C# 10
public record struct Point(int X, int Y);  // Stack allocated

// Readonly record struct
public readonly record struct ReadOnlyPoint(int X, int Y);
```

### Extended Property Patterns

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// EXTENDED PROPERTY PATTERNS (C# 10)
// ═══════════════════════════════════════════════════════════════════════════

// Old: Nested pattern
if (person is { Address: { City: "London" } }) { }

// New: Dot notation
if (person is { Address.City: "London" }) { }

// In switch
string GetLocation(Person p) => p switch
{
    { Address.Country: "UK", Address.City: "London" } => "UK Capital",
    { Address.Country: "UK" } => "UK",
    _ => "Other"
};
```

---

## 🎤 Interview Questions

**Q1: What are records in C# 9?**
> Immutable reference types with value-based equality, built-in ToString, Deconstruct, and with-expressions.

**Q2: What is init-only setter?**
> Property that can only be set during object initialization, not after.

**Q3: Difference between record and record struct?**
> Record is reference type (heap). Record struct is value type (stack).

**Q4: What are nullable reference types?**
> C# 8 feature that makes reference types non-nullable by default, reducing null reference exceptions.

**Q5: What is the `with` expression?**
> Creates a copy of a record with specified properties changed: `record with { Prop = value }`

---

## 📝 Summary

| Version | Key Features |
|---------|--------------|
| **C# 7** | Tuples, out var, patterns, local functions |
| **C# 8** | Nullable refs, switch expr, ranges, using decl |
| **C# 9** | Records, init-only, top-level, relational patterns |
| **C# 10** | Global using, file-scoped ns, record struct |
