# 03 - Storage Classes

## Table of Contents
1. [Introduction](#introduction)
2. [Variable Scope Concepts](#variable-scope-concepts)
3. [The Four Storage Classes](#the-four-storage-classes)
4. [Automatic Storage Class](#automatic-storage-class)
5. [Static Storage Class](#static-storage-class)
6. [Register Storage Class](#register-storage-class)
7. [Extern Storage Class](#extern-storage-class)
8. [Memory Areas](#memory-areas)
9. [Comparison Table](#comparison-table)
10. [Practice Questions](#practice-questions)
11. [Interview Questions](#interview-questions)

---

## Introduction

### What are Storage Classes?

Storage classes define **four attributes** of a variable:

| Attribute | Description |
|-----------|-------------|
| **Default Initial Value** | What value the variable has if not explicitly initialized |
| **Memory Location** | Where the variable is stored in memory |
| **Scope** | Where the variable can be accessed from |
| **Lifetime** | How long the variable exists in memory |

### The Four Storage Classes in C/C++

```
┌──────────────────────────────────────────────────────────────────┐
│                   STORAGE CLASSES                                 │
├──────────────────────────────────────────────────────────────────┤
│  1. automatic (auto)    - Default for local variables           │
│  2. static              - Persists between function calls       │
│  3. register            - Request for CPU register storage      │
│  4. extern              - Access external variables             │
└──────────────────────────────────────────────────────────────────┘
```

---

## Variable Scope Concepts

### Local Variable Scope

By default, the scope of a variable is **limited to the block or function** in which it is defined.

```cpp
#include <stdio.h>

void main() {
    void disp();       // Function declaration
    int num1 = 9;      // Local to main()
    disp();
    printf("%d\n", num1);  // Can access num1 here
}

void disp() {
    // printf("%d\n", num1);  // ❌ ERROR! num1 not visible here
    puts("inside disp function");
}
```

### Line-by-Line Explanation

| Line | Explanation |
|------|-------------|
| `int num1 = 9;` | Declares local variable `num1` in main's scope |
| Comment in disp() | `num1` cannot be accessed - it's outside its scope |

### External Variable Scope

The default scope can be extended by defining a variable **outside any function block**.

```cpp
#include <stdio.h>

void main() {
    void disp();
    void disp1();
    disp();    // ❌ ERROR! num not defined yet
    disp1();
}

int num = 20;   // External variable definition

void disp() {
    printf("In disp\t%d\n", num);   // ✅ Can access num
}

void disp1() {
    printf("In disp1\t%d\n", num);  // ✅ Can access num
}
```

> **Note:** External variable is accessible to all functions defined **after its definition**.

### Global Variable

When an external variable is defined **before main()**, it becomes **global** to that file.

```cpp
int var = 20;  // Global variable - accessible everywhere in file

void main() {
    void disp();
    printf("var in main is %d\n", var);  // 20
    var = 400;       // Modify global variable
    disp();
}

void disp() {
    int k = 10;      // Local variable
    printf("%d\t%d\n", k, var);  // 10    400
}
```

### Local vs Global Variable Precedence

When local and global variables have the **same name**, **local takes precedence**.

```cpp
int val = 10;   // Global variable

void main() {
    int val = 20;   // Local variable (shadows global)
    int retVal();
    printf("%d\n", retVal());  // Prints 10 (uses global)
}

int retVal() {
    return val;     // Returns global val (10)
}
```

---

## Automatic Storage Class

### Syntax

```cpp
auto datatype variable;
// OR simply
datatype variable;   // auto is default
```

### Characteristics

| Property | Value |
|----------|-------|
| **Keyword** | `auto` (optional, it's the default) |
| **Default Initial Value** | Garbage (undefined) |
| **Memory Location** | Stack |
| **Scope** | Limited to the block in which it is defined |
| **Lifetime** | As long as control remains in that block |

### Code Example

```cpp
#include <stdio.h>

void main() {
    void disp();
    auto int num;        // Explicit auto (same as just 'int num;')
    printf("%d\n", num); // Prints GARBAGE value!
    disp();
}

void disp() {
    // printf("%d\n", num);  // ❌ ERROR: num out of scope
    puts("in disp");
}
```

### Line-by-Line Explanation

| Line | Code | What Happens |
|------|------|--------------|
| `auto int num;` | Declares automatic variable (not initialized) |
| `printf("%d\n", num);` | Prints garbage because auto vars aren't initialized |
| Comment in disp() | `num` is not accessible outside main() |

### Memory Diagram

```
Stack Memory:
┌─────────────────────────────────────┐
│  main() called:                     │
│  ┌─────────────┐                    │
│  │ num = ???   │ ← Garbage value    │
│  └─────────────┘                    │
│                                     │
│  disp() called:                     │
│  ┌─────────────┐                    │
│  │   (empty)   │ ← num not here     │
│  └─────────────┘                    │
└─────────────────────────────────────┘
```

---

## Static Storage Class

### Syntax

```cpp
static datatype variable;
```

### Characteristics

| Property | Value |
|----------|-------|
| **Keyword** | `static` |
| **Default Initial Value** | Zero (0) |
| **Memory Location** | Data Area (or BSS) |
| **Scope** | Limited to the block in which it is defined |
| **Lifetime** | Till the program execution ends |

### Key Feature: Value Persistence

**Static variables persist their values between different function calls!**

### Code Example

```cpp
#include <stdio.h>

void main() {
    int i;
    void disp();
    for(i = 0; i < 3; i++) {
        disp();
    }
}

void disp() {
    int a = 0;           // Automatic - reinitialized every call
    static int b;        // Static - initialized once, persists
    printf("%d\t%d\n", a++, b++);
}
```

### Output

```
0       0
0       1
0       2
```

### Line-by-Line Explanation

| Call # | `a` starts | `a++` prints | `b` starts | `b++` prints |
|--------|------------|--------------|------------|--------------|
| 1 | 0 (new) | 0 | 0 (first time) | 0 |
| 2 | 0 (new) | 0 | 1 (persisted) | 1 |
| 3 | 0 (new) | 0 | 2 (persisted) | 2 |

### Execution Flow

```
First call to disp():
┌───────────────────────────────────────────────────────────────┐
│ 1. int a = 0;        → a is created and set to 0             │
│ 2. static int b;     → b is created ONCE, initialized to 0   │
│ 3. printf: a++ (0), b++ (0)                                  │
│ 4. After: a=1 (destroyed), b=1 (persists)                    │
└───────────────────────────────────────────────────────────────┘

Second call to disp():
┌───────────────────────────────────────────────────────────────┐
│ 1. int a = 0;        → NEW a created, set to 0               │
│ 2. static int b;     → SAME b from before (value = 1)        │
│ 3. printf: a++ (0), b++ (1)                                  │
│ 4. After: a=1 (destroyed), b=2 (persists)                    │
└───────────────────────────────────────────────────────────────┘

Third call to disp():
┌───────────────────────────────────────────────────────────────┐
│ 1. int a = 0;        → NEW a created, set to 0               │
│ 2. static int b;     → SAME b from before (value = 2)        │
│ 3. printf: a++ (0), b++ (2)                                  │
│ 4. After: a=1 (destroyed), b=3 (persists for next call)      │
└───────────────────────────────────────────────────────────────┘
```

### Use Cases for Static Variables

1. **Counting function calls**
2. **Caching expensive computations**
3. **Maintaining state between calls**
4. **Singleton pattern implementation**

```cpp
int getNextId() {
    static int id = 0;  // Initialized only once!
    return ++id;        // Returns 1, 2, 3, ... on each call
}
```

---

## Register Storage Class

### Syntax

```cpp
register datatype variable;
```

### Characteristics

| Property | Value |
|----------|-------|
| **Keyword** | `register` |
| **Default Initial Value** | Garbage (undefined) |
| **Memory Location** | CPU Register (if available) |
| **Scope** | Limited to the block in which it is defined |
| **Lifetime** | As long as control remains in that block |

### Key Points

1. **Faster Access:** CPU registers are much faster than RAM
2. **Limited Availability:** CPU has limited number of registers
3. **Mere Request:** The `register` keyword is a **request, not a command**
4. **Fallback:** If no register is available, variable becomes automatic

### Code Example

```cpp
#include <stdio.h>

void main() {
    register int i;   // Request to store in CPU register
    for(i = 0; i < 4; i++) {
        printf("%d\n", i);
    }
}
```

### Line-by-Line Explanation

| Line | Code | Explanation |
|------|------|-------------|
| `register int i;` | Requests compiler to store `i` in CPU register for fast access |
| `for(i = 0; ...)` | Loop counter is ideal for register storage (accessed frequently) |

### When to Use Register

```cpp
// GOOD use of register - frequently accessed loop counter
register int i;
for(i = 0; i < 1000000; i++) {
    // Accessing i millions of times
}

// NOT useful - infrequently accessed variable
register int config = 10;  // Only used once or twice
```

> **Modern Note:** Modern compilers often ignore the `register` keyword and optimize register usage automatically.

---

## Extern Storage Class

### Syntax

```cpp
extern datatype variable;   // Declaration (no memory allocated)
datatype variable = value;  // Definition (memory allocated)
```

### Characteristics

| Property | Value |
|----------|-------|
| **Keyword** | `extern` |
| **Default Initial Value** | Zero (0) |
| **Memory Location** | Data Area (or BSS) |
| **Scope** | Global (entire program) |
| **Lifetime** | Till program execution ends |

### Key Concept: Declaration vs Definition

| Aspect | Declaration | Definition |
|--------|-------------|------------|
| Purpose | Tells compiler the variable exists | Creates the variable |
| Memory | No memory allocated | Memory is allocated |
| Keyword | Uses `extern` | No `extern` (or initializes value) |
| Count | Can have many | Must have exactly one |

### Code Example: Using Before Definition

```cpp
#include <stdio.h>

void main() {
    int num1 = 20;
    extern int num2;      // Declaration: "num2 exists somewhere"
    void disp();
    
    printf("%d\n", num1); // 20
    printf("%d\n", num2); // 40
    disp();
}

int num2 = 40;            // Definition: memory allocated here

void disp() {
    printf("%d\n", num2); // 40
}
```

### Execution Flow

```
┌───────────────────────────────────────────────────────────────┐
│ 1. main() starts                                              │
│ 2. num1 = 20 (local variable created)                        │
│ 3. extern int num2; → Compiler notes num2 exists somewhere   │
│ 4. printf(num1) → Prints 20                                  │
│ 5. printf(num2) → Looks up num2, finds it defined later (40) │
│ 6. disp() called → Also accesses num2 (40)                   │
└───────────────────────────────────────────────────────────────┘
```

### Cross-File Usage

**first.c**
```cpp
#include <stdio.h>

void fun() {
    extern int cnt;           // Declare: cnt exists elsewhere
    puts("in fun");
    printf("%d\n", cnt);      // Prints 100
}
```

**second.c**
```cpp
#include "first.c"

int cnt = 100;                // Define: cnt = 100

void main() {
    printf("in main %d\n", cnt);  // Prints 100
    fun();
}
```

### Execution Flow for Cross-File

```
┌───────────────────────────────────────────────────────────────┐
│ 1. second.c includes first.c                                 │
│ 2. int cnt = 100; → cnt is DEFINED with value 100            │
│ 3. main() prints cnt → 100                                   │
│ 4. fun() uses extern cnt → finds cnt = 100 → prints 100      │
└───────────────────────────────────────────────────────────────┘
```

---

## Memory Areas

### Where Variables Are Stored

```
┌─────────────────────────────────────────────────────────────────┐
│                    PROGRAM MEMORY LAYOUT                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│    HIGH ADDRESS                                                 │
│    ┌───────────────────────────────┐                           │
│    │           STACK               │ ← auto, register variables│
│    │   (Grows Downward ↓)          │   function parameters     │
│    │                               │   return addresses        │
│    ├───────────────────────────────┤                           │
│    │                               │                           │
│    │           HEAP                │ ← Dynamic allocation      │
│    │   (Grows Upward ↑)            │   (malloc, new)           │
│    │                               │                           │
│    ├───────────────────────────────┤                           │
│    │            BSS                │ ← Uninitialized static    │
│    │   (Block Started by Symbol)   │   and global variables    │
│    ├───────────────────────────────┤                           │
│    │           DATA                │ ← Initialized static      │
│    │  (Initialized Data Segment)   │   and global variables    │
│    ├───────────────────────────────┤                           │
│    │           TEXT                │ ← Program code            │
│    │      (Code Segment)           │   (read-only)             │
│    └───────────────────────────────┘                           │
│    LOW ADDRESS                                                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Storage Class to Memory Mapping

| Storage Class | Memory Area |
|---------------|-------------|
| `auto` | Stack |
| `static` | Data / BSS |
| `register` | CPU Register (or Stack) |
| `extern` | Data / BSS |

---

## Comparison Table

| Property | auto | static | register | extern |
|----------|------|--------|----------|--------|
| **Keyword** | auto (optional) | static | register | extern |
| **Storage** | Stack | Data/BSS | CPU Register* | Data/BSS |
| **Default Value** | Garbage | 0 | Garbage | 0 |
| **Scope** | Block | Block | Block | Global |
| **Lifetime** | Block execution | Program execution | Block execution | Program execution |
| **Initialization** | Every call | Once | Every call | Once |

*If CPU register not available, treated as auto

---

## Practice Questions

### Question 1: Predict the Output

```cpp
#include <stdio.h>

void fun() {
    static int x = 5;
    x++;
    printf("%d ", x);
}

int main() {
    fun();
    fun();
    fun();
    return 0;
}
```

**Answer:** `6 7 8`

**Explanation:**
- First call: x = 5, x++ makes it 6, prints 6
- Second call: x persists as 6, x++ makes it 7, prints 7
- Third call: x persists as 7, x++ makes it 8, prints 8

### Question 2: What's the difference between declaration and definition?

**Answer:**
- **Declaration:** Tells compiler about variable (no memory)
- **Definition:** Creates variable and allocates memory

```cpp
extern int x;    // Declaration
int x = 10;      // Definition
```

### Question 3: Why use static for local variables?

**Answer:** To maintain state between function calls, such as:
- Counting invocations
- Caching computed values
- Implementing counter functions

---

## Interview Questions

### Q1: What is the default storage class in C/C++?

**Answer:** `auto` (automatic). Any variable declared inside a function without a storage class specifier is automatically `auto`.

### Q2: Can we take the address of a register variable?

**Answer:** In C++, no, because the variable might be in a CPU register which doesn't have a memory address. This is another reason modern compilers often ignore the `register` keyword.

```cpp
register int x = 5;
int *p = &x;  // ❌ Error: cannot take address of register variable
```

### Q3: What happens if we don't initialize a static variable?

**Answer:** Static variables are automatically initialized to **0** (zero). This is guaranteed by the C/C++ standards.

```cpp
static int x;   // x is automatically 0
static int y;   // y is automatically 0
```

### Q4: What is the difference between static local and static global variables?

| Static Local | Static Global |
|--------------|---------------|
| Declared inside a function | Declared outside all functions |
| Scope limited to that function | Scope limited to that file |
| Preserves value between calls | Accessible by all functions in file |

### Q5: Why is extern needed?

**Answer:** `extern` is needed when:
1. You want to use a global variable **before** its definition in the same file
2. You want to access a variable **defined in another file**

---

## Summary

| Concept | Key Points |
|---------|------------|
| **Storage Classes** | Define initial value, memory location, scope, and lifetime |
| **auto** | Default, stored on stack, garbage value, block scope |
| **static** | Persists between calls, stored in data area, initialized to 0 |
| **register** | Request for CPU register, may be ignored, for frequently used variables |
| **extern** | For accessing variables defined elsewhere, declaration vs definition |
| **Memory Areas** | Stack (auto), Data/BSS (static, extern), CPU Register (register) |

---

> **Next Topic:** [04 - Preprocessor Directives](./04_Preprocessor_Directives.md)
