# 21 - Copy Constructor

## Table of Contents
1. [Introduction](#introduction)
2. [When is Copy Constructor Called](#when-is-copy-constructor-called)
3. [Default Copy Constructor](#default-copy-constructor)
4. [Shallow vs Deep Copy](#shallow-vs-deep-copy)
5. [Custom Copy Constructor](#custom-copy-constructor)
6. [Summary](#summary)

---

## Introduction

### What is a Copy Constructor?

A **copy constructor** is called when an object is initialized using another object of the same class.

```cpp
MyClass m1;           // Default constructor
MyClass m2 = m1;      // Copy constructor!
// OR
MyClass m2(m1);       // Copy constructor!
```

### Syntax

```cpp
ClassName(const ClassName& source) {
    // Copy data from source
}
```

---

## When is Copy Constructor Called

| Scenario | Example |
|----------|---------|
| Initialization | `MyClass m2 = m1;` |
| Initialization | `MyClass m2(m1);` |
| Pass by value | `void func(MyClass obj)` |
| Return by value | `MyClass func() { return obj; }` |

---

## Default Copy Constructor

If you don't define a copy constructor, compiler provides one that does **member-wise copy** (shallow copy).

```cpp
class Simple {
    int x;
    int y;
};

Simple s1;
s1.x = 10; s1.y = 20;
Simple s2 = s1;    // Default copy constructor copies x and y
```

---

## Shallow vs Deep Copy

### Shallow Copy (Default)

Copies pointer value, not the data it points to.

```cpp
class Shallow {
    int* data;
public:
    Shallow(int val) {
        data = new int(val);
    }
    ~Shallow() {
        delete data;      // Problem: both objects delete same memory!
    }
};

Shallow s1(10);
Shallow s2 = s1;    // s2.data = s1.data (same pointer!)
// When s1 and s2 are destroyed → CRASH (double delete)
```

### Visual: Shallow Copy Problem

```
After shallow copy:
─────────────────────────────────────────────
s1.data ──────┐
              ├──────▶ [10] (heap memory)
s2.data ──────┘

When s2 destroyed: delete s2.data → memory freed
When s1 destroyed: delete s1.data → CRASH! (already freed)
```

### Deep Copy (Safe)

Creates new memory and copies the actual data.

```cpp
class Deep {
    int* data;
public:
    Deep(int val) {
        data = new int(val);
    }
    
    // Custom copy constructor (deep copy)
    Deep(const Deep& source) {
        data = new int(*source.data);    // NEW allocation!
    }
    
    ~Deep() {
        delete data;    // Safe: each object has its own memory
    }
};
```

### Visual: Deep Copy

```
After deep copy:
─────────────────────────────────────────────
s1.data ──────────▶ [10] (heap memory 1)

s2.data ──────────▶ [10] (heap memory 2) ← Separate copy!
```

---

## Custom Copy Constructor

### Complete Example

```cpp
class Student {
private:
    char* name;
    int age;
    
public:
    // Constructor
    Student(const char* n, int a) {
        name = new char[strlen(n) + 1];
        strcpy(name, n);
        age = a;
    }
    
    // Copy Constructor (Deep Copy)
    Student(const Student& source) {
        name = new char[strlen(source.name) + 1];
        strcpy(name, source.name);
        age = source.age;
        cout << "Copy constructor called" << endl;
    }
    
    // Destructor
    ~Student() {
        delete[] name;
    }
};
```

---

## Summary

| Concept | Key Point |
|---------|-----------|
| **Copy Constructor** | Initializes object from another |
| **Shallow Copy** | Copies pointers (dangerous) |
| **Deep Copy** | Copies actual data (safe) |
| **Rule** | If class has pointers, define copy constructor |

---

> **Next Topic:** [22 - Static Members](./22_Static_Members.md)
