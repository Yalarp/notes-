# 📦 Serialization in C# - Complete Guide

## 📚 Table of Contents
- [Overview](#-overview)
- [Types of Serialization](#-types-of-serialization)
- [XML Serialization](#-xml-serialization)
- [JSON Serialization](#-json-serialization)
- [Best Practices](#-best-practices)
- [Interview Questions](#-interview-questions)

---

## 🎯 Overview

**Serialization** is converting an object's state into a format that can be stored or transmitted, and later reconstructed (deserialized) back into an object.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SERIALIZATION CONCEPT                            │
├─────────────────────────────────────────────────────────────────────┤
│   ┌──────────────┐   Serialize    ┌──────────────┐                 │
│   │   Object     │ ════════════▶  │   Bytes/     │                 │
│   │   in Memory  │                │   Text       │                 │
│   └──────────────┘                └──────┬───────┘                 │
│         ▲                                │                         │
│         │          Deserialize           │                         │
│         └────────────────────────────────┘                         │
│                                                                     │
│   Use Cases: Save state, Send over network, Cache, Deep copy       │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔷 Types of Serialization

| Feature | Binary | XML | JSON |
|---------|--------|-----|------|
| **Size** | Smallest | Largest | Medium |
| **Human Readable** | ❌ No | ✅ Yes | ✅ Yes |
| **Cross-Platform** | ❌ .NET only | ✅ Yes | ✅ Yes |
| **Web APIs** | ❌ Not used | ✅ SOAP | ✅ REST |

---

## 🟢 XML Serialization

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// XML SERIALIZATION EXAMPLE
// ═══════════════════════════════════════════════════════════════════════════

using System;
using System.IO;
using System.Xml.Serialization;

// No [Serializable] needed - uses public properties only
// REQUIRES parameterless constructor
public class Employee
{
    public int Id { get; set; }           // Serialized as <Id>value</Id>
    public string Name { get; set; }       // Serialized as <Name>value</Name>
    
    [XmlIgnore]                           // Excludes from serialization
    public string Password { get; set; }
    
    [XmlAttribute("dept")]                // Makes it an attribute: <Employee dept="IT">
    public string Department { get; set; }
    
    [XmlElement("HireDate")]              // Custom element name
    public DateTime JoiningDate { get; set; }
    
    public Employee() { }                  // REQUIRED for deserialization
}

class XmlDemo
{
    static void Main()
    {
        Employee emp = new Employee
        {
            Id = 101, Name = "Alice", Department = "IT",
            JoiningDate = DateTime.Now, Password = "secret"
        };

        string filePath = @"C:\Demo\employee.xml";
        
        // SERIALIZE
        XmlSerializer serializer = new XmlSerializer(typeof(Employee));
        using (FileStream stream = new FileStream(filePath, FileMode.Create))
        {
            serializer.Serialize(stream, emp);
        }
        
        // DESERIALIZE
        using (FileStream stream = new FileStream(filePath, FileMode.Open))
        {
            Employee loaded = (Employee)serializer.Deserialize(stream);
            Console.WriteLine($"Loaded: {loaded.Name}");
        }
    }
}

/* OUTPUT XML:
<?xml version="1.0"?>
<Employee dept="IT">
  <Id>101</Id>
  <Name>Alice</Name>
  <HireDate>2023-11-15T10:30:00</HireDate>
</Employee>
*/
```

### XML Collection Serialization

```csharp
using System.Collections.Generic;
using System.Xml.Serialization;

[XmlRoot("CompanyData")]              // Custom root element
public class Company
{
    public string CompanyName { get; set; }
    
    [XmlArray("Staff")]               // Collection wrapper element
    [XmlArrayItem("Member")]          // Individual item element
    public List<Employee> Employees { get; set; }
    
    public Company() { Employees = new List<Employee>(); }
}

/* OUTPUT:
<CompanyData>
  <CompanyName>TechCorp</CompanyName>
  <Staff>
    <Member dept="IT"><Id>1</Id><Name>John</Name></Member>
    <Member dept="HR"><Id>2</Id><Name>Jane</Name></Member>
  </Staff>
</CompanyData>
*/
```

---

## 🟠 JSON Serialization

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// JSON SERIALIZATION WITH System.Text.Json (.NET Core 3.0+)
// ═══════════════════════════════════════════════════════════════════════════

using System;
using System.IO;
using System.Text.Json;
using System.Text.Json.Serialization;

public class Product
{
    public int ProductId { get; set; }
    public string ProductName { get; set; }
    public decimal Price { get; set; }
    
    [JsonPropertyName("mfg_date")]    // Custom JSON property name
    public DateTime ManufactureDate { get; set; }
    
    [JsonIgnore]                      // Exclude from JSON
    public string InternalCode { get; set; }
    
    public ProductStatus Status { get; set; }
}

public enum ProductStatus { Available, OutOfStock, Discontinued }

class JsonDemo
{
    static void Main()
    {
        Product product = new Product
        {
            ProductId = 1001, ProductName = "Laptop", Price = 999.99m,
            ManufactureDate = DateTime.Now, Status = ProductStatus.Available
        };

        // ─────────────────────────────────────────────────────────────────
        // Configure Serialization Options
        // ─────────────────────────────────────────────────────────────────
        JsonSerializerOptions options = new JsonSerializerOptions
        {
            WriteIndented = true,                              // Pretty print
            PropertyNamingPolicy = JsonNamingPolicy.CamelCase, // camelCase
            Converters = { new JsonStringEnumConverter() }     // Enum as string
        };

        // SERIALIZE to string
        string json = JsonSerializer.Serialize(product, options);
        Console.WriteLine(json);
        
        /* OUTPUT:
        {
          "productId": 1001,
          "productName": "Laptop",
          "price": 999.99,
          "mfg_date": "2023-11-15T10:30:00",
          "status": "Available"
        }
        */

        // DESERIALIZE from string
        string input = @"{""productId"": 2002, ""productName"": ""Mouse"", ""price"": 25.00}";
        Product loaded = JsonSerializer.Deserialize<Product>(input, options);
        Console.WriteLine($"Loaded: {loaded.ProductName}");

        // SERIALIZE to file
        using (FileStream stream = File.Create(@"C:\Demo\product.json"))
        {
            JsonSerializer.Serialize(stream, product, options);
        }

        // DESERIALIZE from file
        using (FileStream stream = File.OpenRead(@"C:\Demo\product.json"))
        {
            Product fromFile = JsonSerializer.Deserialize<Product>(stream, options);
        }
    }
}
```

### JSON with Collections and Nested Objects

```csharp
public class Order
{
    public int OrderId { get; set; }
    public Customer Customer { get; set; }           // Nested object
    public List<OrderItem> Items { get; set; }       // Array
    public Dictionary<string, string> Meta { get; set; } // Key-value pairs
}

public class Customer { public string Name { get; set; } public string Email { get; set; } }
public class OrderItem { public string Product { get; set; } public int Qty { get; set; } }

/* OUTPUT:
{
  "orderId": 5001,
  "customer": { "name": "Bob", "email": "bob@test.com" },
  "items": [
    { "product": "Mouse", "qty": 2 },
    { "product": "Keyboard", "qty": 1 }
  ],
  "meta": { "source": "web", "promo": "SAVE10" }
}
*/
```

### JsonSerializerOptions Reference

```csharp
var options = new JsonSerializerOptions
{
    // Formatting
    WriteIndented = true,                    // Pretty print
    
    // Naming
    PropertyNamingPolicy = JsonNamingPolicy.CamelCase,  // camelCase props
    PropertyNameCaseInsensitive = true,      // Ignore case on deserialize
    
    // Null handling
    DefaultIgnoreCondition = JsonIgnoreCondition.WhenWritingNull,
    
    // Include fields (not just properties)
    IncludeFields = true,
    
    // Converters
    Converters = { new JsonStringEnumConverter() },
    
    // Security
    MaxDepth = 64,                          // Max nesting
    AllowTrailingCommas = true,             // Allow {a: 1,}
    ReadCommentHandling = JsonCommentHandling.Skip
};
```

---

## ✅ Best Practices

### 1. Reuse Serializer Options
```csharp
// ✅ GOOD: Static singleton options
private static readonly JsonSerializerOptions _options = new()
{
    WriteIndented = true,
    PropertyNamingPolicy = JsonNamingPolicy.CamelCase
};

// ❌ BAD: Creating options each time
JsonSerializer.Serialize(obj, new JsonSerializerOptions { WriteIndented = true });
```

### 2. Always Have Parameterless Constructor (XML)
```csharp
public class Data
{
    public Data() { }  // REQUIRED for XML/JSON deserialization
    public Data(string value) { Value = value; }
    public string Value { get; set; }
}
```

### 3. Handle Circular References
```csharp
var options = new JsonSerializerOptions
{
    ReferenceHandler = ReferenceHandler.Preserve  // Handles cycles
};
```

---

## ⚠️ Common Pitfalls

| Issue | Problem | Solution |
|-------|---------|----------|
| No parameterless ctor | XML/JSON can't deserialize | Add `public ClassName() { }` |
| Circular reference | Stack overflow | Use `[JsonIgnore]` or `ReferenceHandler.Preserve` |
| Private props | Not serialized | Use `[JsonInclude]` |
| Wrong encoding | Garbled text | Specify `Encoding.UTF8` |

---

## 🎤 Interview Questions

**Q1: What is serialization?**
> Converting object state to storable/transmittable format (bytes/text) that can be reconstructed later.

**Q2: JSON vs XML - when to use each?**
> JSON: REST APIs, web apps, cross-platform, smaller size. XML: SOAP services, config files, schema validation needed.

**Q3: How to exclude a property from JSON serialization?**
> Use `[JsonIgnore]` attribute on the property.

**Q4: How to serialize enum as string in JSON?**
> Add `new JsonStringEnumConverter()` to `JsonSerializerOptions.Converters`.

**Q5: What's required for XML deserialization?**
> Parameterless constructor and public properties.

---

## 📝 Summary

| Aspect | XML | JSON |
|--------|-----|------|
| **Namespace** | `System.Xml.Serialization` | `System.Text.Json` |
| **Exclude** | `[XmlIgnore]` | `[JsonIgnore]` |
| **Rename** | `[XmlElement("name")]` | `[JsonPropertyName("name")]` |
| **Constructor** | Required parameterless | Recommended |

```csharp
// Quick Reference
// JSON
string json = JsonSerializer.Serialize(obj, options);
MyClass obj = JsonSerializer.Deserialize<MyClass>(json);

// XML
var xml = new XmlSerializer(typeof(MyClass));
xml.Serialize(stream, obj);
MyClass obj = (MyClass)xml.Deserialize(stream);
```
