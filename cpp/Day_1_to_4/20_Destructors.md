# 20 - Destructors

## Table of Contents
1. [Introduction](#introduction)
2. [Destructor Syntax](#destructor-syntax)
3. [When is Destructor Called](#when-is-destructor-called)
4. [Example with Dynamic Memory](#example-with-dynamic-memory)
5. [Rules](#rules)
6. [Summary](#summary)

---

## Introduction

### What is a Destructor?

A **destructor** is a special member function that:
- Has the **same name** as class, preceded by `~`
- Is called **automatically** when object is destroyed
- Used to **release resources** (memory, files, connections)
- Has **no return type** and **no parameters**

---

## Destructor Syntax

```cpp
class ClassName {
public:
    ~ClassName() {      // Destructor
        // Cleanup code
    }
};
```

### Example

```cpp
class Student {
private:
    string name;
    
public:
    Student(string n) : name(n) {
        cout << "Constructor: " << name << endl;
    }
    
    ~Student() {
        cout << "Destructor: " << name << endl;
    }
};

int main() {
    Student s1("John");
    Student s2("Jane");
}   // Destructors called in REVERSE order
```

**Output:**
```
Constructor: John
Constructor: Jane
Destructor: Jane    ← Reverse order!
Destructor: John
```

---

## When is Destructor Called

| Scenario | When |
|----------|------|
| **Stack object** | Goes out of scope |
| **Heap object** | `delete` is called |
| **Array of objects** | Each element destroyed |
| **Program ends** | Static/global objects |

---

## Example with Dynamic Memory

### Resource Management

```cpp
class DynamicArray {
private:
    int* arr;
    int size;
    
public:
    DynamicArray(int n) : size(n) {
        arr = new int[n];              // Allocate in constructor
        cout << "Memory allocated" << endl;
    }
    
    ~DynamicArray() {
        delete[] arr;                   // Release in destructor
        cout << "Memory released" << endl;
    }
};

int main() {
    DynamicArray da(10);
}   // Destructor called, memory freed automatically
```

---

## Rules

| Rule | Description |
|------|-------------|
| One per class | Cannot be overloaded |
| No parameters | Cannot take arguments |
| No return type | Not even void |
| Called automatically | When object destroyed |
| Cannot be called explicitly | Well, you can, but shouldn't |

---

## Summary

| Concept | Key Point |
|---------|-----------|
| **Destructor** | `~ClassName()` |
| **Purpose** | Release resources |
| **Call order** | Reverse of construction |
| **Best practice** | Free dynamically allocated memory |

---

> **Next Topic:** [21 - Copy Constructor](./21_Copy_Constructor.md)
