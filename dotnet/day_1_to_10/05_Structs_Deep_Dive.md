# 05 - Structs Deep Dive

## Table of Contents
- [What is a Struct?](#what-is-a-struct)
- [Struct vs Class](#struct-vs-class)
- [Declaring Structs](#declaring-structs)
- [Struct Constructors](#struct-constructors)
- [Struct Methods and Properties](#struct-methods-and-properties)
- [Readonly Structs](#readonly-structs)
- [Ref Structs](#ref-structs)
- [Record Structs (C# 10+)](#record-structs-c-10)
- [When to Use Structs](#when-to-use-structs)
- [Key Takeaways](#key-takeaways)

---

## What is a Struct?

A **struct** (structure) is a value type that can encapsulate data and related functionality. Structs are similar to classes but with important differences.

### Basic Concept

```csharp
using System;

// SIMPLE STRUCT
struct Point
{
    public int X;
    public int Y;
}

class StructBasics
{
    static void Main()
    {
        // Create struct instance
        Point p1 = new Point();  // X=0, Y=0 (default values)
        p1.X = 10;
        p1.Y = 20;
        
        Console.WriteLine($"Point 1: ({p1.X}, {p1.Y})");
        
        // VALUE TYPE BEHAVIOR
        Point p2 = p1;  // COPY of p1, not reference!
        p2.X = 100;     // Changes only p2
        
        Console.WriteLine($"Point 1: ({p1.X}, {p1.Y})");  // Still (10, 20)
        Console.WriteLine($"Point 2: ({p2.X}, {p2.Y})");  // Now (100, 20)
    }
}
```

**Line-by-Line Explanation:**
- `struct Point` - Declares a value type structure
  - Unlike class, struct is a value type
  - Stored on stack (if local variable)
- `Point p1 = new Point();` - Creates instance with default values
  - new operator initializes fields to defaults
  - X and Y both become 0
- `Point p2 = p1;` - Creates complete COPY of p1
  - All fields copied
  - p2 is independent of p1
- `p2.X = 100;` - Modifies only p2's copy

**Memory Diagram:**
```
Stack:
┌────────────┐
│ p1         │
│  X: 10     │
│  Y: 20     │
├────────────┤
│ p2         │
│  X: 100    │  ← Different memory location
│  Y: 20     │
└────────────┘
```

---

## Struct vs Class

### Key Differences

| Feature | Struct | Class |
|---------|--------|-------|
| Type | Value type | Reference type |
| Storage | Stack (usually) | Heap |
| Default constructor | Auto-generated | Auto-generated |
| Parameterless constructor | Allowed (C# 10+) | Always allowed |
| Inheritance | Cannot inherit | Can inherit |
| Can be inherited | No | Yes |
| Null | Cannot be null* | Can be null |
| Assignment | Copies value | Copies reference |
| Performance | Better for small data | Better for large data |
| Default | All fields initialized to 0/null/false | Fields not initialized |

*Unless declared as nullable: `Point?`

```csharp
using System;

// STRUCT - Value type
struct StructPoint
{
    public int X;
    public int Y;
    
    public StructPoint(int x, int y)
    {
        X = x;
        Y = y;
    }
}

// CLASS - Reference type
class ClassPoint
{
    public int X;
    public int Y;
    
    public ClassPoint(int x, int y)
    {
        X = x;
        Y = y;
    }
}

class Comparison
{
    static void Main()
    {
        // STRUCT BEHAVIOR
        StructPoint s1 = new StructPoint(10, 20);
        StructPoint s2 = s1;  // COPY
        s2.X = 100;
        
        Console.WriteLine("STRUCT:");
        Console.WriteLine($"s1.X = {s1.X}");  // 10 (unchanged)
        Console.WriteLine($"s2.X = {s2.X}");  // 100
        
        // CLASS BEHAVIOR
        ClassPoint c1 = new ClassPoint(10, 20);
        ClassPoint c2 = c1;  // REFERENCE COPY
        c2.X = 100;
        
        Console.WriteLine("\\nCLASS:");
        Console.WriteLine($"c1.X = {c1.X}");  // 100 (changed!)
        Console.WriteLine($"c2.X = {c2.X}");  // 100
        
        // NULLABILITY
        StructPoint? nullableStruct = null;  // Nullable value type
        ClassPoint nullableClass = null;     // Reference can be null
        
        Console.WriteLine($"\\nNullable struct: {nullableStruct == null}");  // True
        Console.WriteLine($"Nullable class: {nullableClass == null}");        // True
        
        // SIZE MATTERS
        Console.WriteLine($"\\nStruct size: {System.Runtime.InteropServices.Marshal.SizeOf(s1)} bytes");
        Console.WriteLine("Class: Reference size (4 or 8 bytes) + object on heap");
    }
}
```

**Execution Flow:**
1. s1 created on stack with X=10, Y=20
2. s2 receives complete copy of s1's data
3. Modifying s2 doesn't affect s1 (separate copies)
4. c1 created on heap, reference stored on stack
5. c2 receives copy of reference (both point to same object)
6. Modifying through c2 affects the shared object
7. c1 sees the change because it's the same object

---

## Declaring Structs

### Basic Declaration

```csharp
using System;

// STRUCT WITH FIELDS
struct Rectangle
{
    public double Width;
    public double Height;
    
    // METHOD
    public double CalculateArea()
    {
        return Width * Height;
    }
    
    // PROPERTY
    public double Perimeter => 2 * (Width + Height);
}

// STRUCT WITH PROPERTIES (Recommended)
struct Product
{
    // Auto-properties
    public string Name { get; set; }
    public decimal Price { get; set; }
    public int Stock { get; set; }
    
    // Computed property
    public decimal TaxAmount => Price * 0.08m;
    public decimal TotalPrice => Price + TaxAmount;
    
    // Method
    public void Display()
    {
        Console.WriteLine($"Product: {Name}");
        Console.WriteLine($"Price: ${Price:F2}");
        Console.WriteLine($"Stock: {Stock}");
        Console.WriteLine($"Total with tax: ${TotalPrice:F2}");
    }
}

class StructDeclaration
{
    static void Main()
    {
        // Using Rectangle
        Rectangle rect = new Rectangle();
        rect.Width = 10.5;
        rect.Height = 5.0;
        
        Console.WriteLine($"Area: {rect.CalculateArea()}");
        Console.WriteLine($"Perimeter: {rect.Perimeter}");
        
        // Using Product
        Product product = new Product
        {
            Name = "Laptop",
            Price = 999.99m,
            Stock = 50
        };
        
        product.Display();
    }
}
```

**Line-by-Line Explanation:**
- `public double Width;` - Public field, directly accessible
- `public double Perimeter => 2 * (Width + Height);` - Expression-bodied property
  - Computed on each access
  - Read-only (no set accessor)
- `public string Name { get; set; }` - Auto-property
  - Compiler generates backing field
- `new Product { Name = "Laptop", ... }` - Object initializer syntax
  - Sets properties after construction

---

## Struct Constructors

### Parameterized Constructors

```csharp
using System;

struct Point3D
{
    public int X;
    public int Y;
    public int Z;
    
    // PARAMETERIZED CONSTRUCTOR
    public Point3D(int x, int y, int z)
    {
        X = x;  // Must initialize ALL fields
        Y = y;
        Z = z;
    }
    
    // CONSTRUCTOR OVERLOADING
    public Point3D(int value) : this(value, value, value)
    {
        // Calls other constructor
    }
    
    public override string ToString()
    {
        return $"({X}, {Y}, {Z})";
    }
}

// PARAMETERLESS CONSTRUCTOR (C# 10+)
struct Temperature
{
    public double Celsius { get; set; }
    
    // Parameterless constructor allowed in C# 10+
    public Temperature()
    {
        Celsius = 25.0;  // Default room temperature
    }
    
    public Temperature(double celsius)
    {
        Celsius = celsius;
    }
    
    public double Fahrenheit => (Celsius * 9 / 5) + 32;
}

class ConstructorDemo
{
    static void Main()
    {
        // Using parameterized constructor
        Point3D p1 = new Point3D(1, 2, 3);
        Console.WriteLine($"p1: {p1}");
        
        // Using overloaded constructor
        Point3D p2 = new Point3D(5);  // All coordinates = 5
        Console.WriteLine($"p2: {p2}");
        
        // Default constructor (all zeros)
        Point3D p3 = new Point3D();
        Console.WriteLine($"p3 (default): {p3}");
        
        // Parameterless constructor (C# 10+)
        Temperature t1 = new Temperature();  // 25°C
        Console.WriteLine($"Default temperature: {t1.Celsius}°C = {t1.Fahrenheit:F1}°F");
        
        Temperature t2 = new Temperature(100);
        Console.WriteLine($"Boiling point: {t2.Celsius}°C = {t2.Fahrenheit:F1}°F");
        
        // Without 'new' - default values
        Temperature t3;
        t3.Celsius = 0;
        Console.WriteLine($"Freezing point: {t3.Celsius}°C");
    }
}
```

**Line-by-Line Explanation:**
- `public Point3D(int x, int y, int z)` - Constructor must initialize ALL fields
  - Compiler enforces complete initialization
- `: this(value, value, value)` - Constructor chaining
  - Calls another constructor before executing body
- `public Temperature()` - Parameterless constructor (C# 10+)
  - Before C# 10: Not allowed in structs
  - Now allowed: Can set custom defaults
- `Temperature t3;` - Declaration without new
  - Uses default values (0 for numbers, null for references)
  - Fields must be assigned before use

**Constructor Rules:**
1. Must initialize all fields (before C# 10)
2. Can chain to other constructors with `: this(...)`
3. Cannot chain to base (structs don't inherit)
4. Parameterless constructor allowed (C# 10+)

---

## Struct Methods and Properties

```csharp
using System;

struct Vector2D
{
    // Fields
    private double x;
    private double y;
    
    // Properties
    public double X
    {
        get => x;
        set => x = value;
    }
    
    public double Y
    {
        get => y;
        set => y = value;
    }
    
    // Constructor
    public Vector2D(double x, double y)
    {
        this.x = x;
        this.y = y;
    }
    
    // INSTANCE METHODS
    
    public double Magnitude()
    {
        return Math.Sqrt(x * x + y * y);
    }
    
    public Vector2D Normalize()
    {
        double mag = Magnitude();
        return new Vector2D(x / mag, y / mag);
    }
    
    public double DotProduct(Vector2D other)
    {
        return this.x * other.x + this.y * other.y;
    }
    
    // STATIC METHODS
    
    public static Vector2D Add(Vector2D a, Vector2D b)
    {
        return new Vector2D(a.x + b.x, a.y + b.y);
    }
    
    public static double Distance(Vector2D a, Vector2D b)
    {
        double dx = a.x - b.x;
        double dy = a.y - b.y;
        return Math.Sqrt(dx * dx + dy * dy);
    }
    
    // OPERATOR OVERLOADING
    
    public static Vector2D operator +(Vector2D a, Vector2D b)
    {
        return new Vector2D(a.x + b.x, a.y + b.y);
    }
    
    public static Vector2D operator -(Vector2D a, Vector2D b)
    {
        return new Vector2D(a.x - b.x, a.y - b.y);
    }
    
    public static Vector2D operator *(Vector2D v, double scalar)
    {
        return new Vector2D(v.x * scalar, v.y * scalar);
    }
    
    // OVERRIDE OBJECT METHODS
    
    public override string ToString()
    {
        return $"({x:F2}, {y:F2})";
    }
    
    public override bool Equals(object obj)
    {
        if (obj is Vector2D other)
        {
            return this.x == other.x && this.y == other.y;
        }
        return false;
    }
    
    public override int GetHashCode()
    {
        return HashCode.Combine(x, y);
    }
}

class MethodsDemo
{
    static void Main()
    {
        Vector2D v1 = new Vector2D(3, 4);
        Vector2D v2 = new Vector2D(1, 2);
        
        // Instance methods
        Console.WriteLine($"v1: {v1}");
        Console.WriteLine($"Magnitude: {v1.Magnitude()}");
        Console.WriteLine($"Normalized: {v1.Normalize()}");
        Console.WriteLine($"Dot product: {v1.DotProduct(v2)}");
        
        // Static methods
        Vector2D sum = Vector2D.Add(v1, v2);
        double dist = Vector2D.Distance(v1, v2);
        
        Console.WriteLine($"\\nSum: {sum}");
        Console.WriteLine($"Distance: {dist:F2}");
        
        // Operator overloading
        Vector2D v3 = v1 + v2;      // Using + operator
        Vector2D v4 = v1 - v2;      // Using - operator
        Vector2D v5 = v1 * 2.5;     // Using * operator
        
        Console.WriteLine($"\\nv1 + v2 = {v3}");
        Console.WriteLine($"v1 - v2 = {v4}");
        Console.WriteLine($"v1 * 2.5 = {v5}");
        
        // Equality
        Vector2D v6 = new Vector2D(3, 4);
        Console.WriteLine($"\\nv1 == v6: {v1.Equals(v6)}");
    }
}
```

**Line-by-Line Explanation:**
- `public double Magnitude()` - Instance method computes magnitude
  - Uses Math.Sqrt for square root
  - Accesses instance fields x and y
- `public Vector2D Normalize()` - Returns NEW Vector2D (structs are values)
  - Doesn't modify original
  - Creates unit vector (magnitude = 1)
- `public static Vector2D Add(Vector2D a, Vector2D b)` - Static method
  - Doesn't access instance members
  - Called on type: `Vector2D.Add(...)`
- `public static Vector2D operator +(...)` - Operator overloading
  - Defines + operator for Vector2D
  - Must be static
  - Takes two parameters
- `override bool Equals(object obj)` - Overrides Object.Equals
  - Pattern matching: `obj is Vector2D other`
  - Value equality check
- `HashCode.Combine(x, y)` - Generates hash code from multiple values
  - Required when overriding Equals

---

## Readonly Structs

**Readonly structs** are immutable - fields cannot be modified after construction.

```csharp
using System;

// READONLY STRUCT - Immutable
readonly struct ImmutablePoint
{
    public int X { get; }  // Readonly property
    public int Y { get; }
    
    public ImmutablePoint(int x, int y)
    {
        X = x;
        Y = y;
    }
    
    // Methods can only read, not modify
    public double DistanceFrom Origin()
    {
        return Math.Sqrt(X * X + Y * Y);
    }
    
    // Return new instance instead of modifying
    public ImmutablePoint Move(int dx, int dy)
    {
        return new ImmutablePoint(X + dx, Y + dy);
    }
    
    public override string ToString() => $"({X}, {Y})";
}

// MUTABLE STRUCT - For comparison
struct MutablePoint
{
    public int X { get; set; }
    public int Y { get; set; }
    
    public void Move(int dx, int dy)
    {
        X += dx;  // Modifies this instance
        Y += dy;
    }
}

class ReadonlyStructDemo
{
    static void Main()
    {
        // READONLY STRUCT
        ImmutablePoint p1 = new ImmutablePoint(10, 20);
        Console.WriteLine($"Original: {p1}");
        
        // Cannot modify
        // p1.X = 30;  // ERROR: Property is read-only
        
        // Create new instance instead
        ImmutablePoint p2 = p1.Move(5, 10);
        Console.WriteLine($"After move: {p2}");
        Console.WriteLine($"Original unchanged: {p1}");
        
        // MUTABLE STRUCT
        MutablePoint m1 = new MutablePoint { X = 10, Y = 20 };
        Console.WriteLine($"\\nMutable original: ({m1.X}, {m1.Y})");
        
        m1.Move(5, 10);  // Modifies m1
        Console.WriteLine($"After move: ({m1.X}, {m1.Y})");
        
        // PERFORMANCE BENEFIT
        // Compiler can optimize readonly structs better
    }
}
```

**Line-by-Line Explanation:**
- `readonly struct ImmutablePoint` - Entire struct is readonly
  - All fields must be readonly
  - Methods cannot modify state
  - Compiler enforces immutability
- `public int X { get; }` - Readonly property (no setter)
  - Can only be set in constructor
- `public ImmutablePoint Move(int dx, int dy)` - Returns new instance
  - Doesn't modify this
  - Functional programming style
- `readonly` keyword benefits:
  - Thread-safe by design
  - Better compiler optimizations
  - Clear intent: value doesn't change

---

## Ref Structs

**Ref structs** can only exist on the stack (never on heap).

```csharp
using System;

// REF STRUCT - Stack-only
ref struct StackOnlyBuffer
{
    public Span<byte> Buffer;
    
    public StackOnlyBuffer(Span<byte> buffer)
    {
        Buffer = buffer;
    }
    
    public void Fill(byte value)
    {
        Buffer.Fill(value);
    }
}

class RefStructDemo
{
    static void Main()
    {
        // Create buffer on stack
        Span<byte> stackBuffer = stackalloc byte[100];
        
        StackOnlyBuffer buffer = new StackOnlyBuffer(stackBuffer);
        buffer.Fill(42);
        
        Console.WriteLine($"First byte: {stackBuffer[0]}");
        Console.WriteLine($"Last byte: {stackBuffer[99]}");
        
        // REF STRUCT RESTRICTIONS:
        // - Cannot be boxed
        // - Cannot be array element
        // - Cannot be field of non-ref struct
        // - Cannot be captured by lambda
        // - Cannot be used in async methods
        
        // StackOnlyBuffer[] array = new StackOnlyBuffer[10];  // ERROR!
        // object obj = buffer;  // ERROR: Cannot box ref struct
    }
}
```

**Ref Struct Rules:**
- Only on stack, never heap
- Cannot be boxed
- Cannot be array elements
- Cannot be captured by closures
- Cannot be used in async methods
- Use for high-performance scenarios (Span<T>, Memory<T>)

---

## Record Structs (C# 10+)

**Record structs** provide value-based equality and convenient syntax.

```csharp
using System;

// RECORD STRUCT - Value-based equality
record struct Person(string Name, int Age);

// READONLY RECORD STRUCT - Immutable
readonly record struct ImmutablePerson(string Name, int Age);

// REGULAR RECORD STRUCT WITH BODY
record struct Employee
{
    public string Name { get; init; }
    public string Department { get; init; }
    public decimal Salary { get; init; }
    
    // Constructor
    public Employee(string name, string department, decimal salary)
    {
        Name = name;
        Department = department;
        Salary = salary;
    }
    
    // Custom method
    public decimal AnnualBonus() => Salary * 0.1m;
}

class RecordStructDemo
{
    static void Main()
    {
        // POSITIONAL RECORD STRUCT
        Person p1 = new Person("John", 30);
        Person p2 = new Person("John", 30);
        Person p3 = new Person("Jane", 25);
        
        Console.WriteLine($"p1: {p1}");  // ToString auto-generated
        
        // Value-based equality
        Console.WriteLine($"p1 == p2: {p1 == p2}");  // True
        Console.WriteLine($"p1 == p3: {p1 == p3}");  // False
        
        // WITH expression - non-destructive mutation
        Person p4 = p1 with { Age = 31 };
        Console.WriteLine($"p1: {p1}");  // Unchanged
        Console.WriteLine($"p4: {p4}");  // Age changed
        
        // READONLY RECORD STRUCT
        ImmutablePerson ip1 = new ImmutablePerson("Alice", 28);
        // ip1.Age = 29;  // ERROR: init-only property
        
        ImmutablePerson ip2 = ip1 with { Age = 29 };  // Create new
        Console.WriteLine($"\\nip1: {ip1}");
        Console.WriteLine($"ip2: {ip2}");
        
        // RECORD STRUCT WITH BODY
        Employee emp = new Employee("Bob", "IT", 75000);
        Console.WriteLine($"\\nEmployee: {emp}");
        Console.WriteLine($"Bonus: ${emp.AnnualBonus():F2}");
        
        // Deconstruction
        var (name, age) = p1;
        Console.WriteLine($"\\nDeconstructed: {name}, {age}");
    }
}
```

**Line-by-Line Explanation:**
- `record struct Person(string Name, int Age);` - Positional record struct
  - Compiler generates constructor, properties, ToString, Equals, GetHashCode
  - Properties are init-only by default
- `p1 == p2` - Value-based equality (not reference!)
  - Compares all properties
  - Auto-generated by compiler
- `p1 with { Age = 31 }` - With expression
  - Creates new instance with modified property
  - Original unchanged (non-destructive mutation)
- `readonly record struct` - Immutable record struct
  - All properties are readonly
  - Thread-safe
- `var (name, age) = p1;` - Deconstruction
  - Extracts positional parameters

**Record Struct vs Regular Struct:**
- Record: Value equality, ToString, With expressions, Deconstruction
- Regular: Reference equality (unless overridden), manual ToString

---

## When to Use Structs

### Decision Guide

```csharp
// ✅ USE STRUCT when:
// 1. Small size (< 16 bytes recommended)
// 2. Logically represents a single value
// 3. Immutable
// 4. Short-lived
// 5. Value semantics

struct Point { public int X, Y; }  // Good: 8 bytes
struct Color { public byte R, G, B, A; }  // Good: 4 bytes
struct Complex { public double Real, Imaginary; }  // Good: 16 bytes

// ❌ DON'T USE STRUCT when:
// 1. Large size (> 16 bytes)
// 2. Frequently boxed
// 3. Needs inheritance
// 4. Mutable with frequent updates

// BAD - Too large
struct LargeData
{
    public string Name;      // 8 bytes (reference)
    public string Address;   // 8 bytes (reference)
    public int[] Numbers;    // 8 bytes (reference)
    public double[] Values;  // 8 bytes (reference)
}  // 32 bytes + heap allocations = use class instead!
```

### Built-in .NET Structs

```csharp
// EXAMPLES OF .NET STRUCTS

// Numeric types
int number = 42;              // Int32 struct
double price = 19.99;         // Double struct
decimal money = 99.99m;       // Decimal struct

// DateTime
DateTime now = DateTime.Now;
TimeSpan duration = TimeSpan.FromHours(2);

// Guid
Guid id = Guid.NewGuid();

// Nullable
int? nullableInt = null;      // Nullable<int> struct

// Span and Memory (ref structs)
Span<int> span = stackalloc int[100];
```

### Performance Comparison

```csharp
using System;
using System.Diagnostics;

struct StructPoint
{
    public int X, Y;
    public StructPoint(int x, int y) { X = x; Y = y; }
}

class ClassPoint
{
    public int X, Y;
    public ClassPoint(int x, int y) { X = x; Y = y; }
}

class PerformanceTest
{
    static void Main()
    {
        const int iterations = 10_000_000;
        Stopwatch sw = new Stopwatch();
        
        // STRUCT ALLOCATION
        sw.Start();
        for (int i = 0; i < iterations; i++)
        {
            StructPoint p = new StructPoint(i, i);
        }
        sw.Stop();
        Console.WriteLine($"Struct: {sw.ElapsedMilliseconds}ms");
        
        // CLASS ALLOCATION
        sw.Restart();
        for (int i = 0; i < iterations; i++)
        {
            ClassPoint p = new ClassPoint(i, i);
        }
        sw.Stop();
        Console.WriteLine($"Class: {sw.ElapsedMilliseconds}ms");
        
        // Struct is MUCH faster for small, short-lived objects
        // No heap allocation, no GC pressure
    }
}
```

---

## Key Takeaways

### Critical Concepts

1. **Structs are Value Types**
   - Stored on stack (usually)
   - Copy entire value on assignment
   - No inheritance

2. **Small and Simple**
   - Best for < 16 bytes
   - Represent single logical value
   - Examples: Point, Color, Complex number

3. **Immutability Recommended**
   - Use readonly struct
   - Prevents unexpected mutations
   - Better performance

4. **Modern Features**
   - Record structs (C# 10+)
   - Ref structs (stack-only)
   - Parameterless constructors (C# 10+)

### Best Practices

```csharp
// ✅ GOOD: Small, immutable, single value
readonly struct Money
{
    public decimal Amount { get; }
    public string Currency { get; }
    
    public Money(decimal amount, string currency)
    {
        Amount = amount;
        Currency = currency;
    }
}

// ✅ GOOD: Using record struct
record struct Temperature(double Celsius)
{
    public double Fahrenheit => (Celsius * 9 / 5) + 32;
}

// ❌ BAD: Large, mutable struct
struct BadExample  // Use class instead!
{
    public string Name;
    public string Address;
    public int[] Data;
    
    public void ChangeName(string newName)
    {
        Name = newName;  // Mutable!
    }
}

// ✅ BETTER: Use class for complex mutable types
class GoodExample
{
    public string Name { get; set; }
    public string Address { get; set; }
    public List<int> Data { get; set; }
}
```

### Common Mistakes

```csharp
// ❌ MISTAKE: Forgetting value semantics
struct Point
{
    public int X, Y;
}

void ModifyPoint(Point p)
{
    p.X = 100;  // Modifies COPY, not original!
}

Point original = new Point { X = 10, Y = 20 };
ModifyPoint(original);
// original.X is still 10!

// ✅ FIX: Use ref parameter
void ModifyPoint(ref Point p)
{
    p.X = 100;  // Modifies original
}

ModifyPoint(ref original);
// Now original.X is 100
```

### Interview Questions

1. **What is a struct?**
   - Value type that can contain data and methods
   - Stored on stack (usually)
   - Cannot inherit from other types

2. **Struct vs Class?**
   - Struct: Value type, stack, no inheritance, copy on assignment
   - Class: Reference type, heap, inheritance, copy reference

3. **When to use struct vs class?**
   - Struct: Small (<16 bytes), immutable, represents single value
   - Class: Large, mutable, complex behavior, needs inheritance

4. **Can structs have constructors?**
   - Yes, parameterized constructors always allowed
   - Parameterless constructors allowed (C# 10+)
   - Must initialize all fields (before C# 10)

5. **Can structs inherit?**
   - No, cannot inherit from other structs or classes
   - All structs inherit from System.ValueType

6. **What is a readonly struct?**
   - Immutable struct
   - All fields readonly
   - Better performance, thread-safe

---

**Next:** Enums and Usage - Learn about enumeration types!
