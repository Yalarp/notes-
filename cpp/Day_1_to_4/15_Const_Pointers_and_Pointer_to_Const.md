# 15 - Const Pointers and Pointer to Const

## Table of Contents
1. [Introduction](#introduction)
2. [Pointer to Constant](#pointer-to-constant)
3. [Constant Pointer](#constant-pointer)
4. [Constant Pointer to Constant](#constant-pointer-to-constant)
5. [Summary Table](#summary-table)

---

## Introduction

There are three ways to use `const` with pointers:

| Type | Syntax | What's Constant |
|------|--------|-----------------|
| Pointer to constant | `const int* ptr` | The value pointed to |
| Constant pointer | `int* const ptr` | The pointer itself |
| Const pointer to const | `const int* const ptr` | Both |

---

## Pointer to Constant

### Meaning

- Pointer CAN be changed
- Value at pointer CANNOT be changed through pointer

### Syntax

```cpp
const datatype* ptr;
// OR
datatype const* ptr;
```

### Example

```cpp
int num = 100;
const int* ptr = &num;   // Pointer to constant

cout << *ptr << endl;    // ✅ OK: 100

// *ptr = 200;           // ❌ Error! Cannot modify through ptr
num = 200;               // ✅ OK: Can modify directly
ptr++;                   // ✅ OK: Can change pointer
```

---

## Constant Pointer

### Meaning

- Pointer CANNOT be changed
- Value at pointer CAN be changed

### Syntax

```cpp
datatype* const ptr = &var;  // Must initialize!
```

### Example

```cpp
int num = 100;
int* const ptr = &num;   // Constant pointer

cout << *ptr << endl;    // ✅ OK: 100

*ptr = 200;              // ✅ OK: Can modify value
cout << *ptr << endl;    // 200

// ptr++;                // ❌ Error! Cannot change pointer
```

---

## Constant Pointer to Constant

### Meaning

- Pointer CANNOT be changed
- Value CANNOT be changed through pointer
- Read-only pointer

### Syntax

```cpp
const datatype* const ptr = &var;
```

### Example

```cpp
int num = 100;
const int* const ptr = &num;

cout << *ptr << endl;    // ✅ OK: 100

// *ptr = 200;           // ❌ Error!
// ptr++;                // ❌ Error!
num = 200;               // ✅ OK: Direct modification
```

---

## Summary Table

| Type | Change Pointer | Change Value | Syntax |
|------|----------------|--------------|--------|
| Pointer to const | ✅ Yes | ❌ No | `const int* ptr` |
| Const pointer | ❌ No | ✅ Yes | `int* const ptr` |
| Const pointer to const | ❌ No | ❌ No | `const int* const ptr` |

### Memory Trick

```cpp
// Read RIGHT to LEFT:
const int* ptr;        // ptr → *pointer to → const int
int* const ptr;        // ptr → const pointer → *to → int
const int* const ptr;  // ptr → const pointer → *to → const int
```

---

> **Next Topic:** [16 - Function Overloading](./16_Function_Overloading.md)
