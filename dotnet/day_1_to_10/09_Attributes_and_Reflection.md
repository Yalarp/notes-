# 🏷️ Attributes and Reflection in C# - Complete Guide

## 📚 Table of Contents
- [Overview](#-overview)
- [Built-in Attributes](#-built-in-attributes)
- [Custom Attributes](#-custom-attributes)
- [Reflection](#-reflection)
- [Code Examples](#-code-examples)
- [Best Practices](#-best-practices)
- [Interview Questions](#-interview-questions)

---

## 🎯 Overview

**Attributes** are metadata tags that provide additional information about code elements (classes, methods, properties). **Reflection** is the ability to inspect and manipulate this metadata at runtime.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    ATTRIBUTES CONCEPT                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   [Attribute]              At Runtime (Reflection)                  │
│   public class MyClass     ─────────────────────────▶               │
│   {                        Read attribute values                    │
│       [Required]           Make decisions based on metadata         │
│       public string Name;  Validate, Serialize, Document            │
│   }                                                                 │
│                                                                     │
│   Use Cases:                                                        │
│   • Validation (Required, Range, RegularExpression)                 │
│   • Serialization control (JsonIgnore, XmlElement)                  │
│   • Documentation (Description, Obsolete)                           │
│   • Framework integration (Route, HttpGet in ASP.NET)               │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔷 Built-in Attributes

### Common Attributes

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// BUILT-IN ATTRIBUTES EXAMPLES
// ═══════════════════════════════════════════════════════════════════════════

using System;
using System.ComponentModel;
using System.ComponentModel.DataAnnotations;

// [Obsolete] - Marks code as deprecated
public class LegacySystem
{
    [Obsolete("Use NewMethod() instead", error: false)]
    // Line 1: Obsolete marks method as deprecated
    //         First param: warning message shown to developer
    //         error: false = warning only
    //         error: true = compile error if used
    public void OldMethod() { }
    
    public void NewMethod() { }
}

// [Serializable] - Allows binary serialization
[Serializable]
public class Data
{
    public string Value { get; set; }
    
    [NonSerialized]  // Excludes field from serialization
    private string tempData;
}

// [Conditional] - Method only called if symbol defined
public class Debugging
{
    [System.Diagnostics.Conditional("DEBUG")]
    // Line 2: Method calls are removed in Release builds
    //         Only executes when DEBUG symbol is defined
    public static void Log(string message)
    {
        Console.WriteLine($"DEBUG: {message}");
    }
}

// Validation Attributes (System.ComponentModel.DataAnnotations)
public class User
{
    [Required(ErrorMessage = "Name is required")]
    // Line 3: Property must have a value (not null/empty)
    public string Name { get; set; }
    
    [StringLength(50, MinimumLength = 3)]
    // Line 4: String must be 3-50 characters
    public string Username { get; set; }
    
    [Range(18, 120, ErrorMessage = "Age must be 18-120")]
    // Line 5: Numeric value must be in range
    public int Age { get; set; }
    
    [EmailAddress]
    // Line 6: Must be valid email format
    public string Email { get; set; }
    
    [RegularExpression(@"^\d{10}$", ErrorMessage = "Must be 10 digits")]
    // Line 7: Must match regex pattern
    public string Phone { get; set; }
    
    [Compare("Password", ErrorMessage = "Passwords don't match")]
    // Line 8: Must match another property's value
    public string ConfirmPassword { get; set; }
    public string Password { get; set; }
}

// Description and Display Attributes
public class Product
{
    [Description("Unique product identifier")]
    // Line 9: Provides description for documentation/UI
    public int Id { get; set; }
    
    [DisplayName("Product Name")]
    // Line 10: Display label for UI binding
    public string Name { get; set; }
    
    [DefaultValue(0)]
    // Line 11: Specifies default value for designers
    public decimal Price { get; set; }
    
    [Browsable(false)]
    // Line 12: Hides from property grid in designers
    public string InternalCode { get; set; }
}
```

### Attribute Targets

```csharp
using System;

// Attributes can target different code elements
[assembly: AssemblyVersion("1.0.0.0")]  // Assembly level
[module: SomeModuleAttribute]            // Module level

[type: Serializable]                     // Class/struct/enum
public class Example
{
    [field: NonSerialized]               // Field
    private string _data;
    
    [property: Description("Desc")]      // Property
    public string Data 
    { 
        [method: Obsolete]               // Getter method
        get => _data; 
        set => _data = value; 
    }
    
    [method: Obsolete]                   // Method
    [return: MarshalAs(UnmanagedType.Bool)]  // Return value
    public bool Process(
        [param: Required] string input)  // Parameter
    {
        return true;
    }
}
```

---

## 🔶 Custom Attributes

### Creating a Custom Attribute

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// CUSTOM ATTRIBUTE CREATION
// ═══════════════════════════════════════════════════════════════════════════

using System;

// Line 1: Custom attributes inherit from System.Attribute
// Line 2: [AttributeUsage] specifies where attribute can be applied
[AttributeUsage(
    AttributeTargets.Class | AttributeTargets.Method,  // Can apply to classes and methods
    AllowMultiple = false,                              // Only one per element
    Inherited = true                                    // Inherited by derived classes
)]
public class AuthorAttribute : Attribute
{
    // Line 3: Properties store attribute data
    public string Name { get; }
    public string Email { get; set; }  // Optional (has setter)
    public string Version { get; set; }
    
    // Line 4: Constructor for required parameters
    public AuthorAttribute(string name)
    {
        Name = name;
    }
}

// Line 5: Applying custom attribute
[Author("John Doe", Email = "john@example.com", Version = "1.0")]
public class MyComponent
{
    [Author("Jane Smith")]
    public void DoWork() { }
}

// ═══════════════════════════════════════════════════════════════════════════
// VALIDATION ATTRIBUTE EXAMPLE
// ═══════════════════════════════════════════════════════════════════════════

[AttributeUsage(AttributeTargets.Property)]
public class MinValueAttribute : Attribute
{
    public int Minimum { get; }
    public string ErrorMessage { get; set; }
    
    public MinValueAttribute(int minimum)
    {
        Minimum = minimum;
        ErrorMessage = $"Value must be at least {minimum}";
    }
    
    public bool IsValid(int value)
    {
        return value >= Minimum;
    }
}

public class Order
{
    [MinValue(1, ErrorMessage = "Quantity must be at least 1")]
    public int Quantity { get; set; }
}
```

---

## 🔍 Reflection

### Reading Attributes at Runtime

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// REFLECTION - READING ATTRIBUTES AT RUNTIME
// ═══════════════════════════════════════════════════════════════════════════

using System;
using System.Reflection;

class ReflectionDemo
{
    static void Main()
    {
        // ─────────────────────────────────────────────────────────────────
        // GET TYPE INFORMATION
        // ─────────────────────────────────────────────────────────────────
        
        Type type = typeof(MyComponent);
        // Line 1: Get Type object for a class
        //         typeof() is compile-time
        //         obj.GetType() is runtime
        
        Console.WriteLine($"Type: {type.Name}");
        Console.WriteLine($"Full Name: {type.FullName}");
        Console.WriteLine($"Namespace: {type.Namespace}");
        
        // ─────────────────────────────────────────────────────────────────
        // READ CLASS ATTRIBUTES
        // ─────────────────────────────────────────────────────────────────
        
        // Check if attribute exists
        if (type.IsDefined(typeof(AuthorAttribute), inherit: true))
        {
            Console.WriteLine("Has Author attribute");
        }
        
        // Get attribute instance
        AuthorAttribute author = type.GetCustomAttribute<AuthorAttribute>();
        // Line 2: GetCustomAttribute<T>() returns single attribute
        //         Returns null if not found
        
        if (author != null)
        {
            Console.WriteLine($"Author: {author.Name}");
            Console.WriteLine($"Email: {author.Email}");
        }
        
        // Get all attributes
        object[] allAttrs = type.GetCustomAttributes(inherit: true);
        // Line 3: GetCustomAttributes() returns array of all attributes
        //         inherit: true includes attributes from base classes
        
        // ─────────────────────────────────────────────────────────────────
        // READ METHOD ATTRIBUTES
        // ─────────────────────────────────────────────────────────────────
        
        MethodInfo method = type.GetMethod("DoWork");
        // Line 4: GetMethod() finds method by name
        
        AuthorAttribute methodAuthor = method.GetCustomAttribute<AuthorAttribute>();
        if (methodAuthor != null)
        {
            Console.WriteLine($"Method author: {methodAuthor.Name}");
        }
        
        // ─────────────────────────────────────────────────────────────────
        // READ PROPERTY ATTRIBUTES
        // ─────────────────────────────────────────────────────────────────
        
        Type orderType = typeof(Order);
        PropertyInfo prop = orderType.GetProperty("Quantity");
        
        MinValueAttribute minVal = prop.GetCustomAttribute<MinValueAttribute>();
        if (minVal != null)
        {
            Console.WriteLine($"Min value: {minVal.Minimum}");
            
            // Validate a value
            int testValue = 0;
            if (!minVal.IsValid(testValue))
            {
                Console.WriteLine(minVal.ErrorMessage);
            }
        }
    }
}
```

### Reflection for Dynamic Object Inspection

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// REFLECTION - DYNAMIC OBJECT INSPECTION
// ═══════════════════════════════════════════════════════════════════════════

using System;
using System.Reflection;

class DynamicInspection
{
    static void PrintObjectInfo(object obj)
    {
        Type type = obj.GetType();
        // Line 1: Get runtime type of object
        
        Console.WriteLine($"=== {type.Name} ===");
        
        // Get all public properties
        PropertyInfo[] properties = type.GetProperties();
        foreach (PropertyInfo prop in properties)
        {
            object value = prop.GetValue(obj);
            // Line 2: GetValue() reads property value from object
            
            Console.WriteLine($"  {prop.Name}: {value}");
        }
        
        // Get all public methods
        MethodInfo[] methods = type.GetMethods(BindingFlags.Public | BindingFlags.Instance | BindingFlags.DeclaredOnly);
        // Line 3: BindingFlags filter what members to include
        //         DeclaredOnly = exclude inherited members
        
        Console.WriteLine("  Methods:");
        foreach (MethodInfo method in methods)
        {
            Console.WriteLine($"    - {method.Name}()");
        }
    }
    
    static void Main()
    {
        var person = new { Name = "John", Age = 30 };
        PrintObjectInfo(person);
        
        /* Output:
        === <>f__AnonymousType ===
          Name: John
          Age: 30
          Methods:
        */
    }
}
```

### Dynamic Method Invocation

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// REFLECTION - DYNAMIC METHOD INVOCATION
// ═══════════════════════════════════════════════════════════════════════════

using System;
using System.Reflection;

public class Calculator
{
    public int Add(int a, int b) => a + b;
    public int Multiply(int a, int b) => a * b;
}

class DynamicInvocation
{
    static void Main()
    {
        // Create instance dynamically
        Type calcType = typeof(Calculator);
        object calc = Activator.CreateInstance(calcType);
        // Line 1: Activator.CreateInstance creates object of type
        //         Calls parameterless constructor
        
        // Get method info
        MethodInfo addMethod = calcType.GetMethod("Add");
        
        // Invoke method dynamically
        object[] parameters = { 5, 3 };
        object result = addMethod.Invoke(calc, parameters);
        // Line 2: Invoke() calls method with parameters
        //         First param: object instance (null for static)
        //         Second param: array of method arguments
        
        Console.WriteLine($"5 + 3 = {result}");  // Output: 5 + 3 = 8
        
        // Call any method by name
        string methodName = "Multiply";
        MethodInfo method = calcType.GetMethod(methodName);
        result = method.Invoke(calc, new object[] { 4, 7 });
        Console.WriteLine($"4 * 7 = {result}");  // Output: 4 * 7 = 28
    }
}
```

---

## ✅ Best Practices

1. **Keep attributes simple** - Only store metadata, not business logic
2. **Use built-in attributes first** - Don't reinvent the wheel
3. **Use AttributeUsage** - Restrict where custom attributes can be applied
4. **Cache reflection results** - Reflection is slow, cache Type and MemberInfo
5. **Validate with attributes** - Use DataAnnotations for input validation

```csharp
// ✅ GOOD: Cache Type object
private static readonly Type _myType = typeof(MyClass);

// ❌ BAD: Repeated typeof calls in loop
foreach (var item in items)
{
    Type t = typeof(MyClass);  // Wasteful
}
```

---

## 🎤 Interview Questions

**Q1: What are attributes in C#?**
> Attributes are declarative metadata tags that provide additional information about code elements. They're applied using square brackets `[AttributeName]`.

**Q2: What is reflection?**
> Reflection is the ability to inspect types, methods, properties at runtime and dynamically invoke methods or access members.

**Q3: How to create a custom attribute?**
> Create a class inheriting from `System.Attribute`. Use `[AttributeUsage]` to control where it can be applied.

**Q4: What is `typeof()` vs `GetType()`?**
> `typeof(ClassName)` is compile-time, works without instance. `obj.GetType()` is runtime, requires object instance.

**Q5: Why is reflection slow?**
> Reflection bypasses compile-time optimizations, requires metadata parsing, and involves boxing/unboxing.

---

## 📝 Summary

| Concept | Description |
|---------|-------------|
| **Attribute** | Metadata tag: `[AttributeName]` |
| **Custom Attribute** | Inherit from `Attribute` class |
| **AttributeUsage** | Controls valid targets and inheritance |
| **typeof(T)** | Get Type at compile time |
| **GetType()** | Get Type at runtime from instance |
| **GetCustomAttribute<T>** | Read attribute from member |
| **Invoke()** | Call method dynamically |
