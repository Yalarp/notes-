# 03 - Type Casting and Conversion

## Table of Contents
- [Understanding Type Casting](#understanding-type-casting)
- [Implicit Casting](#implicit-casting)
- [Explicit Casting](#explicit-casting)
- [Parse Methods](#parse-methods)
- [Convert Class](#convert-class)
- [TryParse Methods](#tryparse-methods)
- [Parse vs Convert vs TryParse](#parse-vs-convert-vs-tryparse)
- [Boxing and Unboxing](#boxing-and-unboxing)
- [User-Defined Conversions](#user-defined-conversions)
- [Key Takeaways](#key-takeaways)

---

## Understanding Type Casting

**Type casting** is the process of converting a value from one data type to another. C# provides multiple ways to perform type conversions, each with different characteristics and use cases.

### Type Conversion Hierarchy

```
Byte → Short → Int → Long → Float → Double → Decimal
         ↓      ↓      ↓      ↓       ↓
      UShort  UInt   ULong
```

**Widening (Implicit)**: Moving up the hierarchy (no data loss)
**Narrowing (Explicit)**: Moving down the hierarchy (potential data loss)

---

## Implicit Casting

**Implicit casting** happens automatically when there's no risk of data loss. The compiler performs the conversion without requiring explicit syntax.

```csharp
using System;

class ImplicitCasting
{
    static void Main()
    {
        // INTEGER PROMOTIONS (smaller to larger)
        
        byte byteValue = 100;
        short shortValue = byteValue;     // byte → short (implicit)
        int intValue = shortValue;        // short → int (implicit)
        long longValue = intValue;        // int → long (implicit)
        
        Console.WriteLine("Integer Promotions:");
        Console.WriteLine($"byte({sizeof(byte)} bytes): {byteValue}");
        Console.WriteLine($"short({sizeof(short)} bytes): {shortValue}");
        Console.WriteLine($"int({sizeof(int)} bytes): {intValue}");
        Console.WriteLine($"long({sizeof(long)} bytes): {longValue}");
        
        // FLOATING-POINT PROMOTIONS
        
        float floatValue = 3.14f;
        double doubleValue = floatValue;   // float → double (implicit)
        
        Console.WriteLine($"\\nFloating-Point Promotions:");
        Console.WriteLine($"float: {floatValue}");
        Console.WriteLine($"double: {doubleValue}");
        
        // INTEGER TO FLOATING-POINT
        
        int number = 42;
        float floatNumber = number;        // int → float (implicit)
        double doubleNumber = number;      // int → double (implicit)
        
        Console.WriteLine($"\\nInteger to Float:");
        Console.WriteLine($"int: {number}");
        Console.WriteLine($"float: {floatNumber}");
        Console.WriteLine($"double: {doubleNumber}");
        
        // CHAR TO INTEGER
        
        char letter = 'A';
        int asciiValue = letter;           // char → int (implicit)
        
        Console.WriteLine($"\\nChar to Int:");
        Console.WriteLine($"'{letter}' has ASCII value: {asciiValue}");
        
        // EXPRESSIONS WITH MIXED TYPES
        
        byte b = 10;
        short s = 20;
        int result = b + s;                // Promoted to int automatically
        
        Console.WriteLine($"\\nMixed Expression:");
        Console.WriteLine($"byte(10) + short(20) = int({result})");
        
        // IMPLICIT CONVERSION RULES
        // ✅ Allowed: No data loss
        long bigNum = 12345;
        // ❌ Not allowed: Potential data loss
        // int smallNum = bigNum;  // Error: Cannot implicitly convert long to int
    }
}
```

**Line-by-Line Explanation:**
- `short shortValue = byteValue;` - Compiler automatically converts byte to short
  - byte: 8 bits (0-255)
  - short: 16 bits (-32,768 to 32,767)
  - No data loss possible, so implicit conversion allowed
- `int intValue = shortValue;` - short fits entirely within int range
- `long longValue = intValue;` - int fits entirely within long range
- `double doubleValue = floatValue;` - float has less precision, safe to widen
- `int asciiValue = letter;` - char is 16-bit unsigned, converts to Unicode value
- `int result = b + s;` - In expressions, smaller types are promoted to int

**Execution Flow:**
1. byteValue created with value 100
2. shortValue receives promoted copy (byte → short)
3. Chain of promotions continues through int and long
4. floatValue converted to double (precision unchanged at this step)
5. Integer 42 converted to float and double representations
6. Character 'A' (Unicode 65) converted to integer
7. In expression, byte and short promoted to int before addition

**Why Implicit Casting is Safe:**
```
byte (1 byte):  [   100    ]
                     ↓
short (2 bytes): [0][  100  ]  ← Extra space, no loss
                     ↓
int (4 bytes):   [0][0][0][100] ← Even more space
```

---

## Explicit Casting

**Explicit casting** is required when conversion might cause data loss. You must use **cast operator** `(type)` to tell the compiler you understand the risk.

```csharp
using System;

class ExplicitCasting
{
    static void Main()
    {
        // NARROWING CONVERSIONS
        
        long largNumber = 123456789;
        int mediumNumber = (int)largeNumber;    // Explicit cast required
        short smallNumber = (short)mediumNumber; // Explicit cast required
        byte tinyNumber = (byte)smallNumber;     // Explicit cast required
        
        Console.WriteLine("Narrowing Conversions:");
        Console.WriteLine($"long: {largeNumber}");
        Console.WriteLine($"int: {mediumNumber}");
        Console.WriteLine($"short: {smallNumber}");
        Console.WriteLine($"byte: {tinyNumber}");
        
        // DATA LOSS EXAMPLE
        
        int bigValue = 300;
        byte smallValue = (byte)bigValue;  // Data loss!
        
        Console.WriteLine($"\\nData Loss Example:");
        Console.WriteLine($"Original int: {bigValue}");
        Console.WriteLine($"Cast to byte: {smallValue}");  // Output: 44 (300 % 256)
        
        // Why? byte range is 0-255, so 300 wraps around:
        // 300 - 256 = 44
        
        // FLOATING-POINT TO INTEGER
        
        double pi = 3.14159;
        int truncated = (int)pi;           // Decimal part lost!
        
        Console.WriteLine($"\\nFloat to Int:");
        Console.WriteLine($"Original double: {pi}");
        Console.WriteLine($"Cast to int: {truncated}");    // Output: 3
        
        // ROUNDING WITH EXPLICIT CAST
        
        double price = 19.99;
        int roundedDown = (int)price;                    // 19
        int roundedProper = (int)Math.Round(price);      // 20
        int roundedUp = (int)Math.Ceiling(price);        // 20
        
        Console.WriteLine($"\\nRounding Options:");
        Console.WriteLine($"Original: {price}");
        Console.WriteLine($"Cast (truncate): {roundedDown}");
        Console.WriteLine($"Math.Round: {roundedProper}");
        Console.WriteLine($"Math.Ceiling: {roundedUp}");
        
        // INTEGER TO CHAR
        
        int asciiValue = 65;
        char character = (char)asciiValue;  // Explicit cast to char
        
        Console.WriteLine($"\\nInt to Char:");
        Console.WriteLine($"ASCII {asciiValue} = '{character}'");
        
        // OVERFLOW CHECKING
        
        try
        {
            checked
            {
                int maxInt = int.MaxValue;
                int overflow = maxInt + 1;  // OverflowException in checked context
            }
        }
        catch (OverflowException ex)
        {
            Console.WriteLine($"\\nOverflow caught: {ex.Message}");
        }
        
        unchecked
        {
            int maxInt = int.MaxValue;
            int overflow = maxInt + 1;      // Wraps to int.MinValue
            Console.WriteLine($"Unchecked overflow: {maxInt} + 1 = {overflow}");
        }
    }
}
```

**Line-by-Line Explanation:**
- `(int)largeNumber` - Cast operator explicitly converts long to int
  - Syntax: `(TargetType)value`
  - Required because long is larger than int
  - Compiler enforces explicit acknowledgment of potential data loss
- `byte smallValue = (byte)bigValue;` - 300 doesn't fit in byte (0-255)
  - Result: 300 mod 256 = 44
  - Lower 8 bits kept, higher bits discarded
- `(int)pi` - Truncation, not rounding! Decimal part simply removed
- `Math.Round(price)` - Proper rounding, then cast to int
- `checked { }` - Enables overflow checking, throws exception on overflow
- `unchecked { }` - Disables overflow checking, allows wrapping

**Execution Flow:**
1. largeNumber declared as long
2. Explicit cast to int (may lose data if value > int.MaxValue)
3. Chain of narrowing casts continues
4. bigValue (300) cast to byte:
   - byte max is 255
   - 300 in binary: 00000001 00101100
   - Lower 8 bits: 00101100 = 44
5. pi cast to int: decimal removed, not rounded
6. checked block detects overflow, throws exception
7. unchecked block allows wrapping to minimum value

**Overflow Behavior:**
```
int.MaxValue = 2,147,483,647
              + 1
              ───────────────
              2,147,483,648  (exceeds max)
              
In unchecked: wraps to int.MinValue = -2,147,483,648
In checked: throws OverflowException
```

---

## Parse Methods

**Parse** methods convert strings to specific types. They throw exceptions if conversion fails.

```csharp
using System;

class ParseMethods
{
    static void Main()
    {
        // INT PARSE
        
        string numberString = "12345";
        int number = int.Parse(numberString);
        
        Console.WriteLine("Parse Methods:");
        Console.WriteLine($"String '{numberString}' → int: {number}");
        
        // DOUBLE PARSE
        
        string priceString = "19.99";
        double price = double.Parse(priceString);
        
        Console.WriteLine($"String '{priceString}' → double: {price}");
        
        // BOOL PARSE
        
        string flagString = "true";
        bool flag = bool.Parse(flagString);  // Case-insensitive
        
        Console.WriteLine($"String '{flagString}' → bool: {flag}");
        
        // DATETIME PARSE
        
        string dateString = "2024-12-19";
        DateTime date = DateTime.Parse(dateString);
        
        Console.WriteLine($"String '{dateString}' → DateTime: {date:yyyy-MM-dd}");
        
        // PARSE WITH WHITESPACE
        
        string spacedNumber = "  42  ";
        int trimmedNumber = int.Parse(spacedNumber);  // Whitespace ignored
        
        Console.WriteLine($"String '{spacedNumber}' → int: {trimmedNumber}");
        
        // PARSE EXCEPTIONS
        
        try
        {
            string invalidNumber = "abc";
            int result = int.Parse(invalidNumber);  // FormatException!
        }
        catch (FormatException ex)
        {
            Console.WriteLine($"\\nFormatException: Cannot parse 'abc' as int");
        }
        
        try
        {
            string tooLarge = "999999999999999999999";
            int result = int.Parse(tooLarge);  // OverflowException!
        }
        catch (OverflowException ex)
        {
            Console.WriteLine($"OverflowException: Number too large for int");
        }
        
        try
        {
            string nullString = null;
            int result = int.Parse(nullString);  // ArgumentNullException!
        }
        catch (ArgumentNullException ex)
        {
            Console.WriteLine($"ArgumentNullException: Cannot parse null");
        }
        
        // PARSE WITH CULTURE
        
        string germanPrice = "1.234,56";  // German format: period for thousands
        // This would fail with default culture!
        
        var germanCulture = new System.Globalization.CultureInfo("de-DE");
        double germanNumber = double.Parse(germanPrice, germanCulture);
        
        Console.WriteLine($"\\nGerman format '{germanPrice}' → {germanNumber}");
        
        // ENUM PARSE
        
        string dayString = "Monday";
        DayOfWeek day = (DayOfWeek)Enum.Parse(typeof(DayOfWeek), dayString);
        
        Console.WriteLine($"String '{dayString}' → Enum: {day}");
    }
}
```

**Line-by-Line Explanation:**
- `int.Parse(numberString)` - Static method on int type
  - Attempts to convert string to int
  - Throws FormatException if string is not valid number
  - Throws OverflowException if number too large
  - Throws ArgumentNullException if string is null
- `bool.Parse(flagString)` - Accepts "true" or "false" (case-insensitive)
- `DateTime.Parse(dateString)` - Parses various date formats
- Whitespace trimmed automatically before parsing
- `:yyyy-MM-dd` - Format string for DateTime display
- `new CultureInfo("de-DE")` - German culture for parsing European number format
- `Enum.Parse(typeof(DayOfWeek), dayString)` - Parses enum from string name
  - `typeof(DayOfWeek)` gets the enum type
  - Case-sensitive by default

**Exception Hierarchy:**
```
SystemException
    ├─ FormatException        ("abc" is not a number)
    ├─ OverflowException      (Number too large/small)
    └─ ArgumentNullException  (null string passed)
```

---

## Convert Class

**Convert class** provides methods to convert between base types. More flexible than Parse and handles nulls better.

```csharp
using System;

class ConvertClass
{
    static void Main()
    {
        // STRING TO NUMERIC
        
        string numberString = "12345";
        int number = Convert.ToInt32(numberString);
        long longNumber = Convert.ToInt64(numberString);
        double doubleNumber = Convert.ToDouble(numberString);
        
        Console.WriteLine("Convert Class:");
        Console.WriteLine($"ToInt32: {number}");
        Console.WriteLine($"ToInt64: {longNumber}");
        Console.WriteLine($"ToDouble: {doubleNumber}");
        
        // NUMERIC CONVERSIONS
        
        double pi = 3.14159;
        int rounded = Convert.ToInt32(pi);     // Rounds to nearest int!
        
        Console.WriteLine($"\\nDouble {pi} → Int32: {rounded}");  // Output: 3
        
        double price = 19.5;
        int roundedPrice = Convert.ToInt32(price);
        Console.WriteLine($"Double {price} → Int32: {roundedPrice}");  // Output: 20 (rounded up!)
        
        double exact = 19.4;
        int roundedExact = Convert.ToInt32(exact);
        Console.WriteLine($"Double {exact} → Int32: {roundedExact}");  // Output: 19 (rounded down)
        
        // NULL HANDLING
        
        string nullString = null;
        int fromNull = Convert.ToInt32(nullString);  // Returns 0, doesn't throw!
        
        Console.WriteLine($"\\nNull Handling:");
        Console.WriteLine($"Convert.ToInt32(null) = {fromNull}");
        
        // Compare with Parse:
        try
        {
            int parseNull = int.Parse(nullString);  // ArgumentNullException!
        }
        catch (ArgumentNullException)
        {
            Console.WriteLine("int.Parse(null) throws exception!");
        }
        
        // BOOL CONVERSIONS
        
        int zero = 0;
        int nonZero = 42;
        
        bool fromZero = Convert.ToBoolean(zero);        // false
        bool fromNonZero = Convert.ToBoolean(nonZero);  // true
        
        Console.WriteLine($"\\nBoolean Conversions:");
        Console.WriteLine($"Convert.ToBoolean(0) = {fromZero}");
        Console.WriteLine($"Convert.ToBoolean(42) = {fromNonZero}");
        
        // STRING CONVERSIONS
        
        int value = 100;
        string stringValue = Convert.ToString(value);
        
        bool flag = true;
        string stringFlag = Convert.ToString(flag);
        
        Console.WriteLine($"\\nTo String:");
        Console.WriteLine($"int 100 → '{stringValue}'");
        Console.WriteLine($"bool true → '{stringFlag}'");
        
        // BASE CONVERSIONS (Binary, Octal, Hex)
        
        int decimal number = 255;
        
        string binary = Convert.ToString(decimalNumber, 2);    // Base 2
        string octal = Convert.ToString(decimalNumber, 8);     // Base 8
        string hex = Convert.ToString(decimalNumber, 16);      // Base 16
        
        Console.WriteLine($"\\nBase Conversions for {decimalNumber}:");
        Console.WriteLine($"Binary: {binary}");
        Console.WriteLine($"Octal: {octal}");
        Console.WriteLine($"Hexadecimal: {hex}");
        
        // FROM BASE
        
        string hexString = "FF";
        int fromHex = Convert.ToInt32(hexString, 16);  // Parse from base 16
        
        Console.WriteLine($"Hex '{hexString}' → Decimal: {fromHex}");
        
        // DATETIME CONVERSIONS
        
        string dateString = "2024-12-19";
        DateTime dateTime = Convert.ToDateTime(dateString);
        
        Console.WriteLine($"\\nString → DateTime: {dateTime:yyyy-MM-dd}");
        
        // TYPE CHECKING
        
        object obj = 123;
        int converted = Convert.ToInt32(obj);  // Works with object type
        
        Console.WriteLine($"Object → Int32: {converted}");
    }
}
```

**Line-by-Line Explanation:**
- `Convert.ToInt32(numberString)` - Converts string to 32-bit integer
  - Similar to int.Parse but returns 0 for null instead of throwing
  - Rounds floating-point inputs (unlike cast which truncates)
- `Convert.ToInt32(pi)` - **Rounds to nearest integer** (banker's rounding)
  - 3.14159 → 3
  - 19.5 → 20 (round to even: "banker's rounding")
  - Different from `(int)` cast which truncates
- `Convert.ToInt32(nullString)` - Returns 0 for null (doesn't throw)
- `Convert.ToBoolean(0)` - 0 becomes false, any other number becomes true
- `Convert.ToString(value, 2)` - Second parameter is base (2=binary, 8=octal, 16=hex)
- `Convert.ToInt32(hexString, 16)` - Parses from base 16

**Convert vs Cast:**
```csharp
double value = 3.7;

int cast = (int)value;              // Truncates: 3
int convert = Convert.ToInt32(value); // Rounds: 4

// Cast: faster, truncates
// Convert: rounds, handles nulls, more flexible
```

---

## TryParse Methods

**TryParse** is the safest conversion method. It returns bool indicating success/failure and uses an `out` parameter for the result.

```csharp
using System;

class TryParseMethods
{
    static void Main()
    {
        // BASIC TRYPARSE
        
        string validNumber = "12345";
        int result;
        
        bool success = int.TryParse(validNumber, out result);
        
        if (success)
        {
            Console.WriteLine($"Successfully parsed: {result}");
        }
        else
        {
            Console.WriteLine("Parsing failed!");
        }
        
        // TRYPARSE WITH INVALID INPUT
        
        string invalidNumber = "abc";
        int result2;
        
        if (int.TryParse(invalidNumber, out result2))
        {
            Console.WriteLine($"Parsed: {result2}");
        }
        else
        {
            Console.WriteLine($"'{ invalidNumber}' is not a valid number");
            Console.WriteLine($"result2 is now: {result2}");  // Default value: 0
        }
        
        // INLINE OUT VARIABLE (C# 7+)
        
        if (int.TryParse("54321", out int result3))
        {
            Console.WriteLine($"\\nInline out: {result3}");
        }
        
        // TRYPARSE WITH NULL
        
        string nullString = null;
        
        if (int.TryParse(nullString, out int result4))
        {
            Console.WriteLine($"Parsed null: {result4}");
        }
        else
        {
            Console.WriteLine($"\\nCannot parse null (no exception thrown)");
            Console.WriteLine($"result4: {result4}");  // Default value: 0
        }
        
        // REAL-WORLD USAGE WITH USER INPUT
        
        Console.Write("\\nEnter your age: ");
        string input = Console.ReadLine();
        
        if (int.TryParse(input, out int age))
        {
            if (age >= 18)
            {
                Console.WriteLine("You are an adult.");
            }
            else
            {
                Console.WriteLine("You are a minor.");
            }
        }
        else
        {
            Console.WriteLine("Invalid age entered!");
        }
        
        // TRYPARSE WITH DIFFERENT TYPES
        
        if (double.TryParse("3.14159", out double pi))
        {
            Console.WriteLine($"\\nDouble: {pi}");
        }
        
        if (bool.TryParse("true", out bool flag))
        {
            Console.WriteLine($"Bool: {flag}");
        }
        
        if (DateTime.TryParse("2024-12-19", out DateTime date))
        {
            Console.WriteLine($"Date: {date:yyyy-MM-dd}");
        }
        
        // TRYPARSE WITH CULTURE
        
        string europeanNumber = "1.234,56";
        var germanCulture = new System.Globalization.CultureInfo("de-DE");
        
        if (double.TryParse(europeanNumber, 
                           System.Globalization.NumberStyles.Any, 
                           germanCulture, 
                           out double europeanValue))
        {
            Console.WriteLine($"\\nEuropean number: {europeanValue}");
        }
        
        // DISCARD OUT VARIABLE (C# 7+)
        
        if (int.TryParse("12345", out _))  // Discard with underscore
        {
            Console.WriteLine("Valid number (value discarded)");
        }
        
        // ENUM TRYPARSE
        
        if (Enum.TryParse<DayOfWeek>("Monday", out DayOfWeek day))
        {
            Console.WriteLine($"Day: {day}");
        }
        
        // COMBINING WITH NULL-COALESCING
        
        string userInput = "not a number";
        int parsedValue = int.TryParse(userInput, out int temp) ? temp : -1;
        
        Console.WriteLine($"\\nParsed with default: {parsedValue}");  // -1
    }
}
```

**Line-by-Line Explanation:**
- `int.TryParse(validNumber, out result)` - Two parameters:
  - Input string to parse
  - `out result` - output parameter that receives parsed value
  - Returns bool: true if successful, false otherwise
- `out result2` - out parameter MUST be assigned before method returns
  - If parsing fails, receives default value (0 for int)
- `out int result3` - Inline declaration (C# 7+), variable declared in argument list
  - Scope: available after the if statement
- `out _` - Discard syntax, indicates we don't care about the parsed value
  - Just checking if string can be parsed
- `Enum.TryParse<DayOfWeek>` - Generic version for enum parsing
- `int.TryParse(...) ? temp : -1` - Ternary operator for default value

**Execution Flow:**
1. TryParse attempts to parse string
2. If successful:
   - out parameter set to parsed value
   - Returns true
3. If unsuccessful:
   - out parameter set to default value (0, false, null)
   - Returns false
   - No exception thrown!

**Why TryParse is Best:**
```csharp
// ❌ Parse: Can crash program
try
{
    int value = int.Parse(userInput);
}
catch (Exception ex)
{
    // Error handling code
}

// ✅ TryParse: Safe and clean
if (int.TryParse(userInput, out int value))
{
    // Use value
}
else
{
    // Invalid input
}
```

---

## Parse vs Convert vs TryParse

### Comparison Table

| Feature | Parse | Convert | TryParse |
|---------|-------|---------|----------|
| **Syntax** | `int.Parse(s)` | `Convert.ToInt32(s)` | `int.TryParse(s, out n)` |
| **Returns** | Parsed value | Converted value | bool (success/fail) |
| **Null handling** | Throws exception | Returns 0/default | Returns false |
| **Invalid input** | Throws exception | Throws exception | Returns false |
| **Rounding** | N/A | Rounds | N/A |
| **Performance** | Fast | Slower | Fast |
| **Safety** | Unsafe | Unsafe | **Safe** |
| **Use case** | Trusted input | Type conversion | User input |

```csharp
using System;

class ComparisonDemo
{
    static void Main()
    {
        string validString = "123";
        string invalidString = "abc";
        string nullString = null;
        
        Console.WriteLine("=== VALID INPUT ===\\n");
        
        // All three work with valid input
        Console.WriteLine($"Parse: {int.Parse(validString)}");
        Console.WriteLine($"Convert: {Convert.ToInt32(validString)}");
        Console.WriteLine($"TryParse: {(int.TryParse(validString, out int v1) ? v1 : -1)}");
        
        Console.WriteLine("\\n=== INVALID INPUT ===\\n");
        
        // Parse - Throws exception
        try
        {
            int result = int.Parse(invalidString);
        }
        catch (FormatException)
        {
            Console.WriteLine("Parse: Exception thrown ❌");
        }
        
        // Convert - Throws exception
        try
        {
            int result = Convert.ToInt32(invalidString);
        }
        catch (FormatException)
        {
            Console.WriteLine("Convert: Exception thrown ❌");
        }
        
        // TryParse - Returns false
        if (!int.TryParse(invalidString, out int v2))
        {
            Console.WriteLine("TryParse: Returns false ✅");
        }
        
        Console.WriteLine("\\n=== NULL INPUT ===\\n");
        
        // Parse - Throws exception
        try
        {
            int result = int.Parse(nullString);
        }
        catch (ArgumentNullException)
        {
            Console.WriteLine("Parse: Exception thrown ❌");
        }
        
        // Convert - Returns 0
        int convertResult = Convert.ToInt32(nullString);
        Console.WriteLine($"Convert: Returns {convertResult} (no exception) ⚠️");
        
        // TryParse - Returns false
        if (!int.TryParse(nullString, out int v3))
        {
            Console.WriteLine("TryParse: Returns false ✅");
        }
        
        Console.WriteLine("\\n=== FLOATING-POINT CONVERSION ===\\n");
        
        double doubleValue = 3.7;
        
        // Cannot use Parse on double to get int
        // int parseDouble = int.Parse(doubleValue);  // ERROR: wrong type
        
        // Convert - Rounds!
        int convertDouble = Convert.ToInt32(doubleValue);
        Console.WriteLine($"Convert.ToInt32(3.7) = {convertDouble}");  // 4 (rounded)
        
        // Cast - Truncates!
        int castDouble = (int)doubleValue;
        Console.WriteLine($"(int)3.7 = {castDouble}");  // 3 (truncated)
        
        // TryParse - Works with string only
        if (double.TryParse("3.7", out double d) && int.TryParse(((int)d).ToString(), out int v4))
        {
            Console.WriteLine($"TryParse chain: {v4}");
        }
        
        Console.WriteLine("\\n=== RECOMMENDATIONS ===\\n");
        Console.WriteLine("✅ Use TryParse for: User input, external data");
        Console.WriteLine("✅ Use Parse for: Trusted data (config files)");
        Console.WriteLine("✅ Use Convert for: Type conversion, null handling");
        Console.WriteLine("✅ Use Cast for: Performance-critical code");
    }
}
```

**Line-by-Line Explanation:**
- Parse throws on invalid/null input - use only with trusted data
- Convert returns default for null, throws for invalid - use for type conversion
- TryParse never throws, best for user input validation
- Convert rounds floating-point, cast truncates

**Decision Tree:**
```
Is the input from a user or external source?
├─ Yes → Use TryParse ✅
└─ No → Is it a trusted source (config, database)?
    ├─ Yes → Use Parse
    └─ No → Are you converting between numeric types?
        ├─ Yes → Use Convert or Cast
        └─ No → Use TryParse (safest)
```

---

## Boxing and Unboxing

**Boxing**: Converting value type to reference type (wraps value in object)
**Unboxing**: Extracting value type from object

```csharp
using System;

class BoxingUnboxing
{
    static void Main()
    {
        // BOXING - Value type to Reference type
        
        int number = 42;           // Value type on stack
        object boxed = number;     // Boxing: wrapped in object on heap
        
        Console.WriteLine($"Original int: {number}");
        Console.WriteLine($"Boxed object: {boxed}");
        Console.WriteLine($"boxed type: {boxed.GetType().Name}");  // Int32
        
        // UNBOXING - Reference type to Value type
        
        object boxedValue = 100;
        int unboxed = (int)boxedValue;  // Unboxing requires explicit cast
        
        Console.WriteLine($"\\nBoxed: {boxedValue}");
        Console.WriteLine($"Unboxed: {unboxed}");
        
        // INCORRECT UNBOXING
        
        object boxedInt = 42;
        try
        {
            long wrongUnbox = (long)boxedInt;  // InvalidCastException!
        }
        catch (InvalidCastException)
        {
            Console.WriteLine($"\\nCannot unbox int as long!");
        }
        
        // Correct approach - unbox to original type first
        long correct = (long)(int)boxedInt;  // Unbox to int, then cast to long
        Console.WriteLine($"Correct unboxing: {correct}");
        
        // PERFORMANCE IMPACT
        
        Console.WriteLine($"\\n=== Performance Demo ===");
        
        int iterations = 1000000;
        
        // Without boxing (fast)
        var watch1 = System.Diagnostics.Stopwatch.StartNew();
        for (int i = 0; i < iterations; i++)
        {
            int temp = i;
        }
        watch1.Stop();
        Console.WriteLine($"Without boxing: {watch1.ElapsedMilliseconds}ms");
        
        // With boxing (slow)
        var watch2 = System.Diagnostics.Stopwatch.StartNew();
        for (int i = 0; i < iterations; i++)
        {
            object temp = i;  // Boxing on each iteration!
        }
        watch2.Stop();
        Console.WriteLine($"With boxing: {watch2.ElapsedMilliseconds}ms");
        
        // BOXING IN COLLECTIONS (Pre-Generics)
        
        System.Collections.ArrayList list = new System.Collections.ArrayList();
        list.Add(1);      // Boxing
        list.Add(2);      // Boxing
        list.Add(3);      // Boxing
        
        int firstValue = (int)list[0];  // Unboxing
        
        // Better: Use generic collections (no boxing!)
        System.Collections.Generic.List<int> genericList = 
            new System.Collections.Generic.List<int>();
        genericList.Add(1);  // No boxing!
        genericList.Add(2);  // No boxing!
        
        Console.WriteLine($"\\nArrayList (boxes): {list[0].GetType().Name}");
        Console.WriteLine($"List<int> (no box): {genericList[0].GetType().Name}");
    }
}
```

**Line-by-Line Explanation:**
- `object boxed = number;` - Implicit boxing conversion
  - Value copied from stack to heap
  - Wrapped in object container
  - Adds overhead (allocation, GC pressure)
- `(int)boxedValue` - Explicit unboxing required
  - Extracts value from object wrapper
  - Must cast to exact original type
- `(long)boxedInt` - Cannot directly unbox to different type
  - Must unbox to original type first, then cast
- Boxing in loops - Very expensive!
  - Each iteration allocates new object on heap
  - Causes garbage collection pressure
- `ArrayList` - Non-generic, causes boxing for value types
- `List<int>` - Generic, no boxing needed

**Memory Diagram:**
```
BEFORE BOXING:
Stack:
┌──────────┐
│ number:42│
└──────────┘

AFTER BOXING:
Stack:          Heap:
┌──────────┐   ┌─────────────┐
│ boxed ───┼──→│ boxObject   │
└──────────┘   │ value: 42   │
               │ type: Int32 │
               └─────────────┘
```

---

## User-Defined Conversions

You can define custom implicit and explicit conversions for your own types.

```csharp
using System;

// Custom type with conversion operators
class Temperature
{
    public double Celsius { get; set; }
    
    public Temperature(double celsius)
    {
        Celsius = celsius;
    }
    
    // IMPLICIT conversion from double to Temperature
    public static implicit operator Temperature(double celsius)
    {
        return new Temperature(celsius);
    }
    
    // EXPLICIT conversion from Temperature to double
    public static explicit operator double(Temperature temp)
    {
        return temp.Celsius;
    }
    
    // EXPLICIT conversion from Temperature to int (truncated)
    public static explicit operator int(Temperature temp)
    {
        return (int)temp.Celsius;
    }
    
    public override string ToString()
    {
        return $"{Celsius}°C";
    }
}

class UserDefinedConversions
{
    static void Main()
    {
        // IMPLICIT CONVERSION (double → Temperature)
        Temperature temp1 = 25.5;  // Calls implicit operator
        Console.WriteLine($"temp1: {temp1}");
        
        // EXPLICIT CONVERSION (Temperature → double)
        double celsius = (double)temp1;  // Calls explicit operator
        Console.WriteLine($"Celsius: {celsius}");
        
        // EXPLICIT CONVERSION (Temperature → int)
        int truncated = (int)temp1;  // Calls explicit operator
        Console.WriteLine($"Truncated: {truncated}°C");
        
        // Using in expressions
        Temperature outdoor = 30.0;  // Implicit conversion
        Temperature indoor = 22.0;
        
        double outdoorValue = (double)outdoor;  // Explicit
        double indoorValue = (double)indoor;
        double difference = outdoorValue - indoorValue;
        
        Console.WriteLine($"\\nTemperature difference: {difference}°C");
    }
}
```

**Line-by-Line Explanation:**
- `implicit operator Temperature(double celsius)` - Defines implicit conversion
  - Allows: `Temperature temp = 25.5;`
  - No cast needed
  - Use when conversion is safe and obvious
- `explicit operator double(Temperature temp)` - Defines explicit conversion
  - Requires: `double c = (double)temp;`
  - Use when conversion might lose information
- User-defined conversions make APIs more intuitive

---

## Key Takeaways

### Critical Concepts

1. **Implicit vs Explicit Casting**
   - Implicit: Safe, automatic (small→large)
   - Explicit: Requires cast, potential data loss (large→small)

2. **Conversion Methods**
   - **Parse**: Fast, throws exceptions, for trusted data
   - **Convert**: Flexible, handles null, rounds floating-point
   - **TryParse**: Safest, for user input, never throws

3. **Boxing/Unboxing**
   - Avoid in performance-critical code
   - Use generics instead of non-generic collections

4. **Best Practices**
   - Use TryParse for user input
   - Use Convert for type flexibility
   - Avoid unnecessary boxing
   - Define user conversions for custom types

### Decision Guide

```csharp
// ✅ User input → Always TryParse
Console.Write("Enter age: ");
if (int.TryParse(Console.ReadLine(), out int age))
{
    // Safe!
}

// ✅ Config file → Parse (fail fast if corrupted)
int maxConnections = int.Parse(config["MaxConnections"]);

// ✅ Type conversion → Convert
double value = 3.7;
int rounded = Convert.ToInt32(value);  // Rounds to 4

// ✅ Performance critical → Cast
int truncated = (int)value;  // Truncates to 3 (faster)

// ❌ Avoid boxing in loops
for (int i = 0; i < 1000000; i++)
{
    object boxed = i;  // Very slow!
}

// ✅ Use generics
List<int> numbers = new List<int>();  // No boxing
```

### Common Mistakes

```csharp
// ❌ MISTAKE 1: Not handling parse failures
int age = int.Parse(Console.ReadLine());  // Can crash!

// ✅ FIX:
if (int.TryParse(Console.ReadLine(), out int age))
{
    // Use age
}

// ❌ MISTAKE 2: Wrong unboxing type
object boxed = 42;
long value = (long)boxed;  // InvalidCastException!

// ✅ FIX:
long value = (long)(int)boxed;  // Unbox, then cast

// ❌ MISTAKE 3: Expecting rounding from cast
double price = 19.99;
int dollars = (int)price;  // 19, not 20!

// ✅ FIX:
int dollars = (int)Math.Round(price);  // 20

// ❌ MISTAKE 4: Boxing in collections
ArrayList list = new ArrayList();
list.Add(1);  // Boxing!

// ✅ FIX:
List<int> list = new List<int>();
list.Add(1);  // No boxing
```

### Interview Questions

1. **What's the difference between implicit and explicit casting?**
   - Implicit: Automatic, no data loss (byte→int)
   - Explicit: Requires cast operator, may lose data (int→byte)

2. **When should you use TryParse vs Parse?**
   - TryParse: User input, untrusted data (safe)
   - Parse: Trusted data, fail-fast on errors

3. **What is boxing and why should you avoid it?**
   - Boxing: Value type → reference type conversion
   - Creates heap allocation, GC pressure
   - Use generics to avoid

4. **What's the difference between Convert.ToInt32 and (int) cast?**
   - Convert: Rounds floating-point, handles null
   - Cast: Truncates, faster, no null handling

5. **Can you unbox to a different type?**
   - No! Must unbox to exact original type first
   - Then cast to desired type

---

**Next:** Assemblies and Namespaces - Learn how C# organizes code!
