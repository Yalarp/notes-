# 24 - Memory Management (Stack vs Heap)

## Table of Contents
1. [Overview](#overview)
2. [Stack Memory](#stack-memory)
3. [Heap Memory](#heap-memory)
4. [Object Creation](#object-creation)
5. [Memory Leak Prevention](#memory-leak-prevention)
6. [Summary](#summary)

---

## Overview

### Memory Layout

```
┌─────────────────────────────────────────────────────────────────┐
│                    PROGRAM MEMORY                               │
└─────────────────────────────────────────────────────────────────┘

    HIGH ADDRESS
    ┌───────────────────────────────┐
    │           STACK               │ ← Local variables, function calls
    │   (Grows Downward ↓)          │   Automatic cleanup
    │                               │
    ├───────────────────────────────┤
    │                               │
    │           HEAP                │ ← Dynamic allocation (new/delete)
    │   (Grows Upward ↑)            │   Manual cleanup required
    │                               │
    ├───────────────────────────────┤
    │      DATA / BSS               │ ← Global, static variables
    ├───────────────────────────────┤
    │           TEXT                │ ← Program code
    └───────────────────────────────┘
    LOW ADDRESS
```

---

## Stack Memory

### Characteristics

| Feature | Stack |
|---------|-------|
| Allocation | Automatic |
| Deallocation | Automatic (scope exit) |
| Size | Limited (~1-8 MB) |
| Speed | Fast |
| Fragmentation | No |

### Example

```cpp
void function() {
    int x = 10;         // Stack allocation
    Student s1("John"); // Stack object
}   // x and s1 automatically destroyed here
```

---

## Heap Memory

### Characteristics

| Feature | Heap |
|---------|------|
| Allocation | Manual (new) |
| Deallocation | Manual (delete) |
| Size | Large (GBs) |
| Speed | Slower |
| Fragmentation | Possible |

### Example

```cpp
void function() {
    int* x = new int(10);      // Heap allocation
    Student* s1 = new Student("John");  // Heap object
    
    // Must manually delete!
    delete x;
    delete s1;
}
```

---

## Object Creation

### Stack Object

```cpp
Student s1("John");    // Constructor called
// Use s1.method()
// Destructor auto-called when scope ends
```

### Heap Object

```cpp
Student* s1 = new Student("John");  // Constructor called
// Use s1->method() (arrow operator)
delete s1;   // Destructor called, memory freed
```

---

## Memory Leak Prevention

### Memory Leak

```cpp
void bad() {
    int* ptr = new int(5);
    // Forgot delete!
}   // ptr destroyed, but memory still allocated = LEAK
```

### Prevention

```cpp
void good() {
    int* ptr = new int(5);
    // Use ptr...
    delete ptr;       // Free memory
    ptr = nullptr;    // Prevent dangling pointer
}
```

---

## Summary

| Memory | Allocation | Cleanup | Use For |
|--------|------------|---------|---------|
| **Stack** | Automatic | Automatic | Local variables, small objects |
| **Heap** | new | delete | Large objects, dynamic lifetime |

---

> **Next Topic:** [25 - Singleton Design Pattern](./25_Singleton_Design_Pattern.md)
