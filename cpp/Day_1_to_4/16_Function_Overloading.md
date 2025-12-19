# 16 - Function Overloading

## Table of Contents
1. [Introduction](#introduction)
2. [Rules for Overloading](#rules-for-overloading)
3. [Name Mangling](#name-mangling)
4. [Examples](#examples)
5. [Ambiguity Issues](#ambiguity-issues)
6. [Summary](#summary)

---

## Introduction

### What is Function Overloading?

**Function Overloading** allows multiple functions with the **same name** but **different parameters**.

```cpp
void open(char* accno);
void open(char* accno, double balance);
// Both named "open" but different parameters
```

---

## Rules for Overloading

Functions can be overloaded if they differ in:

| Difference | Valid? |
|------------|--------|
| Number of parameters | ✅ Yes |
| Type of parameters | ✅ Yes |
| Order of parameters | ✅ Yes |
| Return type only | ❌ No |

---

## Name Mangling

### What is Name Mangling?

C++ compiler **decorates** function names to include parameter information.

```cpp
void disp(char* ptr)   →   _Z4dispPc      (mangled)
void disp(double d)    →   _Z4dispd       (mangled)
void disp(int a, int b)→   _Z4dispii      (mangled)
```

### Why?

- Allows linker to distinguish overloaded functions
- Provides type-safe linkage
- Catch wrong function declarations at link time

---

## Examples

### Complete Example

```cpp
#include <iostream>
using namespace std;

void open(char* accno) {
    cout << "Account Opened\t" << accno << endl;
}

void open(char* accno, double balance) {
    cout << "Account Opened\t" << accno << "\t" << balance << endl;
}

int main() {
    open("A100");           // Calls first version
    open("A101", 10000);    // Calls second version
    return 0;
}
```

**Output:**
```
Account Opened    A100
Account Opened    A101    10000
```

---

## Ambiguity Issues

### Default Arguments Problem

```cpp
void print(int a, int b = 10);
void print(int a);

print(5);  // ❌ Ambiguous! Which one?
```

### Type Promotion

```cpp
void func(int x);
void func(double x);

func(5.0f);  // float → which promotion?
```

---

## Summary

| Concept | Key Point |
|---------|-----------|
| **Overloading** | Same name, different parameters |
| **Name mangling** | Compiler encodes parameter types in name |
| **Cannot overload** | By return type only |
| **Ambiguity** | Watch out for default arguments |

---

> **Next Topic:** [17 - Introduction to OOP](./17_Introduction_to_OOP.md)
