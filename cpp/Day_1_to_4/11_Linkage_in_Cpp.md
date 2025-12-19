# 11 - Linkage in C++

## Table of Contents
1. [Introduction](#introduction)
2. [No Linkage](#no-linkage)
3. [External Linkage](#external-linkage)
4. [Internal Linkage](#internal-linkage)
5. [Type-Safe Linkage](#type-safe-linkage)
6. [Alternate Linkage (extern "C")](#alternate-linkage)
7. [Summary](#summary)

---

## Introduction

### What is Linkage?

**Linkage** describes how names (variables/functions) can or cannot refer to the same entity throughout the whole program or within a single translation unit (one .cpp file).

### Types of Linkage

| Type | Description |
|------|-------------|
| **No Linkage** | Local variables (stack-based), not visible to linker |
| **External Linkage** | Visible across all translation units |
| **Internal Linkage** | Visible only within its translation unit |
| **Type-Safe Linkage** | C++ name mangling for function overloads |
| **Alternate Linkage** | `extern "C"` for C compatibility |

---

## No Linkage

### What Has No Linkage?

**Automatic (local) variables** have no linkage. They:
- Exist only on the stack
- Are not visible to the linker
- Are only accessible within their block

```cpp
void func() {
    int localVar = 10;  // No linkage
    // localVar dies when func returns
}
```

---

## External Linkage

### Definition

**External linkage** means a single piece of storage is created to represent the identifier for all files being compiled.

### Key Points

| Point | Description |
|-------|-------------|
| **Default for** | Global variables, function definitions |
| **Accessible from** | Any file (translation unit) |
| **Declaration** | Use `extern` keyword |
| **Storage** | Created once, shared everywhere |

### Example: Cross-File Access

**First.cpp**
```cpp
#include <iostream>
using namespace std;

void disp() {
    extern int num;    // Declare: num exists elsewhere
    cout << "in disp\t" << num << endl;
    
    void fun();        // Functions are extern by default
    cout << "from disp calling fun\t";
    fun();
}
```

**Second.cpp**
```cpp
#include <iostream>
using namespace std;
#include "First.cpp"

int num = 100;         // Definition: memory allocated here

void fun() {
    cout << "in fun" << endl;
}

int main() {
    cout << "in main\t" << num << endl;
    disp();
    cout << "from main calling fun\t";
    fun();
}
```

### Execution Flow

```
┌───────────────────────────────────────────────────────────────┐
│ 1. num is DEFINED in Second.cpp with value 100               │
│ 2. num is DECLARED in First.cpp with extern                  │
│ 3. Linker connects the declaration to the definition         │
│ 4. Both files access the SAME num variable                   │
└───────────────────────────────────────────────────────────────┘

Output:
in main    100
in disp    100
from disp calling fun    in fun
from main calling fun    in fun
```

---

## Internal Linkage

### Definition

**Internal linkage** means storage is created for the identifier only within its translation unit. Other files can use the same name without conflict.

### Key Points

| Point | Description |
|-------|-------------|
| **Keyword** | `static` (on global variables/functions) |
| **Scope** | Only within the file |
| **Name conflicts** | None across files |

### Example: static Prevents External Access

**First.cpp**
```cpp
#include <iostream>
using namespace std;

void disp() {
    extern int num;    // Try to access external num
    cout << "in disp\t" << num << endl;
    void fun();
    fun();
}
```

**Second.cpp**
```cpp
#include <iostream>
using namespace std;
#include "First.cpp"

static int num = 100;    // Internal linkage! NOT visible outside
static void fun() {      // Internal linkage!
    cout << "in fun" << endl;
}

int main() {
    cout << "in main\t" << num << endl;
    disp();             // ❌ LINKER ERROR!
}
```

### Result

```
LINKER ERROR: unresolved external symbol "num" and "fun"

Because: static makes them have INTERNAL linkage only.
First.cpp cannot access them.
```

### Visual Comparison

```
External Linkage:                    Internal Linkage:
─────────────────────────────────    ─────────────────────────────────
File1.cpp      File2.cpp             File1.cpp      File2.cpp
    │              │                     │              │
    │    int num   │                     │  static int  │
    └──────┬───────┘                     │    num       │
           │                             │              │
     (same variable)                (separate variables)
           │                             │              │
         Linker                        Linker sees
        connects                       no connection
```

---

## Type-Safe Linkage

### What is Type-Safe Linkage?

C++ uses **name mangling** (name decoration) to encode function signatures into their names. This prevents calling functions with wrong argument types.

### Problem in C

```c
// Library (C)
void disp(char *ptr) {
    printf("%c\n", *ptr);
}

// Client (C) - WRONG declaration
void disp(double);  // Wrong but compiles!

void main() {
    disp(23.7);      // Compiles and links! (Wrong!)
                     // → RUNTIME ERROR!
}
```

**In C:** Linker only checks function NAME, not arguments.

### Solution in C++

```cpp
// Library (C++)
void disp(char *ptr) {
    cout << *ptr << endl;
}
// Mangled name: disp_char_ptr or similar

// Client (C++) - WRONG declaration
void disp(double);  // Mangled as: disp_double

int main() {
    disp(4.5);  // Mangled call: disp_double(4.5)
}
```

**Result:** LINKER ERROR!

```
Linker searches for: disp_double(double)
Library has: disp_char_ptr(char*)
→ No match → Linker error → SAFE!
```

### How Name Mangling Works

```cpp
void disp(char* ptr)  →  _Z4dispPc       (example mangled name)
void disp(double d)   →  _Z4dispd        (different mangled name)
void disp(int a, int b) → _Z4dispii      (another mangled name)
```

**Benefit:** Wrong declarations are caught at LINK time, not at RUN time.

---

## Alternate Linkage

### extern "C"

When you need to call C library functions from C++, you must disable name mangling.

### Syntax

```cpp
extern "C" {
    // C function declarations here
    void c_function(int x);
}

// Or for single function:
extern "C" void c_function(int x);
```

### Why Needed?

| Compiled as C | Compiled as C++ |
|---------------|-----------------|
| `disp` | `_Z4dispPc` |

C++ mangled name won't match C library's unmangled name!

### Example: Calling C from C++

```cpp
// C++ file calling C library function
extern "C" {
    void c_library_func(int x);  // Don't mangle this name
}

int main() {
    c_library_func(10);  // Linker looks for "c_library_func", not mangled
    return 0;
}
```

---

## Summary

| Linkage Type | Keyword | Scope | Use Case |
|--------------|---------|-------|----------|
| **No Linkage** | (local vars) | Block only | Temporary variables |
| **External** | (default) or `extern` | All files | Shared globals, functions |
| **Internal** | `static` | Single file | File-private implementation |
| **Type-Safe** | (automatic in C++) | - | Prevent signature mismatch |
| **Alternate** | `extern "C"` | - | Call C libraries from C++ |

---

> **Next Topic:** [12 - Structs in C++](./12_Structs_in_Cpp.md)
