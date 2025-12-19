# 06 - Static and Dynamic Libraries

## Table of Contents
1. [Introduction](#introduction)
2. [Static Libraries (.lib)](#static-libraries-lib)
3. [Dynamic Libraries (.dll)](#dynamic-libraries-dll)
4. [Comparison](#comparison)
5. [Creating and Using Libraries in Visual Studio](#creating-and-using-libraries)
6. [dllexport and dllimport](#dllexport-and-dllimport)
7. [Practice Questions](#practice-questions)
8. [Interview Questions](#interview-questions)

---

## Introduction

### What are Libraries?

A **library** is a collection of pre-compiled code that can be reused across multiple programs. Libraries help:

- Reduce code duplication
- Improve maintainability
- Enable code sharing
- Hide implementation details

### Types of Libraries

| Type | Extension | Linking |
|------|-----------|---------|
| **Static Library** | `.lib` (Windows), `.a` (Linux) | At compile time |
| **Dynamic Library** | `.dll` (Windows), `.so` (Linux) | At runtime |

---

## Static Libraries (.lib)

### Characteristics

| Feature | Description |
|---------|-------------|
| **Linking** | At compile time |
| **Included in EXE** | Yes (becomes part of executable) |
| **File Size** | Larger executable |
| **Update** | Recompile needed |
| **Runtime Dependencies** | None |

### Visual Representation

```
┌─────────────────────────────────────────────────────────────────┐
│                   STATIC LINKING                                │
└─────────────────────────────────────────────────────────────────┘

         Compile Time:
         
    ┌──────────────┐     ┌──────────────┐
    │  Your Code   │     │  Library     │
    │  (main.cpp)  │     │  (math.lib)  │
    └──────────────┘     └──────────────┘
           │                    │
           └────────┬───────────┘
                    │
                    ▼
            ┌──────────────┐
            │    LINKER    │
            └──────────────┘
                    │
                    ▼
            ┌──────────────┐
            │  Program.exe │  ← Library code COPIED into EXE
            │  (Contains   │
            │  library     │
            │  code)       │
            └──────────────┘
         
         Runtime:
         
    ┌──────────────┐
    │  Program.exe │  ← Runs independently, no external files needed
    │  (Self-      │
    │  contained)  │
    └──────────────┘
```

### Creating a Static Library in Visual Studio

```
Step 1: Create Static Library Project
──────────────────────────────────────
1. File → New → Project
2. Select "Static Library (C++)"
3. Name: MyMathLib
4. Create

Step 2: Add Your Code
──────────────────────────────────────
// MyMath.h
#pragma once
int add(int a, int b);
int subtract(int a, int b);

// MyMath.cpp
#include "MyMath.h"
int add(int a, int b) { return a + b; }
int subtract(int a, int b) { return a - b; }

Step 3: Build
──────────────────────────────────────
Build → Build Solution
Output: MyMathLib.lib

Step 4: Use in Another Project
──────────────────────────────────────
1. Add library path in Project Properties
   - Linker → General → Additional Library Directories
2. Add library name
   - Linker → Input → Additional Dependencies → MyMathLib.lib
3. Include header in your code
   #include "MyMath.h"
```

---

## Dynamic Libraries (.dll)

### Characteristics

| Feature | Description |
|---------|-------------|
| **Linking** | At runtime |
| **Included in EXE** | No (separate file) |
| **File Size** | Smaller executable |
| **Update** | Replace DLL only |
| **Runtime Dependencies** | DLL must be present |

### Visual Representation

```
┌─────────────────────────────────────────────────────────────────┐
│                   DYNAMIC LINKING                               │
└─────────────────────────────────────────────────────────────────┘

         Compile Time:
         
    ┌──────────────┐     ┌──────────────┐
    │  Your Code   │     │  Import Lib  │
    │  (main.cpp)  │     │  (.lib stub) │
    └──────────────┘     └──────────────┘
           │                    │
           └────────┬───────────┘
                    │
                    ▼
            ┌──────────────┐
            │    LINKER    │
            └──────────────┘
                    │
                    ▼
            ┌──────────────┐
            │  Program.exe │  ← Contains only references to DLL
            │  (Small)     │
            └──────────────┘
         
         Runtime:
         
    ┌──────────────┐     ┌──────────────┐
    │  Program.exe │────▶│  Library.dll │  ← DLL loaded at runtime
    │              │     │              │
    └──────────────┘     └──────────────┘
```

### Creating a DLL in Visual Studio

```
Step 1: Create DLL Project
──────────────────────────────────────
1. File → New → Project
2. Select "Dynamic-Link Library (DLL)"
3. Name: MyDll
4. Create

Step 2: Add Code with Export
──────────────────────────────────────
// MyDll.h
#pragma once

#ifdef MYDLL_EXPORTS
    #define MYDLL_API __declspec(dllexport)
#else
    #define MYDLL_API __declspec(dllimport)
#endif

MYDLL_API void disp();
MYDLL_API int add(int a, int b);

// MyDll.cpp
#include "MyDll.h"
#include <iostream>
using namespace std;

MYDLL_API void disp() {
    cout << "inside disp of dll file" << endl;
}

MYDLL_API int add(int a, int b) {
    return a + b;
}

Step 3: Build
──────────────────────────────────────
Output: MyDll.dll + MyDll.lib (import library)

Step 4: Use in Another Project
──────────────────────────────────────
1. Copy .dll to output directory of client project
2. Add import library (.lib) to linker
3. Include header file
```

---

## Comparison

### Static vs Dynamic Libraries

| Aspect | Static (.lib) | Dynamic (.dll) |
|--------|--------------|----------------|
| **Size** | Larger EXE | Smaller EXE |
| **Memory** | Duplicated if multiple programs use it | Shared among programs |
| **Updates** | Recompile required | Just replace DLL |
| **Portability** | Self-contained | Need DLL with EXE |
| **Load Time** | Faster startup | Slightly slower startup |
| **Versioning** | Embedded in EXE | Can cause DLL Hell |

### When to Use Each

| Use Case | Recommended |
|----------|-------------|
| Small, self-contained programs | Static |
| Programs that need updates without recompile | Dynamic |
| Sharing code between multiple programs | Dynamic |
| Plugins and extensions | Dynamic |
| Minimal file count | Static |

---

## dllexport and dllimport

### Purpose

| Keyword | Used Where | Purpose |
|---------|------------|---------|
| `__declspec(dllexport)` | In DLL project | Export function/class from DLL |
| `__declspec(dllimport)` | In client project | Import function/class from DLL |

### Example: DLL Developer Side (Export)

```cpp
// In DLL project - exposing functions to other programs

__declspec(dllexport) void disp2() {
    cout << "inside disp2 of dll file" << endl;
}
```

### Example: Client Side (Import)

```cpp
// In header file provided to clients

__declspec(dllimport) void disp2();

// In client code
int main() {
    disp2();  // Call function from DLL
    return 0;
}
```

### Common Pattern with Macros

```cpp
// Header file (works for both DLL and client)

#ifdef MYDLL_EXPORTS
    // Building the DLL - export
    #define MYDLL_API __declspec(dllexport)
#else
    // Using the DLL - import
    #define MYDLL_API __declspec(dllimport)
#endif

MYDLL_API void myFunction();
MYDLL_API class MyClass { ... };
```

---

## pch.h (Precompiled Header)

### What is pch.h?

**pch.h** (Precompiled Header) is a feature to speed up compilation by pre-compiling commonly used header files.

```cpp
// pch.h
#pragma once

// Add headers that rarely change here
#include <iostream>
#include <string>
#include <vector>
```

### Benefits

- Faster compilation time
- Headers compiled only once
- Include in pch.cpp

---

## Practice Questions

### Question 1: When would you choose static over dynamic library?

**Answer:**
- When you want a self-contained executable
- When the library is small and won't be updated frequently
- When you don't want to distribute multiple files
- When startup time is critical

### Question 2: What is DLL Hell?

**Answer:** A situation where multiple applications require different versions of the same DLL, causing compatibility conflicts.

---

## Interview Questions

### Q1: What is the difference between static and dynamic linking?

| Static Linking | Dynamic Linking |
|----------------|-----------------|
| Library code copied into EXE | Library loaded at runtime |
| Larger executable | Smaller executable |
| No runtime dependencies | DLL must be present |
| Compile-time | Runtime |

### Q2: What is an import library?

**Answer:** A small `.lib` file that contains information about the DLL's exported functions. The linker uses it to resolve references, but the actual code is in the DLL.

### Q3: How can you tell if a function should use dllexport or dllimport?

**Answer:** 
- **dllexport**: When building the DLL (creating the library)
- **dllimport**: When using the DLL (consuming the library)

This is typically handled with preprocessor macros that check if `BUILDING_DLL` is defined.

---

## Summary

| Concept | Key Points |
|---------|------------|
| **Static Library** | Copied into EXE at compile time, self-contained |
| **Dynamic Library** | Loaded at runtime, shared, updateable |
| **dllexport** | Mark functions/classes to be available from DLL |
| **dllimport** | Mark functions/classes to be used from DLL |
| **Import Library** | .lib stub for linking with DLL |

---

> **Next Topic:** [07 - Command Line Arguments](./07_Command_Line_Arguments.md)
