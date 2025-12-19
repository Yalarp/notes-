# 🔧 Extension Methods in C#

## 📚 Table of Contents
- [Overview](#-overview)
- [Creating Extension Methods](#-creating-extension-methods)
- [Common Use Cases](#-common-use-cases)
- [Best Practices](#-best-practices)
- [Interview Questions](#-interview-questions)

---

## 🎯 Overview

**Extension methods** add new methods to existing types without modifying them or creating derived types.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    EXTENSION METHOD CONCEPT                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   String class (cannot modify)                                      │
│   ┌────────────────────────────┐                                   │
│   │ • Length                   │                                   │
│   │ • ToUpper()                │                                   │
│   │ • Substring()              │                                   │
│   │                            │                                   │
│   │ + WordCount()  ◄──── Added via extension method                │
│   │ + Reverse()    ◄──── Not part of original class                │
│   └────────────────────────────┘                                   │
│                                                                     │
│   Appears as instance method, but is static method underneath     │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔷 Creating Extension Methods

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// EXTENSION METHOD SYNTAX
// ═══════════════════════════════════════════════════════════════════════════

using System;

// Line 1: Extension methods must be in STATIC class
public static class StringExtensions
{
    // Line 2: Extension method is STATIC
    // Line 3: First parameter has 'this' keyword = type being extended
    public static int WordCount(this string str)
    {
        if (string.IsNullOrEmpty(str))
            return 0;
            
        return str.Split(' ', StringSplitOptions.RemoveEmptyEntries).Length;
    }
    
    // Extension with additional parameters
    public static string Repeat(this string str, int count)
    // Line 4: 'this string str' = extends string
    //         'int count' = additional parameter
    {
        return string.Concat(Enumerable.Repeat(str, count));
    }
    
    // Extension for nullable type
    public static bool IsNullOrEmpty(this string str)
    {
        return string.IsNullOrEmpty(str);
    }
}

class ExtensionDemo
{
    static void Main()
    {
        string text = "Hello World Example";
        
        // Line 5: Call extension method like instance method
        int count = text.WordCount();
        // Looks like: text.WordCount()
        // Actually:   StringExtensions.WordCount(text)
        
        Console.WriteLine($"Words: {count}");  // Output: 3
        
        // With parameter
        string repeated = "Ha".Repeat(3);
        Console.WriteLine(repeated);  // Output: HaHaHa
        
        // Works on null with null check in method
        string nullStr = null;
        bool isEmpty = nullStr.IsNullOrEmpty();  // true
        // Note: Extension method is called, not instance method
        //       So null doesn't cause NullReferenceException
    }
}
```

---

## 🔶 Common Use Cases

### 1. Extending Built-in Types

```csharp
public static class IntExtensions
{
    // Check if number is between range
    public static bool IsBetween(this int value, int min, int max)
    {
        return value >= min && value <= max;
    }
    
    // Convert to ordinal (1st, 2nd, 3rd)
    public static string ToOrdinal(this int num)
    {
        if (num <= 0) return num.ToString();
        
        int lastDigit = num % 10;
        int lastTwoDigits = num % 100;
        
        if (lastTwoDigits >= 11 && lastTwoDigits <= 13)
            return num + "th";
            
        return lastDigit switch
        {
            1 => num + "st",
            2 => num + "nd",
            3 => num + "rd",
            _ => num + "th"
        };
    }
}

// Usage:
int age = 25;
if (age.IsBetween(18, 65)) { }  // true
Console.WriteLine(1.ToOrdinal());  // "1st"
Console.WriteLine(3.ToOrdinal());  // "3rd"
Console.WriteLine(11.ToOrdinal()); // "11th"
```

### 2. Extending Collections

```csharp
public static class CollectionExtensions
{
    // ForEach for IEnumerable (like List.ForEach)
    public static void ForEach<T>(this IEnumerable<T> source, Action<T> action)
    {
        foreach (T item in source)
        {
            action(item);
        }
    }
    
    // Safe first or default with custom default
    public static T FirstOrDefault<T>(this IEnumerable<T> source, T defaultValue)
    {
        foreach (T item in source)
            return item;
        return defaultValue;
    }
    
    // Check if collection is empty
    public static bool IsEmpty<T>(this IEnumerable<T> source)
    {
        return !source.Any();
    }
}

// Usage:
var numbers = new[] { 1, 2, 3, 4, 5 };

numbers.ForEach(n => Console.WriteLine(n));

var empty = new List<int>();
bool isEmpty = empty.IsEmpty();  // true
```

### 3. Fluent API Pattern

```csharp
public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
    public string Email { get; set; }
}

public static class PersonExtensions
{
    public static Person WithName(this Person p, string name)
    {
        p.Name = name;
        return p;  // Return same object for chaining
    }
    
    public static Person WithAge(this Person p, int age)
    {
        p.Age = age;
        return p;
    }
    
    public static Person WithEmail(this Person p, string email)
    {
        p.Email = email;
        return p;
    }
}

// Usage - Fluent chaining:
Person person = new Person()
    .WithName("John")
    .WithAge(30)
    .WithEmail("john@example.com");
```

---

## ✅ Best Practices

### 1. Use Meaningful Static Class Names
```csharp
// ✅ GOOD: Clear naming
public static class StringExtensions { }
public static class DateTimeExtensions { }

// ❌ BAD: Vague names
public static class Helpers { }
public static class Utils { }
```

### 2. Put in Dedicated Namespace
```csharp
namespace MyApp.Extensions
{
    public static class StringExtensions { }
}

// Consumer must explicitly import
using MyApp.Extensions;
```

### 3. Don't Override Instance Methods
```csharp
// ❌ BAD: Extension with same name as instance method
public static int Length(this string s) { }  // Never called!
// Instance method String.Length always wins
```

---

## 🎤 Interview Questions

**Q1: What is an extension method?**
> Static method that appears as instance method on extended type. Uses `this` keyword on first parameter.

**Q2: Rules for extension methods?**
> Must be in static class, method must be static, first parameter must have `this` modifier.

**Q3: Extension vs instance method priority?**
> Instance methods always take priority. Extension only called if no matching instance method exists.

**Q4: How is LINQ implemented?**
> LINQ operators (Where, Select, etc.) are extension methods on IEnumerable<T>.

---

## 📝 Summary

| Rule | Example |
|------|---------|
| Static class | `public static class Extensions` |
| Static method | `public static int Method()` |
| `this` parameter | `(this string s)` |
| Call syntax | `"text".Method()` |
