# 📨 Delegates, Action, Func, and Predicate in C#

## 📚 Table of Contents
- [What are Delegates?](#-what-are-delegates)
- [Declaring and Using Delegates](#-declaring-and-using-delegates)
- [Multicast Delegates](#-multicast-delegates)
- [Generic Delegates: Action, Func, Predicate](#-generic-delegates)
- [Anonymous Methods and Lambdas](#-anonymous-methods-and-lambdas)
- [Interview Questions](#-interview-questions)

---

## 🎯 What are Delegates?

A **delegate** is a type-safe function pointer - a variable that holds a reference to a method.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    DELEGATE CONCEPT                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ┌──────────────┐     Points to     ┌──────────────┐              │
│   │   Delegate   │ ─────────────────▶│   Method     │              │
│   │   Variable   │                   │              │              │
│   └──────────────┘                   └──────────────┘              │
│                                                                     │
│   Use Cases:                                                        │
│   • Callback functions                                              │
│   • Event handling                                                  │
│   • LINQ operations                                                 │
│   • Strategy pattern                                                │
│   • Asynchronous programming                                        │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔷 Declaring and Using Delegates

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// DELEGATE BASICS
// ═══════════════════════════════════════════════════════════════════════════

using System;

// Line 1: Declare delegate type
//         Defines signature: return type + parameters
//         This delegate points to methods that take 2 ints and return int
public delegate int MathOperation(int a, int b);

// Line 2: Another delegate - void return, string parameter
public delegate void MessageHandler(string message);

class DelegateDemo
{
    // Methods that match delegate signatures
    static int Add(int x, int y) => x + y;
    static int Subtract(int x, int y) => x - y;
    static int Multiply(int x, int y) => x * y;
    
    static void PrintToConsole(string msg) => Console.WriteLine(msg);
    static void PrintUpper(string msg) => Console.WriteLine(msg.ToUpper());
    
    static void Main()
    {
        // ─────────────────────────────────────────────────────────────────
        // CREATING DELEGATE INSTANCES
        // ─────────────────────────────────────────────────────────────────
        
        // Method 1: Explicit instantiation
        MathOperation operation = new MathOperation(Add);
        // Line 3: Create delegate pointing to Add method
        
        // Method 2: Simplified (C# 2.0+)
        MathOperation op2 = Add;
        // Line 4: Direct method group conversion
        
        // ─────────────────────────────────────────────────────────────────
        // INVOKING DELEGATES
        // ─────────────────────────────────────────────────────────────────
        
        int result = operation(10, 5);
        // Line 5: Invoke delegate like a method
        Console.WriteLine($"Add: {result}");  // Output: Add: 15
        
        // Same as:
        result = operation.Invoke(10, 5);
        // Line 6: Explicit Invoke() method
        
        // ─────────────────────────────────────────────────────────────────
        // REASSIGNING DELEGATES
        // ─────────────────────────────────────────────────────────────────
        
        operation = Subtract;  // Now points to Subtract
        result = operation(10, 5);
        Console.WriteLine($"Subtract: {result}");  // Output: Subtract: 5
        
        operation = Multiply;  // Now points to Multiply
        result = operation(10, 5);
        Console.WriteLine($"Multiply: {result}");  // Output: Multiply: 50
        
        // ─────────────────────────────────────────────────────────────────
        // PASSING DELEGATES AS PARAMETERS
        // ─────────────────────────────────────────────────────────────────
        
        ProcessNumbers(10, 5, Add);
        ProcessNumbers(10, 5, Subtract);
        ProcessNumbers(10, 5, Multiply);
    }
    
    // Line 7: Method accepting delegate as parameter
    static void ProcessNumbers(int a, int b, MathOperation op)
    {
        int result = op(a, b);
        Console.WriteLine($"Result: {result}");
    }
}
```

---

## 🔶 Multicast Delegates

Delegates can hold references to **multiple methods** (invocation list).

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// MULTICAST DELEGATES
// ═══════════════════════════════════════════════════════════════════════════

using System;

public delegate void NotificationHandler(string message);

class MulticastDemo
{
    static void SendEmail(string msg) => Console.WriteLine($"📧 Email: {msg}");
    static void SendSMS(string msg) => Console.WriteLine($"📱 SMS: {msg}");
    static void LogToFile(string msg) => Console.WriteLine($"📝 Log: {msg}");
    
    static void Main()
    {
        NotificationHandler notify = null;
        
        // ─────────────────────────────────────────────────────────────────
        // ADDING METHODS (+=)
        // ─────────────────────────────────────────────────────────────────
        
        notify += SendEmail;
        // Line 1: Add first method to invocation list
        
        notify += SendSMS;
        // Line 2: Add second method
        
        notify += LogToFile;
        // Line 3: Add third method
        
        // Alternative syntax
        notify = SendEmail;
        notify = (NotificationHandler)Delegate.Combine(notify, new NotificationHandler(SendSMS));
        
        // Invoke - ALL methods are called in order
        notify("System alert!");
        // Output:
        // 📧 Email: System alert!
        // 📱 SMS: System alert!
        // 📝 Log: System alert!
        
        // ─────────────────────────────────────────────────────────────────
        // REMOVING METHODS (-=)
        // ─────────────────────────────────────────────────────────────────
        
        notify -= SendSMS;
        // Line 4: Remove SMS from invocation list
        
        notify("Another alert!");
        // Output:
        // 📧 Email: Another alert!
        // 📝 Log: Another alert!
        
        // ─────────────────────────────────────────────────────────────────
        // INVOCATION LIST
        // ─────────────────────────────────────────────────────────────────
        
        if (notify != null)
        {
            Delegate[] invocationList = notify.GetInvocationList();
            // Line 5: Get array of all subscribed methods
            
            Console.WriteLine($"Subscribers: {invocationList.Length}");
            foreach (Delegate d in invocationList)
            {
                Console.WriteLine($"  - {d.Method.Name}");
            }
        }
    }
}
```

### Multicast with Return Values

```csharp
// When multicast delegate has return type,
// only LAST method's return value is captured

public delegate int Calculator(int x, int y);

int Add(int a, int b) => a + b;
int Multiply(int a, int b) => a * b;

Calculator calc = Add;
calc += Multiply;

int result = calc(3, 4);
// result = 12 (Multiply is last, its return value is used)
// Add(3,4)=7 is executed but return value is lost!
```

---

## 🔷 Generic Delegates

C# provides built-in generic delegates so you don't need to define custom delegate types.

### Action<T> - No Return Value

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// ACTION<T> - DELEGATES THAT RETURN VOID
// ═══════════════════════════════════════════════════════════════════════════

using System;

class ActionDemo
{
    static void Main()
    {
        // Action = delegate with void return
        
        // ─────────────────────────────────────────────────────────────────
        // Action (no parameters)
        // ─────────────────────────────────────────────────────────────────
        
        Action sayHello = () => Console.WriteLine("Hello!");
        // Line 1: Action with no parameters
        sayHello();  // Output: Hello!
        
        // ─────────────────────────────────────────────────────────────────
        // Action<T> (one parameter)
        // ─────────────────────────────────────────────────────────────────
        
        Action<string> greet = name => Console.WriteLine($"Hello, {name}!");
        // Line 2: Action with one string parameter
        greet("John");  // Output: Hello, John!
        
        // ─────────────────────────────────────────────────────────────────
        // Action<T1, T2> (two parameters)
        // ─────────────────────────────────────────────────────────────────
        
        Action<string, int> printInfo = (name, age) => 
            Console.WriteLine($"{name} is {age} years old");
        // Line 3: Action with two parameters
        printInfo("Alice", 25);  // Output: Alice is 25 years old
        
        // ─────────────────────────────────────────────────────────────────
        // Action with up to 16 parameters
        // ─────────────────────────────────────────────────────────────────
        
        Action<int, int, int> sum3 = (a, b, c) => 
            Console.WriteLine($"Sum: {a + b + c}");
        sum3(1, 2, 3);  // Output: Sum: 6
        
        // ─────────────────────────────────────────────────────────────────
        // Passing Action as parameter
        // ─────────────────────────────────────────────────────────────────
        
        PerformAction("Test", s => Console.WriteLine(s.ToUpper()));
        // Output: TEST
    }
    
    static void PerformAction(string data, Action<string> action)
    {
        action(data);
    }
}
```

### Func<T> - With Return Value

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// FUNC<T> - DELEGATES THAT RETURN A VALUE
// ═══════════════════════════════════════════════════════════════════════════

using System;

class FuncDemo
{
    static void Main()
    {
        // Func = delegate with return value
        // Last type parameter is always the RETURN type
        
        // ─────────────────────────────────────────────────────────────────
        // Func<TResult> (no input, returns value)
        // ─────────────────────────────────────────────────────────────────
        
        Func<int> getRandomNumber = () => new Random().Next(1, 100);
        // Line 1: No parameters, returns int
        int num = getRandomNumber();
        Console.WriteLine($"Random: {num}");
        
        Func<DateTime> getCurrentTime = () => DateTime.Now;
        // Returns DateTime
        
        // ─────────────────────────────────────────────────────────────────
        // Func<T, TResult> (one input, returns value)
        // ─────────────────────────────────────────────────────────────────
        
        Func<int, int> square = x => x * x;
        // Line 2: Takes int, returns int
        Console.WriteLine($"5² = {square(5)}");  // Output: 5² = 25
        
        Func<string, int> getLength = s => s.Length;
        // Takes string, returns int
        Console.WriteLine($"Length: {getLength("Hello")}");  // Output: 5
        
        // ─────────────────────────────────────────────────────────────────
        // Func<T1, T2, TResult> (two inputs)
        // ─────────────────────────────────────────────────────────────────
        
        Func<int, int, int> add = (a, b) => a + b;
        // Line 3: Takes 2 ints, returns int
        Console.WriteLine($"3 + 4 = {add(3, 4)}");  // Output: 7
        
        Func<string, string, string> concat = (a, b) => a + " " + b;
        Console.WriteLine(concat("Hello", "World"));  // Output: Hello World
        
        // ─────────────────────────────────────────────────────────────────
        // Func in LINQ
        // ─────────────────────────────────────────────────────────────────
        
        int[] numbers = { 1, 2, 3, 4, 5 };
        
        // Select uses Func<T, TResult>
        var doubled = numbers.Select(x => x * 2);
        // Line 4: Lambda is Func<int, int>
        
        // Aggregate uses Func<T, T, T>
        int sum = numbers.Aggregate((acc, x) => acc + x);
        // Line 5: Lambda is Func<int, int, int>
    }
    
    static TResult Transform<T, TResult>(T input, Func<T, TResult> transformer)
    {
        return transformer(input);
    }
}
```

### Predicate<T> - Returns Boolean

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// PREDICATE<T> - RETURNS BOOL FOR CONDITION TESTING
// ═══════════════════════════════════════════════════════════════════════════

using System;
using System.Collections.Generic;

class PredicateDemo
{
    static void Main()
    {
        // Predicate<T> = Func<T, bool>
        // Used for conditions/filtering
        
        // ─────────────────────────────────────────────────────────────────
        // Basic Predicates
        // ─────────────────────────────────────────────────────────────────
        
        Predicate<int> isPositive = x => x > 0;
        // Line 1: Returns true if x > 0
        Console.WriteLine($"5 is positive: {isPositive(5)}");   // True
        Console.WriteLine($"-3 is positive: {isPositive(-3)}"); // False
        
        Predicate<string> isEmpty = s => string.IsNullOrEmpty(s);
        Console.WriteLine($"'' is empty: {isEmpty("")}");  // True
        
        Predicate<int> isEven = n => n % 2 == 0;
        Console.WriteLine($"4 is even: {isEven(4)}");  // True
        
        // ─────────────────────────────────────────────────────────────────
        // Predicate with Collections
        // ─────────────────────────────────────────────────────────────────
        
        List<int> numbers = new List<int> { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
        
        // Find - returns first match
        int firstEven = numbers.Find(isEven);
        // Line 2: Find uses Predicate<T>
        Console.WriteLine($"First even: {firstEven}");  // Output: 2
        
        // FindAll - returns all matches
        List<int> allEven = numbers.FindAll(isEven);
        Console.WriteLine($"All even: {string.Join(", ", allEven)}");
        // Output: 2, 4, 6, 8, 10
        
        // FindLast - returns last match
        int lastEven = numbers.FindLast(isEven);
        Console.WriteLine($"Last even: {lastEven}");  // Output: 10
        
        // FindIndex - returns index of first match
        int index = numbers.FindIndex(isEven);
        Console.WriteLine($"Index of first even: {index}");  // Output: 1
        
        // Exists - returns true if any match
        bool hasNegative = numbers.Exists(x => x < 0);
        Console.WriteLine($"Has negative: {hasNegative}");  // False
        
        // TrueForAll - returns true if all match
        bool allPositive = numbers.TrueForAll(x => x > 0);
        Console.WriteLine($"All positive: {allPositive}");  // True
        
        // RemoveAll - removes all matches
        int removed = numbers.RemoveAll(x => x > 5);
        Console.WriteLine($"Removed {removed} items");  // Removed 5 items
    }
}
```

### Comparison Summary

| Delegate | Signature | Use Case |
|----------|-----------|----------|
| `Action` | `void Method()` | Execute without return |
| `Action<T>` | `void Method(T arg)` | Process single item |
| `Action<T1, T2>` | `void Method(T1, T2)` | Process multiple items |
| `Func<TResult>` | `TResult Method()` | Get computed value |
| `Func<T, TResult>` | `TResult Method(T arg)` | Transform item |
| `Predicate<T>` | `bool Method(T arg)` | Test condition |

---

## 🔶 Anonymous Methods and Lambdas

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// ANONYMOUS METHODS AND LAMBDA EXPRESSIONS
// ═══════════════════════════════════════════════════════════════════════════

using System;

class LambdaDemo
{
    static void Main()
    {
        // ─────────────────────────────────────────────────────────────────
        // EVOLUTION OF DELEGATE SYNTAX
        // ─────────────────────────────────────────────────────────────────
        
        // 1. Named method (C# 1.0)
        Func<int, int> square1 = Square;
        
        // 2. Anonymous method (C# 2.0)
        Func<int, int> square2 = delegate(int x) { return x * x; };
        // Line 1: delegate keyword + parameters + body
        
        // 3. Lambda expression (C# 3.0)
        Func<int, int> square3 = (int x) => { return x * x; };
        // Line 2: arrow syntax with explicit types
        
        // 4. Lambda - type inference
        Func<int, int> square4 = (x) => { return x * x; };
        // Line 3: Compiler infers type from Func<int, int>
        
        // 5. Lambda - remove parentheses (single param)
        Func<int, int> square5 = x => { return x * x; };
        // Line 4: Single parameter doesn't need ()
        
        // 6. Expression lambda (single expression)
        Func<int, int> square6 = x => x * x;
        // Line 5: No braces, no return keyword
        //         Expression result is the return value
        
        // All 6 versions produce same result!
        
        // ─────────────────────────────────────────────────────────────────
        // LAMBDA EXPRESSION EXAMPLES
        // ─────────────────────────────────────────────────────────────────
        
        // No parameters
        Action sayHi = () => Console.WriteLine("Hi!");
        
        // One parameter
        Action<string> greet = name => Console.WriteLine($"Hello {name}");
        
        // Multiple parameters
        Func<int, int, int> add = (a, b) => a + b;
        
        // Multiple statements (need braces and return)
        Func<int, int> complex = x => 
        {
            int doubled = x * 2;
            int result = doubled + 10;
            return result;
        };
        
        // ─────────────────────────────────────────────────────────────────
        // CLOSURES - CAPTURING OUTER VARIABLES
        // ─────────────────────────────────────────────────────────────────
        
        int factor = 10;
        Func<int, int> multiplier = x => x * factor;
        // Line 6: Lambda captures 'factor' from outer scope
        
        Console.WriteLine(multiplier(5));  // Output: 50
        
        factor = 20;  // Change outer variable
        Console.WriteLine(multiplier(5));  // Output: 100 (uses current value!)
    }
    
    static int Square(int x) => x * x;
}
```

---

## 🎤 Interview Questions

**Q1: What is a delegate?**
> A delegate is a type-safe function pointer that holds a reference to a method with a specific signature.

**Q2: What is the difference between Action and Func?**
> `Action` is for methods that return void. `Func` is for methods that return a value (last type parameter is return type).

**Q3: What is Predicate<T>?**
> `Predicate<T>` is a delegate that takes one parameter and returns bool. Equivalent to `Func<T, bool>`. Used for filtering conditions.

**Q4: What is a multicast delegate?**
> A delegate that holds references to multiple methods. When invoked, all methods are called in order.

**Q5: What is a lambda expression?**
> A concise syntax for anonymous functions: `(parameters) => expression` or `(parameters) => { statements }`

**Q6: What is a closure?**
> When a lambda captures and uses variables from its outer scope. The variable's current value is used when lambda executes.

---

## 📝 Summary

| Concept | Syntax | Returns |
|---------|--------|---------|
| **Custom Delegate** | `delegate int MyDel(int x)` | Specified |
| **Action** | `Action<T>` | void |
| **Func** | `Func<T, TResult>` | TResult |
| **Predicate** | `Predicate<T>` | bool |
| **Lambda** | `x => x * 2` | Inferred |
| **Multicast** | `del += Method` | Last return |
