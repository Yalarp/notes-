# 01 - C# Setup and Introduction

## Table of Contents
- [Why .NET?](#why-net)
- [Understanding the .NET Framework](#understanding-the-net-framework)
- [Setting Up Visual Studio Code for C#](#setting-up-visual-studio-code-for-c)
- [Creating Your First C# Application](#creating-your-first-c-application)
- [Understanding the .NET Ecosystem](#understanding-the-net-ecosystem)
- [Key Takeaways](#key-takeaways)

---

## Why .NET?

### Introduction to .NET

**.NET** is a free, open-source, cross-platform framework developed by Microsoft for building modern applications. It provides a comprehensive and consistent programming model for building applications that have visually stunning user interfaces, secure communication, and the ability to model complex business processes.

### Why Choose .NET?

#### 1. **Cross-Platform Development**
- **Write Once, Run Anywhere**: .NET applications can run on Windows, Linux, and macOS
- **Unified Development Experience**: Same tools and libraries across all platforms
- **Docker Support**: Easy containerization for deployment

```csharp
// Example: A simple console app that runs on any platform
using System;

namespace CrossPlatformDemo
{
    class Program
    {
        static void Main(string[] args)
        {
            // This will work on Windows, Linux, and macOS
            Console.WriteLine($"Running on: {Environment.OSVersion}");
            Console.WriteLine($"Framework: {Environment.Version}");
        }
    }
}
```

**Line-by-Line Explanation:**
- `using System;` - Imports the System namespace which contains fundamental classes like Console
- `namespace CrossPlatformDemo` - Defines a namespace to organize code and prevent naming conflicts
- `class Program` - Declares the main class that contains the entry point
- `static void Main(string[] args)` - Entry point of the application; static means it belongs to the class itself
- `Console.WriteLine(...)` - Outputs text to the console window
- `Environment.OSVersion` - Gets information about the operating system
- `Environment.Version` - Gets the .NET runtime version

**Execution Flow:**
1. CLR (Common Language Runtime) loads the assembly
2. Finds the Main method as the entry point
3. Executes WriteLine statements sequentially
4. Displays OS and framework information
5. Application terminates

#### 2. **Rich Ecosystem and Libraries**
- **NuGet Package Manager**: Access to over 300,000 packages
- **Built-in Libraries**: Comprehensive standard library (BCL - Base Class Library)
- **Third-Party Support**: Extensive community and commercial libraries

#### 3. **Language Versatility**
- **Multiple Languages**: C#, F#, Visual Basic
- **Interoperability**: Different .NET languages can work together
- **Modern Language Features**: Async/await, LINQ, pattern matching, etc.

#### 4. **Performance**
- **JIT Compilation**: Just-In-Time compilation for optimized performance
- **Garbage Collection**: Automatic memory management
- **Native AOT**: Ahead-of-time compilation for even better performance

#### 5. **Productivity**
- **Rich IDE Support**: Visual Studio, VS Code, Rider
- **IntelliSense**: Code completion and suggestions
- **Refactoring Tools**: Built-in code improvement tools
- **Debugging**: Powerful debugging capabilities

#### 6. **Enterprise-Ready**
- **Security**: Built-in security features
- **Scalability**: Proven in large-scale applications
- **Support**: Professional support from Microsoft
- **Long-term Support (LTS)**: Stable versions with extended support

### .NET vs Other Frameworks

| Feature | .NET | Java | Python | Node.js |
|---------|------|------|--------|---------|
| Performance | Excellent | Excellent | Good | Good |
| Cross-Platform | Yes | Yes | Yes | Yes |
| Type System | Strong, Static | Strong, Static | Dynamic | Dynamic |
| Memory Management | Automatic (GC) | Automatic (GC) | Automatic (GC) | Automatic (GC) |
| Enterprise Support | Excellent | Excellent | Good | Growing |
| Learning Curve | Moderate | Moderate | Easy | Easy |
| Desktop Apps | Yes (WPF, WinForms) | Yes (Swing, JavaFX) | Limited | Limited |
| Web Development | Yes (ASP.NET) | Yes (Spring, Jakarta EE) | Yes (Django, Flask) | Yes (Express) |
| Mobile Apps | Yes (MAUI, Xamarin) | Yes (Android) | Limited | Limited |

---

## Understanding the .NET Framework

### What is a Framework?

A **framework** is a pre-built foundation that provides:
- **Reusable Code**: Common functionality you don't need to write from scratch
- **Structure**: Guidelines on how to organize your application
- **Tools**: Utilities for development, testing, and deployment
- **Runtime Environment**: Where your application executes

Think of it like building a house:
- **Without Framework**: You cut your own lumber, make your own nails, design everything from scratch
- **With Framework**: You have pre-made walls, standardized materials, and architectural patterns to follow

### .NET Framework Architecture

```
┌─────────────────────────────────────────────────────────┐
│                  Your Application                        │
│         (Console, Web, Desktop, Mobile, etc.)           │
└─────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────┐
│              Framework Class Library (FCL)               │
│  Collections │ I/O │ LINQ │ Async │ Networking │ etc.  │
└─────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────┐
│            Common Language Runtime (CLR)                 │
│   ┌──────────┬──────────┬──────────┬──────────┐       │
│   │   JIT    │    GC    │ Security │Exception │       │
│   │Compiler  │          │ Manager  │ Handler  │       │
│   └──────────┴──────────┴──────────┴──────────┘       │
└─────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────┐
│              Operating System (Windows/Linux/macOS)      │
└─────────────────────────────────────────────────────────┘
```

### Key Components

#### 1. **Common Language Runtime (CLR)**

The CLR is the execution engine for .NET applications. It provides:

**A. Memory Management**
```csharp
// Automatic memory management example
using System;

class MemoryDemo
{
    static void Main()
    {
        // Object created on the heap
        Person person = new Person("John", 30);
        
        // Use the object
        person.DisplayInfo();
        
        // No need to manually free memory!
        // GC will automatically collect it when no longer referenced
    }
}

class Person
{
    private string name;  // Reference type - stored on heap
    private int age;      // Value type - stored with object on heap
    
    public Person(string name, int age)
    {
        this.name = name;  // 'this' refers to the current instance
        this.age = age;
    }
    
    public void DisplayInfo()
    {
        Console.WriteLine($"Name: {name}, Age: {age}");
    }
}
```

**Line-by-Line Explanation:**
- `Person person = new Person("John", 30);` - Creates a new Person object on the managed heap
  - `new` keyword allocates memory
  - Constructor initializes the object
  - Reference is stored in the 'person' variable
- `person.DisplayInfo();` - Calls instance method using the reference
- `private string name;` - Instance field, private means accessible only within class
- `this.name = name;` - 'this' disambiguates between parameter and field
- `$"Name: {name}, Age: {age}"` - String interpolation (C# 6.0+), inserts variable values

**Memory Layout:**
```
Stack:                  Heap:
┌────────┐             ┌──────────────┐
│ person │───────────→ │ Person Object│
│ (ref)  │             │ name: "John" │
└────────┘             │ age: 30      │
                       └──────────────┘
```

**B. Just-In-Time (JIT) Compilation**

Your C# code goes through multiple stages:

```
C# Source Code (.cs)
        ↓
    [Compiler (csc.exe)]
        ↓
Intermediate Language (IL) Code (.dll or .exe)
        ↓
    [CLR - JIT Compiler]
        ↓
Native Machine Code
        ↓
    [Execution]
```

```csharp
// This C# code...
public int Add(int a, int b)
{
    return a + b;
}

// Gets compiled to IL (Intermediate Language)
// IL Code (simplified representation):
// .method public hidebysig instance int32 Add(int32 a, int32 b)
// {
//     ldarg.1      // Load first argument
//     ldarg.2      // Load second argument
//     add          // Add them
//     ret          // Return result
// }

// Then JIT compiles to native machine code at runtime
```

**C. Type Safety**
```csharp
// CLR enforces type safety
int number = 42;
string text = "Hello";

// This will cause a compile-time error:
// number = text;  // Error: Cannot convert string to int

// Safe casting
object obj = "Hello";
if (obj is string str)  // Pattern matching (C# 7+)
{
    Console.WriteLine($"Length: {str.Length}");
}
```

#### 2. **Framework Class Library (FCL)**

The FCL is a comprehensive collection of reusable classes, interfaces, and value types.

**Categories:**
- **Collections**: List, Dictionary, Queue, Stack, etc.
- **I/O**: File operations, streams, serialization
- **Networking**: HTTP, TCP, sockets
- **Data Access**: ADO.NET, Entity Framework
- **LINQ**: Language Integrated Query
- **Async/Await**: Asynchronous programming
- **Reflection**: Type inspection and dynamic invocation

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

class FCLDemo
{
    static void Main()
    {
        // Using Collections from FCL
        List<int> numbers = new List<int> { 1, 2, 3, 4, 5 };
        
        // Using LINQ from FCL
        var evenNumbers = numbers.Where(n => n % 2 == 0);
        
        // Using Console from FCL
        Console.WriteLine("Even numbers:");
        foreach (var num in evenNumbers)
        {
            Console.WriteLine(num);
        }
    }
}
```

**Line-by-Line Explanation:**
- `List<int> numbers = new List<int> { 1, 2, 3, 4, 5 };` - Creates a generic list of integers
  - `List<int>` - Generic collection that stores int values
  - `{ 1, 2, 3, 4, 5 }` - Collection initializer syntax
- `var evenNumbers = numbers.Where(n => n % 2 == 0);` - LINQ query
  - `var` - Implicitly typed variable (compiler infers IEnumerable<int>)
  - `Where` - LINQ extension method for filtering
  - `n => n % 2 == 0` - Lambda expression (takes n, returns true if even)
- `foreach (var num in evenNumbers)` - Iterates through the filtered collection
  - Each element is accessed one at a time
  - Type is inferred as int

**Execution Flow:**
1. List is created and populated with 5 integers
2. Where creates a deferred query (not executed immediately)
3. foreach triggers enumeration
4. For each number: checks condition, yields even numbers
5. Each even number is printed

### .NET Flavors

#### 1. **.NET Framework** (Legacy - Windows Only)
- Original .NET implementation
- Windows-only
- Versions 1.0 to 4.8
- Still widely used in enterprise applications

#### 2. **.NET Core** (Merged into .NET 5+)
- Cross-platform rewrite
- Versions 1.0 to 3.1
- Open-source
- Better performance than .NET Framework

#### 3. **.NET** (Modern - Unified Platform)
- Current version (started with .NET 5)
- Unifies .NET Core, .NET Framework, and Xamarin
- .NET 6, 7, 8 (LTS versions: 6 and 8)
- Recommended for all new development

```csharp
// Checking your .NET version
using System;

class VersionCheck
{
    static void Main()
    {
        // Get runtime version
        var version = Environment.Version;
        Console.WriteLine($".NET Runtime Version: {version}");
        
        // Get framework description
        var framework = System.Runtime.InteropServices.RuntimeInformation.FrameworkDescription;
        Console.WriteLine($"Framework: {framework}");
    }
}
```

---

## Setting Up Visual Studio Code for C#

### Step 1: Install Prerequisites

#### A. Install .NET SDK

**What is the SDK?**
- **SDK (Software Development Kit)**: Tools for developing applications
- Includes compiler, libraries, and runtime
- Different from runtime (which only runs applications)

**Installation:**
1. Visit: https://dotnet.microsoft.com/download
2. Download the latest .NET SDK
3. Run the installer
4. Verify installation:

```bash
# Command prompt/terminal
dotnet --version
# Output: 8.0.100 (or your installed version)

dotnet --info
# Shows detailed SDK and runtime information
```

#### B. Install Visual Studio Code

1. Visit: https://code.visualstudio.com/
2. Download for your operating system
3. Install with default settings

#### C. Install C# Extension

1. Open VS Code
2. Click Extensions icon (or Ctrl+Shift+X)
3. Search for "C# Dev Kit" or "C#"
4. Install the official Microsoft C# extension
5. Reload VS Code

**Key Features of C# Extension:**
- IntelliSense (code completion)
- Syntax highlighting
- Debugging support
- Code navigation
- Refactoring tools

### Step 2: Create Your First Project

```bash
# Create a new console application
dotnet new console -n MyFirstApp

# Navigate to project folder
cd MyFirstApp

# This creates:
# MyFirstApp/
#   ├── MyFirstApp.csproj  (Project file)
#   ├── Program.cs         (Main code file)
#   └── obj/               (Build artifacts)
```

**Understanding the Project File (MyFirstApp.csproj):**
```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net8.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>
</Project>
```

**Explanation:**
- `Sdk="Microsoft.NET.Sdk"` - Specifies the SDK type for console apps
- `OutputType>Exe` - Creates an executable (.exe)
- `TargetFramework>net8.0` - Target .NET version
- `Nullable>enable` - Enables nullable reference types (C# 8+)
- `ImplicitUsings>enable` - Auto-includes common namespaces

### Step 3: Run Your Application

```bash
# Build the application
dotnet build
# Compiles code, checks for errors, creates DLL/EXE

# Run the application
dotnet run
# Builds (if needed) and executes
```

---

## Creating Your First C# Application

### Example 1: Hello World (Classic)

```csharp
using System;

namespace HelloWorld
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Hello, World!");
        }
    }
}
```

**Detailed Breakdown:**

1. `using System;`
   - Imports the System namespace
   - Allows us to use classes like Console without fully qualifying them
   - Without this, we'd need to write `System.Console.WriteLine`

2. `namespace HelloWorld`
   - Defines a namespace to organize code
   - Prevents naming conflicts
   - Convention: Use project name or meaningful grouping

3. `class Program`
   - Defines a class named Program
   - Classes are blueprints for objects
   - Entry point class can have any name, but Program is conventional

4. `static void Main(string[] args)`
   - **static**: Belongs to the class, not an instance
   - **void**: Doesn't return a value
   - **Main**: Entry point method name (case-sensitive!)
   - **string[] args**: Command-line arguments array

5. `Console.WriteLine("Hello, World!");`
   - Console: Class from System namespace
   - WriteLine: Method that writes text and adds newline
   - `"Hello, World!"`: String literal parameter

**Execution Flow:**
```
1. CLR loads the assembly
2. Finds the Main method in startup class
3. Creates the args array (empty if no arguments)
4. Executes Main method
5. Outputs "Hello, World!" to console
6. Main returns (implicit return for void)
7. Application terminates
```

### Example 2: Hello World (Modern C# 10+)

```csharp
// Top-level statements (C# 9+)
Console.WriteLine("Hello, World!");
```

**What happened to all the ceremony?**
- Compiler automatically generates the class and Main method
- Perfect for simple programs and scripts
- Still compiles to the same IL code

**Generated Code (by compiler):**
```csharp
using System;

namespace MyApp
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Hello, World!");
        }
    }
}
```

### Example 3: Interactive Console Application

```csharp
using System;

namespace InteractiveDemo
{
    class Program
    {
        static void Main(string[] args)
        {
            // Display welcome message
            Console.WriteLine("=== C# Interactive Demo ===");
            Console.WriteLine();  // Empty line
            
            // Get user input
            Console.Write("Enter your name: ");
            string name = Console.ReadLine();
            
            Console.Write("Enter your age: ");
            string ageInput = Console.ReadLine();
            
            // Convert string to integer
            int age = int.Parse(ageInput);
            
            // Calculate year of birth
            int currentYear = DateTime.Now.Year;
            int birthYear = currentYear - age;
            
            // Display personalized message
            Console.WriteLine();
            Console.WriteLine($"Hello, {name}!");
            Console.WriteLine($"You were born around {birthYear}.");
            
            // Display with different colors
            Console.ForegroundColor = ConsoleColor.Green;
            Console.WriteLine("Welcome to C# programming!");
            Console.ResetColor();
            
            // Wait for user before closing
            Console.WriteLine();
            Console.WriteLine("Press any key to exit...");
            Console.ReadKey();
        }
    }
}
```

**Line-by-Line Explanation:**

- `Console.WriteLine("=== C# Interactive Demo ===");` - Displays header
- `Console.WriteLine();` - Prints empty line for spacing
- `Console.Write("Enter your name: ");` - Prints prompt without newline
- `string name = Console.ReadLine();` - Reads user input until Enter key
  - ReadLine() returns a string
  - Stores user input in 'name' variable
- `int age = int.Parse(ageInput);` - Converts string to integer
  - Parse() throws exception if conversion fails
  - Alternative: int.TryParse() for safer conversion
- `DateTime.Now.Year` - Gets current year from system date
- `int birthYear = currentYear - age;` - Simple arithmetic calculation
- `$"Hello, {name}!"` - String interpolation with embedded expression
- `Console.ForegroundColor = ConsoleColor.Green;` - Changes text color
- `Console.ResetColor();` - Restores default color
- `Console.ReadKey();` - Waits for any key press
  - Prevents console from closing immediately
  - Returns ConsoleKeyInfo object

**Execution Flow:**
```
1. Display header and blank line
2. Prompt user for name
3. Wait for input (blocking operation)
4. Store name in variable
5. Prompt user for age
6. Wait for input
7. Convert string to integer
8. Get current year from system
9. Calculate birth year
10. Display personalized messages
11. Change console color
12. Display welcome message
13. Reset color
14. Wait for key press
15. Exit application
```

**Output Example:**
```
=== C# Interactive Demo ===

Enter your name: John
Enter your age: 25

Hello, John!
You were born around 2024.
Welcome to C# programming!

Press any key to exit...
```

---

## Understanding the .NET Ecosystem

### Development Tools

#### 1. **Visual Studio** (Full IDE)
- Complete integrated development environment
- Windows and Mac versions
- Best for large enterprise projects
- Includes visual designers, profilers, testing tools

#### 2. **Visual Studio Code** (Lightweight Editor)
- Cross-platform
- Fast and lightweight
- Extensible with plugins
- Great for modern .NET development

#### 3. **JetBrains Rider** (Commercial IDE)
- Cross-platform
- Advanced refactoring
- Excellent performance
- Paid license required

### Application Types You Can Build

```csharp
// 1. Console Applications
Console.WriteLine("CLI tools, background services");

// 2. Web Applications (ASP.NET Core)
// APIs, MVC web apps, Blazor SPAs

// 3. Desktop Applications
// WPF (Windows Presentation Foundation)
// WinForms (Windows Forms)
// MAUI (Multi-platform App UI)

// 4. Mobile Applications
// MAUI, Xamarin

// 5. Cloud Applications
// Azure Functions, Azure App Service

// 6. Game Development
// Unity (uses C#)

// 7. IoT
// .NET IoT libraries for Raspberry Pi, etc.

// 8. Machine Learning
// ML.NET framework
```

### .NET CLI Commands Reference

```bash
# Create new projects
dotnet new console          # Console application
dotnet new classlib         # Class library
dotnet new web              # ASP.NET Core web app
dotnet new webapi           # ASP.NET Core Web API

# Build and run
dotnet build                # Compile project
dotnet run                  # Build and run
dotnet watch run            # Run with hot reload

# Testing
dotnet test                 # Run unit tests

# Package management
dotnet add package <name>   # Add NuGet package
dotnet restore              # Restore dependencies

# Publishing
dotnet publish              # Create deployment package
dotnet publish -c Release   # Optimized for production

# Information
dotnet --version            # SDK version
dotnet --info               # Detailed information
dotnet --list-sdks          # Installed SDKs
```

---

## Key Takeaways

### Essential Concepts

1. **. NET is a Framework**
   - Provides runtime (CLR) and libraries (FCL)
   - Cross-platform and open-source
   - Supports multiple languages (C#, F#, VB.NET)

2. **CLR Benefits**
   - Automatic memory management (garbage collection)
   - Type safety and security
   - JIT compilation for performance

3. **C# is Modern**
   - Strongly typed language
   - Object-oriented with functional features
   - Constant evolution with new versions

4. **Development is Streamlined**
   - .NET CLI for productivity
   - Rich tooling (VS Code, Visual Studio)
   - Comprehensive class libraries

### Best Practices

```csharp
// ✅ DO: Use meaningful names
string customerName = "John";
int orderCount = 5;

// ❌ DON'T: Use cryptic abbreviations
string cn = "John";
int oc = 5;

// ✅ DO: Follow naming conventions
public class CustomerService { }      // PascalCase for classes
public void ProcessOrder() { }        // PascalCase for methods
private string firstName;             // camelCase for fields

// ✅ DO: Use string interpolation
Console.WriteLine($"Customer: {customerName}, Orders: {orderCount}");

// ❌ DON'T: Use concatenation for complex strings
Console.WriteLine("Customer: " + customerName + ", Orders: " + orderCount);

// ✅ DO: Handle errors appropriately
try
{
    int age = int.Parse(input);
}
catch (FormatException)
{
    Console.WriteLine("Please enter a valid number");
}

// ✅ DO: Use using statements for disposables
using (var file = File.OpenRead("data.txt"))
{
    // File automatically closed when block exits
}
```

### Next Steps

- **Day 02**: Deep dive into data types and variables
- **Day 03**: Object-oriented programming fundamentals
- **Day 04**: Advanced OOP concepts
- **Day 05**: LINQ and modern C# features

### Interview Questions

1. **What is the difference between .NET Framework and .NET Core?**
   - .NET Framework: Windows-only, versions up to 4.8
   - .NET Core: Cross-platform, open-source, better performance
   - .NET 5+ unified both into a single platform

2. **What is the CLR?**
   - Common Language Runtime
   - Execution engine for .NET applications
   - Provides JIT compilation, garbage collection, type safety

3. **What is JIT compilation?**
   - Just-In-Time compilation
   - Converts IL code to native machine code at runtime
   - Optimizes for the specific hardware

4. **Explain the Main method signature.**
   - `static`: Can be called without creating an instance
   - `void`: Doesn't return a value (can also return int)
   - `string[] args`: Command-line arguments

5. **What are top-level statements?**
   - C# 9+ feature
   - Removes ceremony for simple programs
   - Compiler generates class and Main automatically

---

**Congratulations!** You now understand the fundamentals of C# and the .NET platform. In the next section, we'll explore data types in detail.
