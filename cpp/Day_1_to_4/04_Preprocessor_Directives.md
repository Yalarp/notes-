# 04 - Preprocessor Directives

## Table of Contents
1. [Introduction](#introduction)
2. [#include Directive](#include-directive)
3. [#define Directive](#define-directive)
4. [Conditional Compilation](#conditional-compilation)
5. [#pragma Directive](#pragma-directive)
6. [How to View Preprocessed File](#how-to-view-preprocessed-file)
7. [Practice Questions](#practice-questions)
8. [Interview Questions](#interview-questions)

---

## Introduction

### What is a Preprocessor?

The **preprocessor** is a program that processes source code **before** compilation. It handles special directives that begin with `#`.

### Preprocessor Directives

| Directive | Purpose |
|-----------|---------|
| `#include` | Include header files |
| `#define` | Define macros/constants |
| `#ifdef` / `#ifndef` | Conditional compilation |
| `#if` / `#elif` / `#else` / `#endif` | Conditional compilation |
| `#pragma` | Compiler-specific instructions |
| `#undef` | Undefine a macro |
| `#error` | Generate compile-time error |
| `#warning` | Generate compile-time warning |

### When Does Processing Happen?

```
┌─────────────────────────────────────────────────────────────────┐
│                    PROCESSING ORDER                             │
└─────────────────────────────────────────────────────────────────┘

         Source.cpp
              │
              ▼
    ┌─────────────────┐
    │   PREPROCESSOR  │ ← Handles #include, #define, #ifdef, etc.
    │  (runs FIRST)   │
    └─────────────────┘
              │
              ▼
         Source.i (expanded)
              │
              ▼
    ┌─────────────────┐
    │    COMPILER     │ ← Converts to assembly
    └─────────────────┘
              │
              ▼
         Assembly/Object code
```

---

## #include Directive

### Purpose

Includes the contents of a header file at the point of the directive.

### Two Forms

```cpp
#include <filename>    // System headers (looks in system directories)
#include "filename"    // User headers (looks in current directory first)
```

### Example

```cpp
#include <iostream>    // Standard library header
#include "myheader.h"  // User-defined header
```

### How It Works

```
Before preprocessing:           After preprocessing:
─────────────────────           ────────────────────
#include <iostream>             [Contents of iostream]
                                [thousands of lines...]
int main() {                    int main() {
    ...                             ...
}                               }
```

---

## #define Directive

### Purpose

Creates **macros** - symbolic names that are replaced by the preprocessor.

### Syntax

```cpp
#define MACRO_NAME value           // Object-like macro
#define MACRO_NAME(args) expression // Function-like macro
```

---

### Example 1: Constant Definition

```cpp
#include <iostream>
using namespace std;

#define SIZE 5                     // Define constant

int main() {
    int arr[SIZE] = {10, 20, 30, 40, 50};  // SIZE replaced with 5
    int i;
    
    for(i = 0; i < SIZE; i++) {    // SIZE replaced with 5
        cout << arr[i] << endl;
    }
    return 0;
}
```

### Line-by-Line Explanation

| Line | Code | What Preprocessor Does |
|------|------|------------------------|
| `#define SIZE 5` | Tells preprocessor: replace "SIZE" with "5" |
| `int arr[SIZE]` | Becomes `int arr[5]` |
| `i < SIZE` | Becomes `i < 5` |

---

### Example 2: Function-like Macro

```cpp
#include <iostream>
using namespace std;

#define sqr(x) x*x                 // Macro with parameter

int main() {
    int num = 10;
    cout << sqr(num) << endl;      // Becomes: num*num → 10*10 → 100
    return 0;
}
```

**Output:** `100`

### How It Expands

```
Before preprocessing:              After preprocessing:
──────────────────────             ─────────────────────
cout << sqr(num) << endl;   →     cout << num*num << endl;
```

---

### Example 3: Multi-Statement Macro (Drawback)

```cpp
#include <iostream>
using namespace std;

#define ADD_TWO(x, y) x += 2; y += 2;

int main() {
    int var = 10;
    int j = 5, k = 7;
    
    // PROBLEM: This doesn't work as expected!
    if(!var)
        ADD_TWO(j, k);   // Only first statement is controlled by if!
    
    cout << j << "\t" << k << endl;  // Prints: 5    9
    return 0;
}
```

### The Problem Explained

```cpp
// What programmer wrote:
if(!var)
    ADD_TWO(j, k);

// What preprocessor creates:
if(!var)
    j += 2; k += 2;    // Only j+=2 is inside if! k+=2 runs always!

// Correct way using braces:
if(!var) {
    ADD_TWO(j, k);
}
// Expands to:
if(!var) {
    j += 2; k += 2;    // Both inside the block
}
```

> **Lesson:** Macros can have subtle bugs. Prefer `inline` functions in C++.

---

## Conditional Compilation

### Purpose

Include or exclude code based on conditions at **compile time**.

### Directives

| Directive | Meaning |
|-----------|---------|
| `#ifdef MACRO` | If MACRO is defined |
| `#ifndef MACRO` | If MACRO is NOT defined |
| `#if expression` | If expression is true (non-zero) |
| `#elif expression` | Else if expression is true |
| `#else` | Otherwise |
| `#endif` | End of conditional block |

---

### Example 1: #ifdef

```cpp
#include <iostream>
using namespace std;

#define SIZE 5              // SIZE is defined

int main() {
    #ifdef SIZE             // Check if SIZE is defined
        cout << "SIZE is defined" << endl;
        cout << "hello" << endl;
    #else                   // If SIZE is NOT defined
        cout << "SIZE is not defined" << endl;
        cout << "welcome" << endl;
    #endif                  // End conditional block
    
    return 0;
}
```

**Output:**
```
SIZE is defined
hello
```

### Line-by-Line Explanation

| Line | What Happens |
|------|--------------|
| `#define SIZE 5` | SIZE is now defined |
| `#ifdef SIZE` | True! CODE between #ifdef and #else is KEPT |
| `#else` block | REMOVED by preprocessor |
| `#endif` | Marks end of conditional |

---

### Example 2: #ifndef (Define if Not Defined)

```cpp
#include <iostream>
using namespace std;

int main() {
    #ifndef SIZE           // If SIZE is NOT defined
        #define SIZE 5     // Define it
    #endif
    
    int i;
    int arr[SIZE] = {10, 20, 30, 40, 50};
    
    for(i = 0; i < SIZE; i++) {
        cout << arr[i] << endl;
    }
    return 0;
}
```

### Use Case: Header Guards

The most common use of `#ifndef` is **header guards**:

```cpp
// myheader.h
#ifndef MYHEADER_H         // If not already included
#define MYHEADER_H         // Mark as included

// Header content goes here
class MyClass {
    // ...
};

#endif                     // End guard
```

This prevents the same header from being included multiple times.

---

## #pragma Directive

### Purpose

`#pragma` provides **compiler-specific instructions**. Different compilers may support different pragmas.

### Common Pragmas

| Pragma | Purpose |
|--------|---------|
| `#pragma once` | Include header only once (alternative to header guards) |
| `#pragma pack(n)` | Set structure packing alignment |
| `#pragma warning` | Control compiler warnings |
| `#pragma comment` | Add library/linker comments |

---

### #pragma pack Example

```cpp
#include <iostream>
using namespace std;

#pragma pack(1)            // Pack structures tightly (1-byte alignment)

struct mystruct {
    char ch = 'A';         // 1 byte
    double d = 6.7;        // 8 bytes
} m;

int main() {
    cout << m.ch << "\t" << m.d << endl;
    cout << sizeof(m) << endl;    // 9 (with pack(1))
}
```

### Without #pragma pack(1)

```
Normal alignment:
┌─────┬─────────────────────────────────────────────┐
│ ch  │ [padding - 7 bytes] │        d             │
│ 1B  │       7 bytes       │       8 bytes        │
└─────┴─────────────────────────────────────────────┘
sizeof = 16 bytes (due to padding)

With #pragma pack(1):
┌─────┬─────────────────────────────────────────────┐
│ ch  │                    d                        │
│ 1B  │                  8 bytes                    │
└─────┴─────────────────────────────────────────────┘
sizeof = 9 bytes (no padding)
```

### When to Use #pragma pack

| Use Case | Recommended |
|----------|-------------|
| Memory-critical applications | Yes |
| Network protocols | Yes (exact byte layout) |
| General purpose | No (slower access) |

> **Warning:** Packed structures may cause slower memory access on some architectures.

---

## How to View Preprocessed File

### Purpose

See what the preprocessor produces (expanded macros, included headers).

### Command Line (GCC/G++)

```bash
g++ -E source.cpp -o source.i
```

| Flag | Meaning |
|------|---------|
| `-E` | Stop after preprocessing |
| `-o source.i` | Output to source.i file |

### Visual Studio

```
1. Right-click project → Properties
2. C/C++ → Preprocessor
3. Generate Preprocessed File → Yes
4. Build the project
5. Check the .i file in output directory
```

### Example

**Source file (ConsoleApplication1.cpp):**
```cpp
#include <iostream>
using namespace std;
int main() {
    cout << "Hello" << endl;
    return 0;
}
```

**Preprocessed file (ConsoleApplication1.i):**
```cpp
// Thousands of lines from <iostream>...
// Type definitions, function declarations, etc.

int main() {
    cout << "Hello" << endl;
    return 0;
}
```

The `.i` file can be **hundreds of thousands of lines** because all the standard library headers are expanded!

---

## Practice Questions

### Question 1: Define a Macro for MAX of Two Numbers

```cpp
#include <iostream>
using namespace std;

#define MAX(a, b) ((a) > (b) ? (a) : (b))

int main() {
    int x = 10, y = 20;
    cout << "MAX: " << MAX(x, y) << endl;  // 20
    return 0;
}
```

> **Note:** Extra parentheses prevent operator precedence issues!

### Question 2: Predict the Output

```cpp
#include <iostream>
using namespace std;

#define MULTI(a) a * a

int main() {
    cout << MULTI(2 + 3) << endl;
    return 0;
}
```

**Answer:** `11` (not 25!)

**Explanation:**
```
MULTI(2 + 3) expands to: 2 + 3 * 2 + 3
                       = 2 + 6 + 3
                       = 11
```

**Fix:** Use parentheses: `#define MULTI(a) ((a) * (a))`

---

## Interview Questions

### Q1: What is the difference between #include <> and #include ""?

| Form | Search Order |
|------|--------------|
| `#include <file>` | System directories first |
| `#include "file"` | Current directory first, then system directories |

### Q2: What are header guards?

**Answer:** Header guards prevent multiple inclusion of the same header file:

```cpp
#ifndef HEADER_H
#define HEADER_H
// contents
#endif
```

Or use `#pragma once` (simpler but compiler-specific).

### Q3: Why prefer inline functions over macros?

| Macro | Inline Function |
|-------|-----------------|
| No type checking | Type-safe |
| Text replacement | Proper compilation |
| Can have side effects | No hidden bugs |
| Harder to debug | Can be debugged |

### Q4: What does #pragma pack do?

**Answer:** It controls the alignment of structure members in memory. `#pragma pack(1)` removes padding, making structures compact but potentially slower to access.

### Q5: What is conditional compilation used for?

**Answer:**
1. Platform-specific code (`#ifdef WINDOWS`)
2. Debug/Release builds (`#ifdef DEBUG`)
3. Feature toggles
4. Header guards
5. Optional features

---

## Summary

| Concept | Key Points |
|---------|------------|
| **Preprocessor** | Runs before compiler, handles # directives |
| **#include** | Inserts file contents |
| **#define** | Creates macros (use parentheses!) |
| **#ifdef/#ifndef** | Conditional compilation based on definitions |
| **#pragma pack** | Controls structure alignment |
| **Header Guards** | Prevent multiple inclusion |

---

> **Next Topic:** [05 - Dynamic Memory Allocation](./05_Dynamic_Memory_Allocation.md)
