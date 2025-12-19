# 02 - Data Types and Variables

## Table of Contents
- [Understanding the Type System](#understanding-the-type-system)
- [Value Types](#value-types)
- [Reference Types](#reference-types)
- [Variable Declaration and Initialization](#variable-declaration-and-initialization)
- [Type Inference with var](#type-inference-with-var)
- [Constants and Readonly](#constants-and-readonly)
- [Default Values](#default-values)
- [Key Takeaways](#key-takeaways)

---

## Understanding the Type System

### C# Type System Hierarchy

```
                    System.Object (object)
                           │
        ┌──────────────────┴──────────────────┐
        │                                      │
   Value Types                           Reference Types
        │                                      │
  ┌─────┴─────┐                       ┌────────┴────────┐
  │           │                       │                 │
Simple    Struct/Enum              Class            String/Arrays
Types                              Delegate          Interfaces
  │
Numeric, Bool, Char
```

### Value Types vs Reference Types

**Key Differences:**

| Aspect | Value Types | Reference Types |
|--------|-------------|-----------------|
| Storage | Stack (usually) | Heap |
| Contains | Actual data | Reference (pointer) to data |
| Default | 0, false, '\\0' | null |
| Assignment | Copies value | Copies reference |
| Inheritance | Cannot inherit | Can inherit |
| Performance | Faster access | Extra indirection |

```csharp
using System;

class TypeComparison
{
    static void Main()
    {
        // VALUE TYPE EXAMPLE
        int num1 = 10;           // num1 stores the actual value 10
        int num2 = num1;         // num2 gets a COPY of the value
        num2 = 20;               // Changing num2 doesn't affect num1
        
        Console.WriteLine($"num1: {num1}");  // Output: 10
        Console.WriteLine($"num2: {num2}");  // Output: 20
        
        // REFERENCE TYPE EXAMPLE
        Person person1 = new Person("John");  // person1 stores reference
        Person person2 = person1;             // person2 stores SAME reference
        person2.Name = "Jane";                // Changes the same object!
        
        Console.WriteLine($"person1: {person1.Name}");  // Output: Jane
        Console.WriteLine($"person2: {person2.Name}");  // Output: Jane
    }
}

class Person
{
    public string Name { get; set; }
    public Person(string name) { Name = name; }
}
```

**Memory Diagram:**

```
VALUE TYPES:
Stack:
┌──────────┐
│ num1: 10 │
├──────────┤
│ num2: 20 │
└──────────┘

REFERENCE TYPES:
Stack:              Heap:
┌───────────┐      ┌─────────────────┐
│ person1 ──┼─────→│ Person Object   │
├───────────┤   ┌─→│ Name: "Jane"    │
│ person2 ──┼───┘  └─────────────────┘
└───────────┘
```

**Line-by-Line Explanation:**
- `int num1 = 10;` - Declares int (value type), stores 10 directly in num1
- `int num2 = num1;` - Creates independent copy of num1's value
- `num2 = 20;` - Changes only num2's copy, num1 remains 10
- `Person person1 = new Person("John");` - Creates object on heap, stores reference in person1
- `Person person2 = person1;` - Copies reference, both variables point to SAME object
- `person2.Name = "Jane";` - Modifies the shared object through person2's reference

**Execution Flow:**
1. Stack allocates space for num1, stores 10
2. Stack allocates space for num2, copies value from num1
3. num2 changes to 20 (num1 unaffected)
4. Heap allocates Person object, person1 stores address
5. person2 copies person1's address (both point to same object)
6. Setting Name through person2 affects shared object
7. Both person1 and person2 see "Jane"

---

## Value Types

### Integer Types

```csharp
using System;

class IntegerTypes
{
    static void Main()
    {
        // SIGNED INTEGERS (can be positive or negative)
        
        sbyte tinyNumber = -128;      // 8-bit, range: -128 to 127
        short smallNumber = -32768;   // 16-bit, range: -32,768 to 32,767
        int mediumNumber = -2147483648; // 32-bit, most common
        long largeNumber = -9223372036854775808L; // 64-bit, note the 'L' suffix
        
        // UNSIGNED INTEGERS (only positive)
        
        byte positiveOnly = 255;      // 8-bit, range: 0 to 255
        ushort unsignedShort = 65535; // 16-bit, range: 0 to 65,535
        uint unsignedInt = 4294967295U; // 32-bit, note the 'U' suffix
        ulong unsignedLong = 18446744073709551615UL; // 64-bit, 'UL' suffix
        
        // DISPLAYING VALUES
        Console.WriteLine("Integer Types:");
        Console.WriteLine($"sbyte: {tinyNumber}, Size: {sizeof(sbyte)} bytes");
        Console.WriteLine($"short: {smallNumber}, Size: {sizeof(short)} bytes");
        Console.WriteLine($"int: {mediumNumber}, Size: {sizeof(int)} bytes");
        Console.WriteLine($"long: {largeNumber}, Size: {sizeof(long)} bytes");
        
        Console.WriteLine($"\\nbyte: {positiveOnly}, Size: {sizeof(byte)} bytes");
        Console.WriteLine($"ushort: {unsignedShort}, Size: {sizeof(ushort)} bytes");
        Console.WriteLine($"uint: {unsignedInt}, Size: {sizeof(uint)} bytes");
        Console.WriteLine($"ulong: {unsignedLong}, Size: {sizeof(ulong)} bytes");
        
        // LIMITS
        Console.WriteLine($"\\nint range: {int.MinValue} to {int.MaxValue}");
        Console.WriteLine($"long range: {long.MinValue} to {long.MaxValue}");
    }
}
```

**Line-by-Line Explanation:**
- `sbyte tinyNumber = -128;` - Signed 8-bit integer, can store -128 to 127
- `long largeNumber = -9223372036854775808L;` - 'L' suffix tells compiler this is a long literal
- `uint unsignedInt = 4294967295U;` - 'U' suffix for unsigned integer literal
- `sizeof(sbyte)` - Compile-time operator that returns size in bytes
- `int.MinValue` - Static property giving minimum value for int type
- `$"int: {mediumNumber}"` - String interpolation embeds variable value

**When to Use Each Type:**
- **byte**: File I/O, network protocols, RGB colors (0-255)
- **short**: Rarely used, only when memory is critical
- **int**: Default choice for integers (most common)
- **long**: Large numbers (timestamps, IDs, calculations exceeding billions)

### Floating-Point Types

```csharp
using System;

class FloatingPointTypes
{
    static void Main()
    {
        // FLOATING-POINT TYPES
        
        float singlePrecision = 3.14159f;        // 32-bit, ~6-9 digits precision
        double doublePrecision = 3.14159265358979; // 64-bit, ~15-17 digits
        decimal highPrecision = 3.14159265358979323846m; // 128-bit, 28-29 digits
        
        Console.WriteLine("Floating-Point Types:");
        Console.WriteLine($"float: {singlePrecision}, Size: {sizeof(float)} bytes");
        Console.WriteLine($"double: {doublePrecision}, Size: {sizeof(double)} bytes");
        Console.WriteLine($"decimal: {highPrecision}, Size: {sizeof(decimal)} bytes");
        
        // PRECISION DEMONSTRATION
        float f = 0.1f + 0.2f;
        double d = 0.1 + 0.2;
        decimal m = 0.1M + 0.2M;
        
        Console.WriteLine($"\\nPrecision Test (should be 0.3):");
        Console.WriteLine($"float: {f}");      // May show 0.30000001
        Console.WriteLine($"double: {d}");      // May show 0.30000000000000004
        Console.WriteLine($"decimal: {m}");     // Exactly 0.3
        
        // WHEN TO USE EACH
        Console.WriteLine($"\\nUse Cases:");
        Console.WriteLine("float: Graphics, scientific calculations (less precise)");
        Console.WriteLine("double: Default for floating-point (general purpose)");
        Console.WriteLine("decimal: Financial calculations (precise)");
        
        // DECIMAL FOR MONEY
        decimal price = 19.99M;
        decimal tax = 0.08M;
        decimal total = price + (price * tax);
        Console.WriteLine($"\\nPrice: ${price:F2}");
        Console.WriteLine($"Total with tax: ${total:F2}");
    }
}
```

**Line-by-Line Explanation:**
- `float singlePrecision = 3.14159f;` - 'f' suffix required for float literals
- `decimal highPrecision = 3.14159265358979323846m;` - 'm' or 'M' suffix for decimal
- `float f = 0.1f + 0.2f;` - Floating-point math has precision issues
- `${total:F2}` - Format specifier: F2 means fixed-point with 2 decimal places

**Precision Issues:**
```csharp
// WHY DECIMALS ARE BETTER FOR MONEY
double money1 = 0.1 + 0.2;  // May not equal exactly 0.3!
decimal money2 = 0.1M + 0.2M;  // Always exactly 0.3

if (money1 == 0.3)      // May be false due to rounding!
    Console.WriteLine("Equal (double)");
    
if (money2 == 0.3M)     // Always true
    Console.WriteLine("Equal (decimal)");
```

### Boolean Type

```csharp
using System;

class BooleanType
{
    static void Main()
    {
        // BOOLEAN: true or false (1 byte storage)
        bool isStudent = true;
        bool hasLicense = false;
        
        Console.WriteLine($"isStudent: {isStudent}, Size: {sizeof(bool)} byte");
        Console.WriteLine($"hasLicense: {hasLicense}");
        
        // BOOLEAN OPERATIONS
        bool canDrive = !isStudent && hasLicense;  // AND operation
        bool eligible = isStudent || hasLicense;    // OR operation
        bool opposite = !isStudent;                 // NOT operation
        
        Console.WriteLine($"\\nLogical Operations:");
        Console.WriteLine($"canDrive (NOT student AND has license): {canDrive}");
        Console.WriteLine($"eligible (student OR has license): {eligible}");
        Console.WriteLine($"opposite (NOT student): {opposite}");
        
        // COMPARISON OPERATORS
        int age = 25;
        bool isAdult = age >= 18;
        bool isTeen = age >= 13 && age <= 19;
        bool isMinor = age < 18;
        
        Console.WriteLine($"\\nAge: {age}");
        Console.WriteLine($"isAdult: {isAdult}");
        Console.WriteLine($"isTeen: {isTeen}");
        Console.WriteLine($"isMinor: {isMinor}");
    }
}
```

**Line-by-Line Explanation:**
- `bool isStudent = true;` - Boolean can only be true or false (not 0/1 like C)
- `sizeof(bool)` - Always 1 byte, even though it only needs 1 bit
- `!isStudent && hasLicense` - ! is NOT, && is AND
- `isStudent || hasLicense` - || is OR (short-circuit evaluation)
- `age >= 18` - Comparison operators return boolean values

**Short-Circuit Evaluation:**
```csharp
bool result1 = false && ExpensiveOperation();  // ExpensiveOperation NOT called!
bool result2 = true || ExpensiveOperation();   // ExpensiveOperation NOT called!
```

### Character Type

```csharp
using System;

class CharacterType
{
    static void Main()
    {
        // CHARACTER: Unicode character (16-bit)
        char letter = 'A';          // Single quotes for char
        char digit = '9';           // This is a character, not a number!
        char symbol = '@';
        char unicode = '\\u0041';   // Unicode escape for 'A'
        char newline = '\\n';       // Special character
        
        Console.WriteLine($"Character Type:");
        Console.WriteLine($"letter: {letter}, Size: {sizeof(char)} bytes");
        Console.WriteLine($"digit: {digit} (not a number!)");
        Console.WriteLine($"symbol: {symbol}");
        Console.WriteLine($"unicode: {unicode}");
        
        // CHAR TO INT CONVERSION
        int asciiValue = letter;    // Implicit conversion to ASCII/Unicode value
        Console.WriteLine($"\\nASCII value of '{letter}': {asciiValue}");
        
        // CHARACTER OPERATIONS
        char nextLetter = (char)(letter + 1);  // 'A' + 1 = 'B'
        Console.WriteLine($"Next letter after '{letter}': {nextLetter}");
        
        // CHECKING CHARACTER TYPES
        Console.WriteLine($"\\nCharacter Checks:");
        Console.WriteLine($"Is '{letter}' a letter? {char.IsLetter(letter)}");
        Console.WriteLine($"Is '{digit}' a digit? {char.IsDigit(digit)}");
        Console.WriteLine($"Is '{letter}' uppercase? {char.IsUpper(letter)}");
        Console.WriteLine($"Is '{symbol}' punctuation? {char.IsPunctuation(symbol)}");
        
        // CASE CONVERSION
        char lower = char.ToLower(letter);
        char upper = char.ToUpper('a');
        Console.WriteLine($"\\nLowercase '{letter}': {lower}");
        Console.WriteLine($"Uppercase 'a': {upper}");
    }
}
```

**Line-by-Line Explanation:**
- `char letter = 'A';` - Single quotes for character literals (vs double quotes for strings)
- `char unicode = '\\u0041';` - Unicode escape sequence (hexadecimal)
- `int asciiValue = letter;` - Implicit conversion, char is 16-bit unsigned integer internally
- `(char)(letter + 1)` - Arithmetic on chars produces int, explicit cast back to char
- `char.IsLetter(letter)` - Static method checks if character is a letter

**Escape Sequences:**
```csharp
char tab = '\\t';         // Tab
char newline = '\\n';     // New line
char backslash = '\\\\';  // Backslash itself
char quote = '\\'';       // Single quote
char nullChar = '\\0';    // Null character
```

### Struct Types

```csharp
using System;

// CUSTOM VALUE TYPE - STRUCT
struct Point
{
    public int X;      // Field
    public int Y;      // Field
    
    // Constructor
    public Point(int x, int y)
    {
        X = x;         // Initialize X
        Y = y;         // Initialize Y
    }
    
    // Method
    public double DistanceFromOrigin()
    {
        return Math.Sqrt(X * X + Y * Y);  // Pythagorean theorem
    }
    
    // Override ToString for display
    public override string ToString()
    {
        return $"({X}, {Y})";
    }
}

class StructExample
{
    static void Main()
    {
        // CREATE STRUCT INSTANCES
        Point p1 = new Point(3, 4);       // Using constructor
        Point p2;                          // Default initialization
        p2.X = 10;
        p2.Y = 20;
        
        Console.WriteLine($"Point 1: {p1}");
        Console.WriteLine($"Point 2: {p2}");
        Console.WriteLine($"Distance from origin: {p1.DistanceFromOrigin():F2}");
        
        // VALUE TYPE BEHAVIOR
        Point p3 = p1;                     // Creates a copy
        p3.X = 100;                        // Changes only p3
        
        Console.WriteLine($"\\nAfter modifying p3:");
        Console.WriteLine($"p1: {p1}");    // Still (3, 4)
        Console.WriteLine($"p3: {p3}");    // Now (100, 4)
        
        // BUILT-IN STRUCT EXAMPLES
        DateTime today = DateTime.Now;     // DateTime is a struct
        TimeSpan duration = TimeSpan.FromHours(2.5);
        
        Console.WriteLine($"\\nBuilt-in Structs:");
        Console.WriteLine($"Today: {today:yyyy-MM-dd HH:mm:ss}");
        Console.WriteLine($"Duration: {duration.TotalMinutes} minutes");
    }
}
```

**Line-by-Line Explanation:**
- `struct Point` - Defines custom value type (stored on stack)
- `public int X;` - Public field, directly accessible
- `public Point(int x, int y)` - Constructor to initialize fields
- `Math.Sqrt(X * X + Y * Y)` - Calculates distance using square root
- `override string ToString()` - Overrides object's ToString method
- `Point p2;` - Default initialization (X=0, Y=0)
- `Point p3 = p1;` - Copies entire struct (value semantics)
- `:F2` - Format specifier: 2 decimal places

**Execution Flow:**
1. p1 created on stack with X=3, Y=4
2. p2 created with default values (0, 0)
3. p2 fields set to 10, 20
4. ToString called to display points
5. DistanceFromOrigin calculates: sqrt(3²+4²) = 5.00
6. p3 receives complete copy of p1's data
7. p3.X changed to 100 (p1 unaffected)

---

## Reference Types

### String Type

```csharp
using System;

class StringType
{
    static void Main()
    {
        // STRING BASICS
        string name = "John Doe";          // Double quotes for strings
        string empty = "";                  // Empty string
        string nullString = null;           // Null reference
        
        Console.WriteLine($"name: '{name}'");
        Console.WriteLine($"empty: '{empty}', Length: {empty.Length}");
        Console.WriteLine($"nullString: {nullString ?? "null"}");
        
        // STRING PROPERTIES
        Console.WriteLine($"\\nString Properties:");
        Console.WriteLine($"Length: {name.Length}");
        Console.WriteLine($"First character: {name[0]}");     // Indexer
        Console.WriteLine($"Last character: {name[name.Length - 1]}");
        
        // STRING IMMUTABILITY
        string original = "Hello";
        string modified = original.ToUpper();   // Creates NEW string
        
        Console.WriteLine($"\\nImmutability:");
        Console.WriteLine($"Original: {original}");  // Still "Hello"
        Console.WriteLine($"Modified: {modified}");  // "HELLO"
        
        // STRING METHODS
        string text = "  Hello World  ";
        Console.WriteLine($"\\nString Methods:");
        Console.WriteLine($"Original: '{text}'");
        Console.WriteLine($"ToUpper: '{text.ToUpper()}'");
        Console.WriteLine($"ToLower: '{text.ToLower()}'");
        Console.WriteLine($"Trim: '{text.Trim()}'");
        Console.WriteLine($"Contains 'World': {text.Contains("World")}");
        Console.WriteLine($"StartsWith 'Hello': {text.Trim().StartsWith("Hello")}");
        Console.WriteLine($"Replace: '{text.Replace("World", "C#")}'");
        
        // STRING CONCATENATION
        string firstName = "John";
        string lastName = "Doe";
        
        string fullName1 = firstName + " " + lastName;         // Concatenation
        string fullName2 = string.Concat(firstName, " ", lastName);
        string fullName3 = $"{firstName} {lastName}";          // Interpolation (best)
        string fullName4 = string.Format("{0} {1}", firstName, lastName);
        
        Console.WriteLine($"\\nConcatenation Methods:");
        Console.WriteLine($"Using +: {fullName1}");
        Console.WriteLine($"Using Concat: {fullName2}");
        Console.WriteLine($"Using interpolation: {fullName3}");
        Console.WriteLine($"Using Format: {fullName4}");
        
        // STRING COMPARISON
        string str1 = "hello";
        string str2 = "Hello";
        
        Console.WriteLine($"\\nString Comparison:");
        Console.WriteLine($"str1 == str2: {str1 == str2}");  // Case-sensitive
        Console.WriteLine($"Equals (ignore case): {str1.Equals(str2, StringComparison.OrdinalIgnoreCase)}");
        
        // VERBATIM STRINGS
        string path1 = "C:\\\\Users\\\\John\\\\Documents";  // Escaped backslashes
        string path2 = @"C:\\Users\\John\\Documents";        // Verbatim string (no escaping)
        
        Console.WriteLine($"\\nVerbatim Strings:");
        Console.WriteLine($"Escaped: {path1}");
        Console.WriteLine($"Verbatim: {path2}");
        
        // RAW STRING LITERALS (C# 11+)
        string json = """
            {
                "name": "John",
                "age": 30
            }
            """;
        Console.WriteLine($"\\nRaw String (JSON):\\n{json}");
    }
}
```

**Line-by-Line Explanation:**
- `string name = "John Doe";` - String is reference type but has special syntax
- `nullString ?? "null"` - Null-coalescing operator: use "null" if nullString is null
- `name[0]` - Strings support indexing like arrays (returns char)
- `original.ToUpper()` - Returns NEW string; original unchanged (immutability)
- `text.Trim()` - Removes whitespace from start and end
- `text.Contains("World")` - Returns true if substring found
- `$"{firstName} {lastName}"` - String interpolation (most readable)
- `string.Format("{0} {1}", ...)` - Older formatting method (positional)
- `StringComparison.OrdinalIgnoreCase` - Enum value for case-insensitive comparison
- `@"C:\\Users"` - Verbatim string: @ prefix disables escape sequences
- `"""..."""` - Raw string literal (C# 11+), no escaping needed

**String Immutability:**
```
Memory when modifying strings:

original = "Hello"
   ↓
Heap: ["Hello"] ← original points here
   ↓
modified = original.ToUpper()
   ↓
Heap: ["Hello"] ← original still here
      ["HELLO"] ← NEW string created for modified
```

### Array Type

```csharp
using System;

class ArrayType
{
    static void Main()
    {
        // SINGLE-DIMENSIONAL ARRAYS
        
        // Declaration and initialization
        int[] numbers = new int[5];            // Array of 5 integers (all 0)
        string[] names = { "John", "Jane", "Bob" };  // Collection initializer
        double[] prices = new double[] { 19.99, 29.99, 39.99 };
        
        Console.WriteLine("Single-Dimensional Arrays:");
        Console.WriteLine($"numbers length: {numbers.Length}");
        Console.WriteLine($"names: {string.Join(", ", names)}");
        
        // Accessing elements (zero-based indexing)
        numbers[0] = 10;     // First element
        numbers[1] = 20;
        numbers[4] = 50;     // Last element (index 4 for length 5)
        
        Console.WriteLine($"\\nnumbers[0]: {numbers[0]}");
        Console.WriteLine($"numbers[4]: {numbers[4]}");
        
        // Iterating with for loop
        Console.WriteLine($"\\nAll numbers:");
        for (int i = 0; i < numbers.Length; i++)
        {
            Console.WriteLine($"numbers[{i}] = {numbers[i]}");
        }
        
        // Iterating with foreach
        Console.WriteLine($"\\nAll names:");
        foreach (string name in names)
        {
            Console.WriteLine($"  {name}");
        }
        
        // MULTI-DIMENSIONAL ARRAYS (Rectangular)
        
        int[,] matrix = new int[3, 4];  // 3 rows, 4 columns
        int[,] grid = {
            { 1, 2, 3 },
            { 4, 5, 6 },
            { 7, 8, 9 }
        };
        
        Console.WriteLine($"\\n2D Array (grid):");
        for (int row = 0; row < grid.GetLength(0); row++)
        {
            for (int col = 0; col < grid.GetLength(1); col++)
            {
                Console.Write($"{grid[row, col]} ");
            }
            Console.WriteLine();
        }
        
        // JAGGED ARRAYS (Array of arrays)
        
        int[][] jagged = new int[3][];  // 3 rows, varying columns
        jagged[0] = new int[] { 1, 2 };
        jagged[1] = new int[] { 3, 4, 5 };
        jagged[2] = new int[] { 6, 7, 8, 9 };
        
        Console.WriteLine($"\\nJagged Array:");
        for (int i = 0; i < jagged.Length; i++)
        {
            Console.Write($"Row {i}: ");
            for (int j = 0; j < jagged[i].Length; j++)
            {
                Console.Write($"{jagged[i][j]} ");
            }
            Console.WriteLine();
        }
        
        // ARRAY METHODS
        int[] unsorted = { 5, 2, 8, 1, 9 };
        Console.WriteLine($"\\nOriginal: {string.Join(", ", unsorted)}");
        
        Array.Sort(unsorted);
        Console.WriteLine($"Sorted: {string.Join(", ", unsorted)}");
        
        Array.Reverse(unsorted);
        Console.WriteLine($"Reversed: {string.Join(", ", unsorted)}");
        
        int index = Array.IndexOf(unsorted, 5);
        Console.WriteLine($"Index of 5: {index}");
        
        // REFERENCE TYPE BEHAVIOR
        int[] arr1 = { 1, 2, 3 };
        int[] arr2 = arr1;         // Both reference same array!
        arr2[0] = 100;
        
        Console.WriteLine($"\\nReference behavior:");
        Console.WriteLine($"arr1[0]: {arr1[0]}");  // 100 (affected!)
        Console.WriteLine($"arr2[0]: {arr2[0]}");  // 100
        
        // Creating true copy
        int[] arr3 = (int[])arr1.Clone();
        arr3[0] = 999;
        
        Console.WriteLine($"After cloning:");
        Console.WriteLine($"arr1[0]: {arr1[0]}");  // 100 (unchanged)
        Console.WriteLine($"arr3[0]: {arr3[0]}");  // 999
    }
}
```

**Line-by-Line Explanation:**
- `int[] numbers = new int[5];` - Creates array of 5 integers, all initialized to 0
- `{ "John", "Jane", "Bob" }` - Collection initializer syntax
- `numbers[0]` - Zero-based indexing, first element
- `i < numbers.Length` - Length property gives number of elements
- `foreach (string name in names)` - Iterates through each element (read-only)
- `int[,] matrix` - Comma indicates 2D array (rectangular)
- `grid.GetLength(0)` - Gets length of first dimension (rows)
- `grid[row, col]` - Access 2D element with two indices
- `int[][] jagged` - Array of arrays (can have different lengths)
- `jagged[0] = new int[] { 1, 2 };` - Each row is separately allocated
- `Array.Sort(unsorted)` - Static method sorts array in-place
- `string.Join(", ", unsorted)` - Joins array elements with separator
- `int[] arr2 = arr1;` - Copies reference, not data!
- `(int[])arr1.Clone()` - Creates shallow copy of array

**Memory Diagram:**
```
Stack:              Heap:
┌─────────┐        ┌───┬───┬───┬───┬───┐
│ numbers │───────→│ 10│ 20│ 0 │ 0 │ 50│
└─────────┘        └───┴───┴───┴───┴───┘

┌─────────┐        ┌───────┬───────┬───────┐
│ names   │───────→│"John" │"Jane" │ "Bob" │
└─────────┘        └───────┴───────┴───────┘

Reference sharing:
┌──────┐
│ arr1 │─────┐    ┌───┬───┬───┐
├──────┤     ├───→│100│ 2 │ 3 │
│ arr2 │─────┘    └───┴───┴───┘
└──────┘
```

---

## Variable Declaration and Initialization

### Declaration Syntax

```csharp
using System;

class VariableDeclaration
{
    static void Main()
    {
        // BASIC DECLARATION
        int age;                    // Declared but not initialized
        // Console.WriteLine(age);  // ERROR: Use of unassigned local variable
        
        age = 25;                   // Initialization
        Console.WriteLine($"Age: {age}");
        
        // DECLARATION WITH INITIALIZATION
        int count = 0;              // Declare and initialize
        string name = "John";
        
        // MULTIPLE DECLARATIONS
        int x = 10, y = 20, z = 30;  // Same type, same line
        Console.WriteLine($"x={x}, y={y}, z={z}");
        
        // EXPLICIT TYPE
        double price = 19.99;
        string message = "Hello";
        bool isActive = true;
        
        // CONSTANTS
        const double PI = 3.14159;   // Cannot be changed
        const int MAX_SIZE = 100;
        
        // PI = 3.14;                 // ERROR: Cannot assign to constant
        
        Console.WriteLine($"\\nConstants:");
        Console.WriteLine($"PI: {PI}");
        Console.WriteLine($"MAX_SIZE: {MAX_SIZE}");
        
        // READONLY (can be set in constructor or declaration)
        readonly int readonlyValue;  // ERROR in local scope!
        // readonly is for class fields only
        
        // DEFAULT VALUES
        int defaultInt = default;           // 0
        bool defaultBool = default;         // false
        string defaultString = default;     // null
        
        Console.WriteLine($"\\nDefault values:");
        Console.WriteLine($"defaultInt: {defaultInt}");
        Console.WriteLine($"defaultBool: {defaultBool}");
        Console.WriteLine($"defaultString: {defaultString ?? "null"}");
    }
}
```

---

## Type Inference with var

### Using var Keyword

```csharp
using System;
using System.Collections.Generic;

class TypeInference
{
    static void Main()
    {
        // TYPE INFERENCE WITH var
        
        var number = 42;                  // Inferred as int
        var price = 19.99;                // Inferred as double (not decimal!)
        var name = "John";                // Inferred as string
        var isActive = true;              // Inferred as bool
        
        // var is NOT dynamic - type is determined at compile time
        // number = "hello";              // ERROR: Cannot convert string to int
        
        Console.WriteLine("Type inference:");
        Console.WriteLine($"number is {number.GetType().Name}");
        Console.WriteLine($"price is {price.GetType().Name}");
        Console.WriteLine($"name is {name.GetType().Name}");
        
        // WHEN var IS REQUIRED
        
        var anonymous = new { Name = "John", Age = 30 };  // Anonymous type
        Console.WriteLine($"\\nAnonymous type: {anonymous.Name}, {anonymous.Age}");
        
        // WHEN var IS HELPFUL
        
        var dictionary = new Dictionary<string, List<int>>();  // Reduces repetition
        // vs
        Dictionary<string, List<int>> dictionary2 = new Dictionary<string, List<int>>();
        
        // WHEN NOT TO USE var
        
        var unclear = GetData();          // Type not obvious
        string clear = GetData();         // Better: explicit type
        
        // BEST PRACTICES
        
        var list = new List<string>();    // Good: obvious from right side
        var items = GetItems();           // Bad: what type is returned?
        List<string> items2 = GetItems(); // Good: explicit and clear
    }
    
    static string GetData()
    {
        return "Data";
    }
    
    static List<string> GetItems()
    {
        return new List<string> { "Item1", "Item2" };
    }
}
```

**Line-by-Line Explanation:**
- `var number = 42;` - Compiler infers type as int from right side
- `var price = 19.99;` - Inferred as double (use 19.99M for decimal!)
- `new { Name = "John", Age = 30 }` - Anonymous type, requires var
- `Dictionary<string, List<int>>` - Complex type, var reduces verbosity
- `GetData()` - Return type not obvious, explicit type is clearer

**var Rules:**
1. Must be initialized at declaration
2. Cannot be null (use explicit type for null)
3. Type determined at compile time (NOT dynamic)
4. Cannot be used for fields (class members)
5. Cannot be used for method parameters or return types

---

## Default Values

### Default Value Table

| Type | Default Value |
|------|---------------|
| bool | false |
| byte, sbyte | 0 |
| short, ushort | 0 |
| int, uint | 0 |
| long, ulong | 0 |
| float | 0.0f |
| double | 0.0 |
| decimal | 0.0M |
| char | '\\0' (null character) |
| string | null |
| enum | 0 (first enumvalue or explicit 0) |
| struct | All fields set to their defaults |
| class | null |
| Nullable<T> | null |

```csharp
using System;

class DefaultValues
{
    // CLASS FIELDS GET AUTOMATIC DEFAULT VALUES
    static int classField;           // Automatically 0
    static string classString;       // Automatically null
    static bool classBool;           // Automatically false
    
    static void Main()
    {
        // LOCAL VARIABLES MUST BE INITIALIZED
        int localVar;
        // Console.WriteLine(localVar);  // ERROR: unassigned variable
        
        // USING default KEYWORD
        int num = default(int);          // 0
        bool flag = default(bool);       // false
        string text = default(string);   // null
        
        // C# 7.1+ - shorter syntax
        int num2 = default;
        
        Console.WriteLine("Default values:");
        Console.WriteLine($"int: {num}");
        Console.WriteLine($"bool: {flag}");
        Console.WriteLine($"string: {text ?? "null"}");
        
        // CLASS FIELDS
        Console.WriteLine($"\\nClass fields (automatic defaults):");
        Console.WriteLine($"classField: {classField}");
        Console.WriteLine($"classBool: {classBool}");
        Console.WriteLine($"classString: {classString ?? "null"}");
    }
}
```

---

## Key Takeaways

### Critical Concepts

1. **Value vs Reference Types**
   - Value types: Copy data (int, bool, struct)
   - Reference types: Copy reference (class, string, array)

2. **Type Safety**
   - C# is strongly typed
   - Type must be known at compile time (except dynamic)
   - Prevents many runtime errors

3. **Choose the Right Type**
   - int for whole numbers
   - double for general floating-point
   - decimal for money
   - string for text
   - bool for true/false

4. **var Usage**
   - Use when type is obvious
   - Required for anonymous types
   - Still strongly typed

### Best Practices

```csharp
// ✅ DO: Use appropriate types
decimal money = 19.99M;
int count = 100;
string name = "John";

// ❌ DON'T: Use wrong types for data
double money = 19.99;  // Use decimal for money!
byte count = 255;       // Use int unless memory critical

// ✅ DO: Initialize variables
int age = 0;
string name = string.Empty;  // Better than null for strings

// ❌ DON'T: Leave variables uninitialized
int age;
Console.WriteLine(age);  // Compiler error!

// ✅ DO: Use var when obvious
var list = new List<string>();
var name = "John";

// ❌ DON'T: Use var when unclear
var data = GetData();  // What type is this?

// ✅ DO: Use meaningful names
int customerCount = 0;
string firstName = "John";

// ❌ DON'T: Use cryptic names
int cc = 0;
string fn = "John";
```

### Common Pitfalls

```csharp
// PITFALL 1: Reference vs value
int[] arr1 = { 1, 2, 3 };
int[] arr2 = arr1;      // Both point to same array!
arr2[0] = 999;
// arr1[0] is now 999!

// PITFALL 2: Floating-point precision
double result = 0.1 + 0.2;     // May not be exactly 0.3!
decimal precise = 0.1M + 0.2M;  // Exactly 0.3

// PITFALL 3: Integer division
int half = 5 / 2;       // Result is 2, not 2.5!
double correct = 5.0 / 2;  // Result is 2.5

// PITFALL 4: Null reference
string name = null;
int length = name.Length;  // NullReferenceException!

// Safe:
int length = name?.Length ?? 0;  // Returns 0 if null
```

### Interview Questions

1. **What's the difference between value and reference types?**
   - Value types store actual data, copied on assignment
   - Reference types store memory address, assignment copies reference
   - Value types: stack, reference types: heap

2. **What's the difference between string and String?**
   - No difference! `string` is alias for `System.String`
   - Convention: use `string` in C#, `String` for .NET APIs

3. **Can you change a string?**
   - No, strings are immutable
   - Methods like ToUpper() return NEW strings
   - Use StringBuilder for multiple modifications

4. **What's the default value of a reference type?**
   - null

5. **What's the difference between const and readonly?**
   - const: Compile-time constant, must be initialized at declaration
   - readonly: Runtime constant, can be set in constructor
   - (More details in later notes)

---

**Next:** Type Casting and Conversion - Learn how to safely convert between types!
