# 04 - Assemblies and Namespaces

## Table of Contents
- [Understanding Assemblies](#understanding-assemblies)
- [What is an Assembly?](#what-is-an-assembly)
- [Types of Assemblies](#types-of-assemblies)
- [Assembly Structure](#assembly-structure)
- [Namespaces](#namespaces)
- [Using Directives](#using-directives)
- [Namespace Aliases](#namespace-aliases)
- [Global Using (C# 10+)](#global-using-c-10)
- [Assembly Versioning](#assembly-versioning)
- [Strong Naming](#strong-naming)
- [Key Takeaways](#key-takeaways)

---

## Understanding Assemblies

An **assembly** is the fundamental unit of deployment, version control, and security in .NET applications. It's a compiled code library used by .NET applications.

### What is an Assembly?

Think of an assembly as a container that holds:
- **Compiled code** (IL - Intermediate Language)
- **Metadata** (information about types, methods, etc.)
- **Resources** (images, strings, etc.)
- **Manifest** (assembly information, versioning, dependencies)

```
Assembly (.dll or .exe)
├── Manifest (metadata about the assembly)
│   ├── Version information
│   ├── Culture information
│   ├── Referenced assemblies
│   └── Security permissions
├── Type Metadata
│   ├── Classes
│   ├── Interfaces
│   ├── Structs
│   └── Enums
├── IL Code (Intermediate Language)
└── Resources
    ├── Images
    ├── Strings
    └── Other files
```

---

## Types of Assemblies

### 1. Private Assembly

**Private assemblies** are used by a single application.

```csharp
// MyApp.exe structure
MyApp/
├── MyApp.exe           // Main executable assembly
├── MyLib.dll            // Private assembly (only MyApp uses it)
└── appsettings.json    // Configuration
```

**Characteristics:**
- Stored in application directory
- No version conflicts (each app has its own copy)
- No registration required
- Default type for most applications

### 2. Shared Assembly (GAC)

**Shared assemblies** can be used by multiple applications. Stored in **Global Assembly Cache (GAC)**.

```
C:\Windows\Microsoft.NET\assembly\GAC_MSIL\
├── System/
├── System.Data/
└── Your.Shared.Assembly/
```

**Characteristics:**
- Installed in GAC (Global Assembly Cache)
- Must be strongly named
- Shared across multiple applications
- Requires admin rights to install
- Common for framework libraries

### 3. Executable Assembly (.exe)

Contains an entry point (Main method).

```csharp
// Program.cs - Creates .exe assembly
using System;

namespace MyApplication
{
    class Program
    {
        static void Main(string[] args)  // Entry point
        {
            Console.WriteLine("This is an executable assembly!");
        }
    }
}
```

**Compilation:**
```bash
csc Program.cs
# Produces: MyApplication.exe
```

### 4. Library Assembly (.dll)

Contains reusable code, no entry point.

```csharp
// MathLibrary.cs - Creates .dll assembly
namespace MathLibrary
{
    public class Calculator
    {
        public int Add(int a, int b)
        {
            return a + b;
        }
        
        public int Subtract(int a, int b)
        {
            return a - b;
        }
    }
}
```

**Compilation:**
```bash
csc /target:library MathLibrary.cs
# Produces: MathLibrary.dll
```

**Using the DLL:**
```csharp
// ConsumerApp.cs
using MathLibrary;  // Import the namespace

class Program
{
    static void Main()
    {
        Calculator calc = new Calculator();
        int result = calc.Add(5, 3);
        Console.WriteLine($"Result: {result}");
    }
}
```

**Compile with reference:**
```bash
csc /reference:MathLibrary.dll ConsumerApp.cs
```

---

## Assembly Structure

### Viewing Assembly Contents

```csharp
using System;
using System.Reflection;  // For assembly inspection

class AssemblyInspection
{
    static void Main()
    {
        // GET CURRENT ASSEMBLY
        Assembly currentAssembly = Assembly.GetExecutingAssembly();
        
        Console.WriteLine("=== Current Assembly Information ===");
        Console.WriteLine($"Full Name: {currentAssembly.FullName}");
        Console.WriteLine($"Location: {currentAssembly.Location}");
        Console.WriteLine($"Version: {currentAssembly.GetName().Version}");
        
        // GET ALL TYPES IN ASSEMBLY
        Console.WriteLine($"\\n=== Types in Assembly ===");
        Type[] types = currentAssembly.GetTypes();
        foreach (Type type in types)
        {
            Console.WriteLine($"- {type.FullName} ({type.GetType Kind()})");
        }
        
        // GET REFERENCED ASSEMBLIES
        Console.WriteLine($"\\n=== Referenced Assemblies ===");
        AssemblyName[] references = currentAssembly.GetReferencedAssemblies();
        foreach (AssemblyName reference in references)
        {
            Console.WriteLine($"- {reference.Name}, Version: {reference.Version}");
        }
        
        // LOAD ASSEMBLY BY NAME
        Assembly systemAssembly = Assembly.Load("System.Runtime");
        Console.WriteLine($"\\nLoaded: {systemAssembly.GetName().Name}");
        
        // CUSTOM ATTRIBUTES
        Console.WriteLine($"\\n=== Custom Attributes ===");
        object[] attributes = currentAssembly.GetCustomAttributes(false);
        foreach (object attr in attributes)
        {
            Console.WriteLine($"- {attr.GetType().Name}");
        }
    }
}
```

**Line-by-Line Explanation:**
- `Assembly.GetExecutingAssembly()` - Gets assembly containing currently executing code
  - Returns Assembly object representing the current .exe or .dll
- `currentAssembly.FullName` - Gets complete assembly name with version and culture
  - Format: "AssemblyName, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null"
- `currentAssembly.Location` - Physical file path to the assembly
- `currentAssembly.GetTypes()` - Returns array of all types defined in assembly
- `type.GetTypeKind()` - Indicates if Class, Interface, Struct, Enum, etc.
- `GetReferencedAssemblies()` - Lists all assemblies this assembly depends on
- `Assembly.Load("System.Runtime")` - Loads assembly by name from GAC or app directory

**Execution Flow:**
1. Get reference to executing assembly
2. Display assembly metadata (name, location, version)
3. Enumerate all types (classes, structs, etc.) in assembly
4. List all referenced assemblies
5. Demonstrate loading external assembly
6. Display custom attributes applied to assembly

### Assembly Manifest

The **manifest** contains metadata about the assembly:

```csharp
// AssemblyInfo.cs (automatically generated in older projects)
using System.Reflection;
using System.Runtime.CompilerServices;
using System.Runtime.InteropServices;

// General Information
[assembly: AssemblyTitle("MyApplication")]
[assembly: AssemblyDescription("A sample application")]
[assembly: AssemblyConfiguration("")]
[assembly: AssemblyCompany("My Company")]
[assembly: AssemblyProduct("MyApplication")]
[assembly: AssemblyCopyright("Copyright © 2024")]
[assembly: AssemblyTrademark("")]
[assembly: AssemblyCulture("")]

// Version Information
[assembly: AssemblyVersion("1.0.0.0")]
[assembly: AssemblyFileVersion("1.0.0.0")]

// COM Visibility
[assembly: ComVisible(false)]

// GUID for COM exposure
[assembly: Guid("12345678-1234-1234-1234-123456789012")]
```

**Modern .csproj (SDK-style):**
```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net8.0</TargetFramework>
    
    <!-- Assembly Information -->
    <AssemblyName>MyApplication</AssemblyName>
    <RootNamespace>MyApplication</RootNamespace>
    <Version>1.0.0</Version>
    <Authors>Your Name</Authors>
    <Company>Your Company</Company>
    <Product>MyApplication</Product>
    <Description>Application description</Description>
    <Copyright>Copyright © 2024</Copyright>
  </PropertyGroup>
</Project>
```

---

## Namespaces

**Namespaces** organize code into logical groups and prevent naming conflicts.

### Why Namespaces?

```csharp
// WITHOUT NAMESPACES - Naming conflicts!

// File1.cs
class Customer
{
    public string Name { get; set; }
}

// File2.cs (from third-party library)
class Customer  // ERROR: 'Customer' already defined!
{
    public int ID { get; set; }
}
```

```csharp
// WITH NAMESPACES - No conflict!

// File1.cs
namespace MyCompany.Sales
{
    class Customer
    {
        public string Name { get; set; }
    }
}

// File2.cs
namespace ThirdParty.CRM
{
    class Customer
    {
        public int ID { get; set; }
    }
}

// Usage - Fully qualified names
class Program
{
    static void Main()
    {
        MyCompany.Sales.Customer salesCustomer = 
            new MyCompany.Sales.Customer();
            
        ThirdParty.CRM.Customer crmCustomer = 
            new ThirdParty.CRM.Customer();
    }
}
```

### Declaring Namespaces

```csharp
using System;

// SINGLE NAMESPACE
namespace MyCompany.ProjectA
{
    class ClassA
    {
        public void Method1()
        {
            Console.WriteLine("ClassA.Method1");
        }
    }
}

// NESTED NAMESPACES (Traditional)
namespace MyCompany
{
    namespace ProjectA
    {
        namespace Utilities
        {
            class Helper
            {
                public static void DoWork()
                {
                    Console.WriteLine("Helper.DoWork");
                }
            }
        }
    }
}

// NESTED NAMESPACES (Modern - C# 10+)
namespace MyCompany.ProjectA.Utilities;  // File-scoped namespace

class Helper2
{
    public static void DoWork()
    {
        Console.WriteLine("Helper2.DoWork");
    }
}
// Everything in this file is in MyCompany.ProjectA.Utilities
```

**Line-by-Line Explanation:**
- `namespace MyCompany.ProjectA` - Declares namespace using dot notation
  - Logical grouping, not physical folder structure
- Nested syntax creates hierarchy
- `namespace MyCompany.ProjectA.Utilities;` - File-scoped namespace (C# 10+)
  - Semicolon instead of braces
  - Applies to entire file
  - Reduces indentation

### Namespace Best Practices

```csharp
// ✅ GOOD: Hierarchical organization
namespace CompanyName.ProductName.Feature.SubFeature
{
    // Classes related to this feature
}

// Examples:
namespace Microsoft.AspNetCore.Mvc.Rendering { }
namespace System.Collections.Generic { }
namespace MyCompany.ECommerce.ShoppingCart.Payment { }

// ✅ GOOD: Consistent naming
// Format: CompanyName.Technology.Feature
namespace Contoso.Web.Controllers { }
namespace Contoso.Data.Repositories { }

// ❌ BAD: Too short
namespace Utils { }
namespace Helpers { }

// ❌ BAD: Conflicts with .NET namespaces
namespace System.MyCode { }  // Don't extend System namespace!
```

---

## Using Directives

The `using` directive imports namespaces, so you don't need fully qualified names.

```csharp
// WITHOUT using directive
class Program
{
    static void Main()
    {
        System.Console.WriteLine("Hello");  // Fully qualified
        System.DateTime now = System.DateTime.Now;
        System.Collections.Generic.List<int> numbers = 
            new System.Collections.Generic.List<int>();
    }
}

// WITH using directives
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        Console.WriteLine("Hello");  // Short form
        DateTime now = DateTime.Now;
        List<int> numbers = new List<int>();
    }
}
```

### Using Directive Types

```csharp
// 1. NAMESPACE IMPORT
using System;
using System.Collections.Generic;

// 2. ALIAS FOR NAMESPACE
using Project =  MyCompany.ProjectManagement;
using Utilities = ThirdParty.CommonUtilities;

// 3. ALIAS FOR TYPE
using StringBuilder = System.Text.StringBuilder;
using Console = System.Console;

// 4. STATIC USING (C# 6+)
using static System.Console;  // Import static members
using static System.Math;

class Example
{
    static void Main()
    {
        // Using namespace alias
        Project.Task task = new Project.Task();
        
        // Using type alias
        StringBuilder sb = new StringBuilder();
        
        // Using static members directly
        WriteLine("Hello");      // Instead of Console.WriteLine
        double result = Sqrt(16);  // Instead of Math.Sqrt
        double pi = PI;            // Instead of Math.PI
        
        WriteLine($"Square root: {result}, Pi: {pi}");
    }
}
```

**Line-by-Line Explanation:**
- `using System;` - Imports entire System namespace
  - Makes all System types available without qualification
- `using Project = MyCompany.ProjectManagement;` - Creates alias for long namespace
  - Now use `Project.Task` instead of `MyCompany.ProjectManagement.Task`
- `using StringBuilder = System.Text.StringBuilder;` - Type alias
  - Shorthand for specific type
- `using static System.Console;` - Imports static members
  - Can call `WriteLine` instead of `Console.WriteLine`
  - Only works with static members

---

## Namespace Aliases

Aliases resolve naming conflicts and simplify long namespace names.

```csharp
using System;

// Two libraries with same class name
using WinForms = System.Windows.Forms;
using WPF = System.Windows;

namespace AliasDemo
{
    class Program
    {
        static void Main()
        {
            // Use aliases to distinguish
            WinForms.Button winButton = new WinForms.Button();
            // WPF.Button wpfButton = new WPF.Button();  // If WPF referenced
            
            Console.WriteLine("Using aliases to avoid conflicts");
        }
    }
}

// EXTERN ALIAS (Advanced - for version conflicts)
// In .csproj, you can specify aliases for assemblies
/*
<ItemGroup>
  <Reference Include="LibraryV1">
    <Aliases>V1</Aliases>
  </Reference>
  <Reference Include="LibraryV2">
    <Aliases>V2</Aliases>
  </Reference>
</ItemGroup>
*/

// Then in code:
extern alias V1;
extern alias V2;

class MultiVersionDemo
{
    static void Main()
    {
        V1::MyNamespace.MyClass class1 = new V1::MyNamespace.MyClass();
        V2::MyNamespace.MyClass class2 = new V2::MyNamespace.MyClass();
    }
}
```

---

## Global Using (C# 10+)

**Global usings** apply to all files in the project.

```csharp
// GlobalUsings.cs (convention: single file for global usings)
global using System;
global using System.Collections.Generic;
global using System.Linq;
global using System.Threading.Tasks;

// Now these namespaces are available in ALL files without explicit using
```

**Implicit Global Usings (SDK):**

In .csproj:
```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>
</Project>
```

Auto-includes common namespaces:
- System
- System.Collections.Generic
- System.IO
- System.Linq
- System.Net.Http
- System.Threading
- System.Threading.Tasks

```csharp
// With ImplicitUsings enabled, no need for:
// using System;
// using System.Collections.Generic;
// using System.Linq;

class Program
{
    static void Main()
    {
        // These work without explicit using directives!
        Console.WriteLine("Hello");
        List<int> numbers = new List<int> { 1, 2, 3 };
        int sum = numbers.Sum();
    }
}
```

---

## Assembly Versioning

### Version Number Format

```
Major.Minor.Build.Revision
1   . 2   . 3   . 4

Major: Breaking changes (incompatible API changes)
Minor: New features (backward compatible)
Build: Bug fixes, small changes
Revision: Builds, patches
```

### Setting Version

```xml
<!-- In .csproj -->
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <Version>1.2.3.4</Version>
    <AssemblyVersion>1.2.0.0</AssemblyVersion>
    <FileVersion>1.2.3.4</FileVersion>
  </PropertyGroup>
</Project>
```

- **AssemblyVersion**: Used by CLR for binding
- **FileVersion**: Displayed in file properties
- **Version**: NuGet package version

### Retrieving Version at Runtime

```csharp
using System;
using System.Reflection;

class VersionInfo
{
    static void Main()
    {
        Assembly assembly = Assembly.GetExecutingAssembly();
        Version version = assembly.GetName().Version;
        
        Console.WriteLine($"Version: {version}");
        Console.WriteLine($"Major: {version.Major}");
        Console.WriteLine($"Minor: {version.Minor}");
        Console.WriteLine($"Build: {version.Build}");
        Console.WriteLine($"Revision: {version.Revision}");
        
        // Attribute-based version
        var fileVersion = assembly.GetCustomAttribute<AssemblyFileVersionAttribute>();
        Console.WriteLine($"File Version: {fileVersion.Version}");
    }
}
```

---

## Strong Naming

**Strong naming** provides a unique identity for an assembly using a cryptographic signature.

### Why Strong Name?

- Guarantees uniqueness
- Protects against tampering
- Required for GAC installation
- Enables version side-by-side execution

### Creating Strong Name Key

```bash
# Generate key pair
sn -k MyKey.snk
```

```xml
<!-- In .csproj -->
<PropertyGroup>
  <SignAssembly>true</SignAssembly>
  <AssemblyOriginatorKeyFile>MyKey.snk</AssemblyOriginatorKeyFile>
</PropertyGroup>
```

### Strong Name Format

```
AssemblyName, Version=1.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089

- AssemblyName: Name of the assembly
- Version: Four-part version number
- Culture: Localization (neutral = no localization)
- PublicKeyToken: 8-byte hash of public key (from .snk file)
```

```csharp
using System;
using System.Reflection;

class StrongNameDemo
{
    static void Main()
    {
        // Check if assembly is strongly named
        Assembly assembly = Assembly.GetExecutingAssembly();
        byte[] publicKey = assembly.GetName().GetPublicKeyToken();
        
        if (publicKey != null && publicKey.Length > 0)
        {
            Console.WriteLine("Assembly is strongly named");
            Console.Write("Public Key Token: ");
            foreach (byte b in publicKey)
            {
                Console.Write($"{b:x2}");
            }
            Console.WriteLine();
        }
        else
        {
            Console.WriteLine("Assembly is NOT strongly named");
        }
    }
}
```

---

## Key Takeaways

### Critical Concepts

1. **Assemblies are Containers**
   - .exe for applications (has entry point)
   - .dll for libraries (no entry point)
   - Contains IL code, metadata, manifest, resources

2. **Namespaces Organize Code**
   - Prevent naming conflicts
   - Logical grouping (not physical folders)
   - Use hierarchical naming: Company.Product.Feature

3. **Using Directives Import Namespaces**
   - Avoid fully qualified names
   - Create aliases for conflicts
   - Use global usings (C# 10+) for common namespaces

4. **Versioning Matters**
   - Follow semantic versioning (Major.Minor.Patch)
   - AssemblyVersion for binding
   - FileVersion for display

5. **Strong Naming for Security**
   - Required for GAC
   - Guarantees assembly identity
   - Prevents tampering

### Best Practices

```csharp
// ✅ DO: Use meaningful namespace hierarchy
namespace MyCompany.ProductName.FeatureName
{
    public class ClassA { }
}

// ✅ DO: Match namespace to folder structure (for clarity)
// Folder: MyCompany/ProductName/FeatureName/File.cs
namespace MyCompany.ProductName.FeatureName { }

// ✅ DO: Use file-scoped namespaces (C# 10+)
namespace MyCompany.ProductName.FeatureName;

public class ClassA { }
// Less indentation!

// ✅ DO: Use global usings for common namespaces
// GlobalUsings.cs
global using System;
global using System.Collections.Generic;

// ✅ DO: Use using static for frequently used static classes
using static System.Console;
using static System.Math;

WriteLine($"Square root: {Sqrt(16)}");

// ❌ DON'T: Create too many nested namespaces
namespace A.B.C.D.E.F.G { }  // Too deep!

// ❌ DON'T: Use generic names
namespace Utilities { }  // Too vague
namespace Helpers { }    // What kind of helpers?

// ✅ BETTER:
namespace MyCompany.StringUtilities { }
namespace MyCompany.DateTimeHelpers { }
```

### Common Patterns

```csharp
// PATTERN 1: Namespace per layer
namespace MyApp.UI { }          // User interface
namespace MyApp.Business { }    // Business logic
namespace MyApp.Data { }        // Data access

// PATTERN 2: Namespace per feature
namespace MyApp.Authentication { }
namespace MyApp.Payment { }
namespace MyApp.Reporting { }

// PATTERN 3: Shared across features
namespace MyApp.Common { }      // Shared utilities
namespace MyApp.Models { }      // Shared data models
```

### Interview Questions

1. **What is an assembly?**
   - Compiled code library (.dll or .exe)
   - Contains IL code, metadata, manifest, resources
   - Fundamental unit of deployment in .NET

2. **What's the difference between .exe and .dll?**
   - .exe: Has entry point (Main), can be executed
   - .dll: Library, no entry point, referenced by other assemblies

3. **What is a namespace?**
   - Logical grouping of types
   - Prevents naming conflicts
   - Not related to physical folder structure

4. **What is the purpose of using directive?**
   - Import namespaces
   - Avoid fully qualified type names
   - Create aliases

5. **What is strong naming?**
   - Cryptographic signature for assembly
   - Provides unique identity
   - Required for GAC installation
   - Prevents tampering

6. **What is GAC?**
   - Global Assembly Cache
   - Repository for shared assemblies
   - Located in C:\\Windows\\Microsoft.NET\\assembly

7. **What's the difference between AssemblyVersion and FileVersion?**
   - AssemblyVersion: Used by CLR for binding
   - FileVersion: Displayed in file properties

---

**Next:** Structs Deep Dive - Learn about custom value types!
