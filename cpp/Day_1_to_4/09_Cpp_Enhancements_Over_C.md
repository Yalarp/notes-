# 09 - C++ Enhancements Over C

## Table of Contents
1. [Introduction](#introduction)
2. [cin and cout](#cin-and-cout)
3. [Scope Resolution Operator](#scope-resolution-operator)
4. [Variable Declaration Anywhere](#variable-declaration-anywhere)
5. [Default Arguments](#default-arguments)
6. [Summary of Enhancements](#summary-of-enhancements)

---

## Introduction

C++ provides many enhancements over C. This note covers the key differences.

### List of Enhancements

| Enhancement | C | C++ |
|-------------|---|-----|
| Input/Output | scanf/printf | cin/cout |
| Scope Resolution | Not available | `::` operator |
| Variable Declaration | Top of block only | Anywhere |
| Default Arguments | Not available | Supported |
| Function Overloading | Not available | Supported |
| Inline Functions | Not available | Supported |
| References | Not available | Supported |
| new/delete | Not available | Supported |
| const as true constant | No | Yes |

---

## cin and cout

### What are cin and cout?

| Object | Type | Purpose | Operator |
|--------|------|---------|----------|
| `cin` | istream | Input | `>>` (extraction) |
| `cout` | ostream | Output | `<<` (insertion) |

### Comparison with C

| Feature | C (scanf/printf) | C++ (cin/cout) |
|---------|------------------|----------------|
| Format specifiers | Required (`%d`, `%s`) | Not needed |
| Address operator | Required (`&`) for input | Not needed for basic types |
| Header | `<stdio.h>` | `<iostream>` |
| Type safety | No | Yes |

### Code Example

```cpp
/*
cin and cout are objects of istream and ostream classes respectively.
No "&" and "format specifiers" required.
cin needs >> (extraction operator)
cout needs << (insertion operator)
*/

#include <iostream>
using namespace std;

int main() {
    char name[20];
    int age;
    
    cout << "Enter name and age" << endl;
    cin >> name >> age;              // No & needed for name, no & for age either
    
    cout << name << "\t" << age << endl;
    
    return 0;
}
```

### Line-by-Line Explanation

| Line | Code | Explanation |
|------|------|-------------|
| `cin >> name >> age;` | Reads name into array, age into int (no format specifiers) |
| `cout << name << age;` | Outputs both values (no format specifiers) |

---

## Scope Resolution Operator

### What is `::`?

The **scope resolution operator** (`::`) is used to access global variables when a local variable has the same name.

### Syntax

```cpp
::variable    // Access global variable
```

### Code Example

```cpp
#include <iostream>
using namespace std;

int val = 100;   // Global variable

int main() {
    int val = 20;    // Local variable (shadows global)
    
    cout << endl << val << endl;      // Prints 20 (local)
    cout << endl << ::val << endl;    // Prints 100 (global)
}
```

### Execution Flow

```
┌───────────────────────────────────────────────────────────────┐
│ Global scope: val = 100                                       │
├───────────────────────────────────────────────────────────────┤
│ Local scope (main): val = 20                                  │
├───────────────────────────────────────────────────────────────┤
│ cout << val      → Uses local (20)                           │
│ cout << ::val    → Uses global (100)                         │
└───────────────────────────────────────────────────────────────┘
```

---

## Variable Declaration Anywhere

### C vs C++

| C | C++ |
|---|-----|
| Variables must be declared at the top of a block | Variables can be declared anywhere |

### C++ Example

```cpp
int main() {
    cout << "Starting" << endl;
    
    int x = 10;     // Declared where needed
    
    for(int i = 0; i < 5; i++) {   // i declared in for loop
        cout << i << endl;
    }
    
    int y = 20;     // Declared later in function
    
    return 0;
}
```

---

## Default Arguments

### What are Default Arguments?

Parameters that have a default value if not provided by the caller.

### Rules

1. Default arguments must be from **right to left**
2. Once a default is provided, all parameters to its right must also have defaults

### Example

```cpp
#include <iostream>
using namespace std;

void display(int a, int b = 10, int c = 20) {
    cout << a << ", " << b << ", " << c << endl;
}

int main() {
    display(5);         // Output: 5, 10, 20
    display(5, 15);     // Output: 5, 15, 20
    display(5, 15, 25); // Output: 5, 15, 25
    return 0;
}
```

---

## Summary of Enhancements

| Feature | Description |
|---------|-------------|
| **cin/cout** | Type-safe I/O without format specifiers |
| **::** | Access global when shadowed by local |
| **Variable declaration** | Declare variables anywhere |
| **Default arguments** | Provide default parameter values |
| **References** | Alias for variables (see Note 10) |
| **new/delete** | Operators for dynamic memory |
| **Function overloading** | Same name, different parameters |
| **Inline functions** | Inlined for performance |

---

> **Next Topic:** [10 - References Complete Guide](./10_References_Complete_Guide.md)
