# 06 - Enums and Usage

## Table of Contents
- [What is an Enum?](#what-is-an-enum)
- [Declaring Enums](#declaring-enums)
- [Enum Underlying Types](#enum-underlying-types)
- [Enum Methods and Conversion](#enum-methods-and-conversion)
- [Flags Enums](#flags-enums)
- [Enum Best Practices](#enum-best-practices)
- [Key Takeaways](#key-takeaways)

---

## What is an Enum?

An **enum** (enumeration) is a value type that represents a set of named constants. Enums make code more readable and maintainable by replacing magic numbers with meaningful names.

### Basic Concept

```csharp
using System;

// ENUM DECLARATION
enum DayOfWeek
{
    Sunday,      // 0
    Monday,      // 1
    Tuesday,     // 2
    Wednesday,   // 3
    Thursday,    // 4
    Friday,      // 5
    Saturday     // 6
}

class EnumBasics
{
    static void Main()
    {
        // DECLARING ENUM VARIABLES
        DayOfWeek today = DayOfWeek.Monday;
        DayOfWeek weekend = DayOfWeek.Saturday;
        
        Console.WriteLine($"Today is: {today}");
        Console.WriteLine($"Weekend day: {weekend}");
        
        // ENUM VALUES ARE INTEGERS
        Console.WriteLine($"Monday value: {(int)today}");
        Console.WriteLine($"Saturday value: {(int)weekend}");
        
        // USING IN CONDITIONAL STATEMENTS
        if (today == DayOfWeek.Monday)
        {
            Console.WriteLine("Start of work week!");
        }
        
        // SWITCH STATEMENT
        switch (today)
        {
            case DayOfWeek.Monday:
                Console.WriteLine("Back to work");
                break;
            case DayOfWeek.Friday:
                Console.WriteLine("Almost weekend!");
                break;
            case DayOfWeek.Saturday:
            case DayOfWeek.Sunday:
                Console.WriteLine("Weekend!");
                break;
            default:
                Console.WriteLine("Midweek");
                break;
        }
    }
}
```

**Line-by-Line Explanation:**
- `enum DayOfWeek` - Declares enumeration type
  - By default, first value is 0
  - Each subsequent value increments by 1
- `DayOfWeek today = DayOfWeek.Monday;` - Creates enum variable
  - Must use fully qualified name: `DayOfWeek.Monday`
  - Strongly typed (type-safe)
- `(int)today` - Cast enum to underlying integer value
  - Monday = 1 (second value)
- `today == DayOfWeek.Monday` - Enum comparison
  - Uses value comparison
  - Type-safe (cannot compare different enum types)

**Memory:**
```
Enum is stored as integer:
DayOfWeek.Monday → 1 (int32)
Size: 4 bytes (default int)
```

---

## Declaring Enums

### Basic Declaration

```csharp
using System;

// SIMPLE ENUM
enum Status
{
    Pending,     // 0
    Approved,    // 1
    Rejected     // 2
}

// ENUM WITH EXPLICIT VALUES
enum Priority
{
    Low = 1,
    Medium = 2,
    High = 3,
    Critical = 5   // Can skip numbers
}

// ENUM WITH MIXED VALUES
enum ErrorCode
{
    Success = 0,
    InvalidInput = 100,
    NotFound = 404,
    Unauthorized = 401,
    ServerError = 500
}

// ENUM IN NAMESPACE
namespace MyApp
{
    enum UserRole
    {
        Guest,
        User,
        Moderator,
        Admin
    }
}

class EnumDeclaration
{
    static void Main()
    {
        // Using enums
        Status orderStatus = Status.Pending;
        Priority taskPriority = Priority.High;
        ErrorCode error = ErrorCode.NotFound;
        
        Console.WriteLine($"Order: {orderStatus} ({(int)orderStatus})");
        Console.WriteLine($"Priority: {taskPriority} ({(int)taskPriority})");
        Console.WriteLine($"Error: {error} ({(int)error})");
        
        // Default value (0)
        Status defaultStatus = default(Status);
        Console.WriteLine($"Default: {defaultStatus} ({(int)defaultStatus})");
        
        // Using namespaced enum
        MyApp.UserRole role = MyApp.UserRole.Admin;
        Console.WriteLine($"Role: {role}");
    }
}
```

**Line-by-Line Explanation:**
- `enum Status` - Automatic numbering starts at 0
- `Low = 1` - Explicit value assignment
  - Subsequent values auto-increment from last defined value
- `Critical = 5` - Can skip numbers (1, 2, 3, 5)
- `InvalidInput = 100` - Can use any integer value
- `default(Status)` - Returns 0 (first value if defined, or default int)
- Enum members are static constants
  - Accessed via type name: `Status.Pending`

### Naming Conventions

```csharp
// ✅ GOOD: Singular name for enum
enum Color { Red, Green, Blue }
enum Direction { North, South, East, West }

// ❌ BAD: Plural name
enum Colors { Red, Green, Blue }  // Don't use plural

// ✅ GOOD: PascalCase for enum and members
enum FileAccess
{
    Read,
    Write,
    ReadWrite
}

// ❌ BAD: ALL_CAPS or other conventions
enum FileAccess
{
    READ,       // Wrong
    WRITE,      // Wrong
    read_write  // Wrong
}

// ✅ GOOD: Descriptive names
enum OrderStatus
{
    Pending,
    Processing,
    Shipped,
    Delivered,
    Cancelled
}

// ❌ BAD: Abbreviated or unclear
enum OrderStatus
{
    Pend,   // Unclear
    Proc,   // Unclear
    Shp,    // Unclear
}
```

---

## Enum Underlying Types

By default, enums use `int` (32-bit), but you can specify different integer types.

```csharp
using System;

// DEFAULT (INT)
enum DefaultEnum
{
    Value1,
    Value2
}  // Stored as int (4 bytes)

// BYTE (1 BYTE)
enum ByteEnum : byte
{
    Small = 0,
    Medium = 128,
    Large = 255  // Max for byte
}

// SHORT (2 BYTES)
enum ShortEnum : short
{
    Min = -32768,
    Zero = 0,
    Max = 32767
}

// LONG (8 BYTES)
enum LongEnum : long
{
    VeryLarge = 1000000000000L
}

// UNSIGNED TYPES
enum UIntEnum : uint
{
    Value = 4000000000  // Larger than int.MaxValue
}

class UnderlyingTypes
{
    static void Main()
    {
        Console.WriteLine("Enum Sizes:");
        Console.WriteLine($"DefaultEnum: {sizeof(DefaultEnum)} bytes");
        Console.WriteLine($"ByteEnum: {sizeof(ByteEnum)} byte");
        Console.WriteLine($"ShortEnum: {sizeof(ShortEnum)} bytes");
        Console.WriteLine($"LongEnum: {sizeof(LongEnum)} bytes");
        
        // GETTING UNDERLYING TYPE
        Type defaultType = Enum.GetUnderlyingType(typeof(DefaultEnum));
        Type byteType = Enum.GetUnderlyingType(typeof(ByteEnum));
        
        Console.WriteLine($"\\nDefaultEnum underlying type: {defaultType.Name}");
        Console.WriteLine($"ByteEnum underlying type: {byteType.Name}");
        
        // VALID UNDERLYING TYPES:
        // byte, sbyte, short, ushort, int, uint, long, ulong
        
        // INVALID:
        // enum FloatEnum : float { }  // ERROR!
        // enum StringEnum : string { }  // ERROR!
    }
}
```

**Line-by-Line Explanation:**
- `enum ByteEnum : byte` - Specifies byte as underlying type
  - Saves memory (1 byte vs 4 bytes)
  - Limited range: 0-255
- `sizeof(ByteEnum)` - Returns size in bytes
  - ByteEnum: 1, ShortEnum: 2, DefaultEnum: 4, LongEnum: 8
- `Enum.GetUnderlyingType(typeof(ByteEnum))` - Reflection method
  - Returns Type object representing underlying type
- Valid types: byte, sbyte, short, ushort, int, uint, long, ulong
  - Must be integral types
  - Cannot use float, double, decimal, string, etc.

**When to Use Different Types:**
```csharp
// Use byte when: Small set of values (0-255)
enum HttpMethod : byte
{
    Get, Post, Put, Delete, Patch
}

// Use int (default) for: General enums
enum Status { Pending, Approved, Rejected }

// Use long when: Need very large values
enum LargeIds : long
{
    Database1 = 10000000000000001,
    Database2 = 10000000000000002
}
```

---

## Enum Methods and Conversion

### String Conversion

```csharp
using System;

enum Color
{
    Red,
    Green,
    Blue
}

class EnumConversion
{
    static void Main()
    {
        Color color = Color.Green;
        
        // ENUM TO STRING
        string name1 = color.ToString();
        string name2 = Enum.GetName(typeof(Color), color);
        string name3 = nameof(Color.Green);  // Compile-time constant
        
        Console.WriteLine($"ToString: {name1}");
        Console.WriteLine($"GetName: {name2}");
        Console.WriteLine($"nameof: {name3}");
        
        // STRING TO ENUM (PARSE)
        string input = "Blue";
        Color parsed = (Color)Enum.Parse(typeof(Color), input);
        Console.WriteLine($"\\nParsed '{input}': {parsed}");
        
        // CASE-INSENSITIVE PARSE
        Color parsed2 = (Color)Enum.Parse(typeof(Color), "red", ignoreCase: true);
        Console.WriteLine($"Parsed 'red': {parsed2}");
        
        // TRYPARSE (SAFE)
        string userInput = "Yellow";
        if (Enum.TryParse<Color>(userInput, out Color result))
        {
            Console.WriteLine($"Parsed: {result}");
        }
        else
        {
            Console.WriteLine($"'{userInput}' is not a valid Color");
        }
        
        // ENUM TO INT
        int value = (int)color;
        Console.WriteLine($"\\nGreen value: {value}");
        
        // INT TO ENUM
        int number = 2;
        Color fromInt = (Color)number;
        Console.WriteLine($"Value 2: {fromInt}");
        
        // INVALID INT (NO ERROR!)
        Color invalid = (Color)999;  // No exception, but undefined!
        Console.WriteLine($"Invalid value 999: {invalid}");  // Prints "999"
        
        // CHECK IF DEFINED
        bool isDefined = Enum.IsDefined(typeof(Color), 999);
        Console.WriteLine($"Is 999 defined: {isDefined}");  // False
    }
}
```

**Line-by-Line Explanation:**
- `color.ToString()` - Instance method, returns enum member name as string
- `Enum.GetName(typeof(Color), color)` - Static method, same result
  - Requires Type parameter
- `nameof(Color.Green)` - Compile-time constant
  - Resolved at compilation
  - No runtime overhead
- `Enum.Parse(typeof(Color), input)` - Parses string to enum
  - Case-sensitive by default
  - Throws ArgumentException if not found
  - Returns object, requires cast
- `Enum.TryParse<Color>(userInput, out result)` - Safe parsing
  - Generic version (C# added later)
  - Returns bool (success/failure)
  - Out parameter receives result
  - No exception if invalid
- `(int)color` - Cast to underlying type
- `(Color)number` - Cast from int to enum
  - **WARNING**: No validation! Any int value accepted
- `Enum.IsDefined(typeof(Color), 999)` - Checks if value exists
  - Returns false for undefined values

**Execution Flow:**
1. Convert enum to string using ToString
2. Parse string back to enum (with validation)
3. TryParse safely handles invalid input
4. Convert between enum and int
5. Check if value is defined in enum

### Getting All Values

```csharp
using System;

enum Season
{
    Spring,
    Summer,
    Fall,
    Winter
}

class EnumIteration
{
    static void Main()
    {
        // GET ALL VALUES
        Season[] seasons = (Season[])Enum.GetValues(typeof(Season));
        
        Console.WriteLine("All seasons:");
        foreach (Season season in seasons)
        {
            Console.WriteLine($"- {season} ({(int)season})");
        }
        
        // GET ALL NAMES
        string[] names = Enum.GetNames(typeof(Season));
        
        Console.WriteLine($"\\nSeason names:");
        foreach (string name in names)
        {
            Console.WriteLine($"- {name}");
        }
        
        // COUNT
        int count = Enum.GetValues(typeof(Season)).Length;
        Console.WriteLine($"\\nTotal seasons: {count}");
        
        // LOOP THROUGH ALL
        Console.WriteLine($"\\nUsing Enum.GetValues:");
        foreach (Season s in Enum.GetValues<Season>())  // C# 10+ generic version
        {
            Console.WriteLine($"{s}: {(int)s}");
        }
    }
}
```

**Line-by-Line Explanation:**
- `Enum.GetValues(typeof(Season))` - Returns array of all enum values
  - Returns Array, cast to Season[]
- `Enum.GetNames(typeof(Season))` - Returns string array of member names
  - Just names, not values
- `Enum.GetValues<Season>()` - Generic version (C# 10+)
  - No cast needed
  - Type-safe

---

## Flags Enums

**Flags enums** represent combinations of values using bitwise operations.

```csharp
using System;

// FLAGS ENUM - Can combine multiple values
[Flags]
enum FilePermissions
{
    None = 0,           // 0000
    Read = 1,           // 0001
    Write = 2,          // 0010
    Execute = 4,        // 0100
    Delete = 8,         // 1000
    
    // ComBINED VALUES
    ReadWrite = Read | Write,              // 0011 (3)
    ReadExecute = Read | Execute,          // 0101 (5)
    FullControl = Read | Write | Execute | Delete  // 1111 (15)
}

class FlagsDemo
{
    static void Main()
    {
        // SINGLE PERMISSION
        FilePermissions perms = FilePermissions.Read;
        Console.WriteLine($"Single permission: {perms}");
        
        // COMBINE MULTIPLE (BITWISE OR)
        FilePermissions multiPerms = FilePermissions.Read | FilePermissions.Write;
        Console.WriteLine($"Combined: {multiPerms} ({(int)multiPerms})");
        
        // CHECK IF HAS PERMISSION (BITWISE AND)
        bool canRead = (multiPerms & FilePermissions.Read) == FilePermissions.Read;
        bool canExecute = (multiPerms & FilePermissions.Execute) == FilePermissions.Execute;
        
        Console.WriteLine($"\\nCan read: {canRead}");
        Console.WriteLine($"Can execute: {canExecute}");
        
        // HASFLAG METHOD (Easier)
        Console.WriteLine($"Has Read: {multiPerms.HasFlag(FilePermissions.Read)}");
        Console.WriteLine($"Has Execute: {multiPerms.HasFlag(FilePermissions.Execute)}");
        
        // ADD PERMISSION
        multiPerms = multiPerms | FilePermissions.Execute;
        Console.WriteLine($"\\nAfter adding Execute: {multiPerms}");
        
        // REMOVE PERMISSION (BITWISE AND NOT)
        multiPerms = multiPerms & ~FilePermissions.Write;
        Console.WriteLine($"After removing Write: {multiPerms}");
        
        // TOGGLE PERMISSION (BITWISE XOR)
        multiPerms = multiPerms ^ FilePermissions.Delete;
        Console.WriteLine($"After toggling Delete: {multiPerms}");
        
        // FULL CONTROL
        FilePermissions admin = FilePermissions.FullControl;
        Console.WriteLine($"\\nFull control: {admin}");
        Console.WriteLine($"Has all permissions: {admin.HasFlag(FilePermissions.ReadWrite)}");
        
        // DISPLAY ALL SET FLAGS
        Console.WriteLine($"\\nSet flags in admin:");
        foreach (FilePermissions perm in Enum.GetValues<FilePermissions>())
        {
            if (perm != FilePermissions.None && admin.HasFlag(perm))
            {
                Console.WriteLine($"- {perm}");
            }
        }
    }
}
```

**Line-by-Line Explanation:**
- `[Flags]` - Attribute indicates this is a flags enum
  - Changes ToString behavior to show combined values
  - Not required but strongly recommended
- `Read = 1, Write = 2, Execute = 4` - Powers of 2
  - 1 = 2^0, 2 = 2^1, 4 = 2^2, 8 = 2^3
  - Each bit represents one flag
- `Read | Write` - Bitwise OR combines flags
  - 0001 | 0010 = 0011 (3)
- `(multiPerms & FilePermissions.Read) == FilePermissions.Read` - Bitwise AND checks flag
  - If result equals flag, it's set
- `HasFlag(FilePermissions.Read)` - Built-in method to check flags
  - Easier than manual bitwise check
  - Slightly slower (uses boxing)
- `multiPerms | FilePermissions.Execute` - Add flag
- `multiPerms & ~FilePermissions.Write` - Remove flag
  - ~ is bitwise NOT
  - &~ clears the bit
- `multiPerms ^ FilePermissions.Delete` - Toggle flag
  - XOR: sets if unset, unsets if set

**Bitwise Operations:**
```
OR (|) - Add flag:
  Read:    0001
  Write:   0010
           ────  |
  Result:  0011 (Read | Write)

AND (&) - Check flag:
  Combined: 0011
  Read:     0001
            ────  &
  Result:   0001 (has Read)

AND NOT (&~) - Remove flag:
  Combined: 0011
  ~Write:   1101
            ────  &
  Result:   0001 (Read only)

XOR (^) - Toggle flag:
  Combined: 0011
  Write:    0010
            ────  ^
  Result:   0001 (Write removed)
```

### Flags Best Practices

```csharp
// ✅ GOOD: Powers of 2
[Flags]
enum Permissions
{
    None = 0,
    Read = 1,      // 2^0
    Write = 2,     // 2^1
    Execute = 4,   // 2^2
    Delete = 8     // 2^3
}

// ✅ GOOD: Using bit shift
[Flags]
enum Features
{
    None = 0,
    Feature1 = 1 << 0,  // 1
    Feature2 = 1 << 1,  // 2
    Feature3 = 1 << 2,  // 4
    Feature4 = 1 << 3   // 8
}

// ❌ BAD: No [Flags] attribute
enum BadFlags
{
    None = 0,
    Read = 1,
    Write = 2
}
// Will work but ToString shows number instead of "Read, Write"

// ❌ BAD: Not powers of 2
[Flags]
enum BadValues
{
    Value1 = 1,
    Value2 = 3,  // Wrong! Not a power of 2
    Value3 = 5   // Wrong!
}
```

---

## Enum Best Practices

### Design Guidelines

```csharp
using System;

// ✅ 1. Use enums for fixed sets of related constants
enum OrderStatus
{
    Pending,
    Processing,
    Shipped,
    Delivered,
    Cancelled
}

// ✅ 2. Provide a None or Default value (0)
enum Priority
{
    None = 0,     // Or Default, Unknown
    Low = 1,
    Medium = 2,
    High = 3
}

// ✅ 3. Don't use magic numbers
void ProcessOrder(OrderStatus status)  // ✅ Clear
{
    if (status == OrderStatus.Shipped) { }
}

void ProcessOrderInt(int status)  // ❌ What does 2 mean?
{
    if (status == 2) { }  // Unclear!
}

// ✅ 4. Use [Flags] for combinable values
[Flags]
enum DaysOfWeek
{
    None = 0,
    Monday = 1,
    Tuesday = 2,
    Wednesday = 4,
    Thursday = 8,
    Friday = 16,
    Saturday = 32,
    Sunday = 64,
    Weekdays = Monday | Tuesday | Wednesday | Thursday | Friday,
    Weekend = Saturday | Sunday,
    All = Weekdays | Weekend
}

// ✅ 5. Consider using struct with constants for complex cases
struct HttpStatusCode
{
    public const int OK = 200;
    public const int NotFound = 404;
    public const int ServerError = 500;
}

// ✅ 6. Document unusual values
enum Temperature
{
    /// <summary>Absolute zero</summary>
    AbsoluteZero = -273,
    
    /// <summary>Water freezes</summary>
    Freezing = 0,
    
    /// <summary>Room temperature</summary>
    Room = 20,
    
    /// <summary>Water boils</summary>
    Boiling = 100
}

class BestPractices
{
    static void Main()
    {
        // Validate enum values from external sources
        int userInput = 999;
        
        if (Enum.IsDefined(typeof(Priority), userInput))
        {
            Priority priority = (Priority)userInput;
            Console.WriteLine($"Valid: {priority}");
        }
        else
        {
            Console.WriteLine("Invalid priority value!");
        }
        
        // Switch with enum - use all values
        OrderStatus status = OrderStatus.Shipped;
        
        switch (status)
        {
            case OrderStatus.Pending:
                break;
            case OrderStatus.Processing:
                break;
            case OrderStatus.Shipped:
                break;
            case OrderStatus.Delivered:
                break;
            case OrderStatus.Cancelled:
                break;
            default:
                throw new ArgumentException($"Unknown status: {status}");
        }
    }
}
```

---

## Key Takeaways

### Critical Concepts

1. **Enums are Named Constants**
   - Set of related values
   - Type-safe (cannot mix enums)
   - Stored as integers

2. **Underlying Type**
   - Default: int (4 bytes)
   - Can specify: byte, short, long, etc.
   - Must be integral type

3. **Flags Enums**
   - Use [Flags] attribute
   - Values must be powers of 2
   - Combine with | (OR)
   - Check with & (AND) or HasFlag

4. **Conversion**
   - ToString() for name
   - Parse/TryParse for string to enum
   - Cast for int conversion
   - IsDefined to validate

### Best Practices

```csharp
// ✅ DO: Use enum for fixed sets
enum Color { Red, Green, Blue }

// ✅ DO: Provide default/none value
enum Status
{
    None = 0,
    Active = 1,
    Inactive = 2
}

// ✅ DO: Use TryParse for user input
if (Enum.TryParse<Color>(input, out Color color))
{
    // Safe
}

// ✅ DO: Use [Flags] for combinable values
[Flags]
enum Permissions
{
    None = 0,
    Read = 1,
    Write = 2,
    ReadWrite = Read | Write
}

// ❌ DON'T: Use magic numbers
void Process(int status)  // What values are valid?
{
    if (status == 1) { }  // What does 1 mean?
}

// ✅ DO: Use enum instead
void Process(OrderStatus status)
{
    if (status == OrderStatus.Pending) { }  // Clear!
}

// ❌ DON'T: Cast without validation
Color color = (Color)999;  // Undefined! No error!

// ✅ DO: Validate first
if (Enum.IsDefined(typeof(Color), value))
{
    Color color = (Color)value;
}
```

### Common Pitfalls

```csharp
// PITFALL 1: Cast without validation
enum Color { Red, Green, Blue }

int value = 999;
Color color = (Color)value;  // NO ERROR, but undefined!
Console.WriteLine(color);     // Prints "999"

// FIX:
if (Enum.IsDefined(typeof(Color), value))
{
    Color validColor = (Color)value;
}

// PITFALL 2: Forgetting [Flags] attribute
enum Permissions  // Missing [Flags]!
{
    Read = 1,
    Write = 2
}

Permissions perms = Permissions.Read | Permissions.Write;
Console.WriteLine(perms);  // Prints "3" instead of "Read, Write"

// FIX:
[Flags]
enum Permissions { ... }

// PITFALL 3: Non-power-of-2 in flags
[Flags]
enum Bad
{
    Value1 = 1,
    Value2 = 3,  // Not power of 2!
    Value3 = 5
}  // Can't properly combine values

// FIX: Use powers of 2
[Flags]
enum Good
{
    Value1 = 1,   // 2^0
    Value2 = 2,   // 2^1
    Value3 = 4    // 2^2
}
```

### Interview Questions

1. **What is an enum?**
   - Set of named constants
   - Value type
   - Stored as integer (default int)

2. **What is the default underlying type?**
   - int (32-bit integer)
   - Can change to byte, short, long, etc.

3. **What is the [Flags] attribute?**
   - Indicates enum can combine multiple values
   - Uses bitwise operations
   - Values should be powers of 2

4. **How to convert string to enum?**
   - Enum.Parse (throws exception)
   - Enum.TryParse (safe, returns bool)

5. **Can you cast any int to an enum?**
   - Yes, but it might be undefined!
   - Use Enum.IsDefined to validate

6. **What are the valid underlying types for enums?**
   - Integral types only: byte, sbyte, short, ushort, int, uint, long, ulong
   - Cannot use float, double, decimal, string

---

**Congratulations!** You've completed Day 01 - C# Fundamentals. You now understand data types, type conversion, assemblies, namespaces, structs, and enums. Ready for Day 02 on Object-Oriented Programming!
