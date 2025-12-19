# 23 - Const Member Functions

## Table of Contents
1. [Introduction](#introduction)
2. [Syntax](#syntax)
3. [Rules](#rules)
4. [Const Objects](#const-objects)
5. [mutable Keyword](#mutable-keyword)
6. [Summary](#summary)

---

## Introduction

A **const member function** promises NOT to modify any data members of the object.

---

## Syntax

```cpp
class Student {
private:
    string name;
    int age;
    
public:
    // Const member function
    void display() const {
        cout << name << ", " << age << endl;
        // age = 20;  // ❌ Error! Cannot modify
    }
    
    // Non-const member function
    void setAge(int a) {
        age = a;      // ✅ OK
    }
};
```

---

## Rules

| Rule | Description |
|------|-------------|
| Cannot modify members | Any attempt causes compile error |
| Can only call const functions | Cannot call non-const functions |
| Can be called on const objects | Required for const objects |

### Example

```cpp
class Demo {
    int value;
public:
    Demo(int v) : value(v) {}
    
    int getValue() const {     // Can be called on const objects
        return value;
    }
    
    void setValue(int v) {      // Cannot be called on const objects
        value = v;
    }
};

int main() {
    const Demo d(10);
    cout << d.getValue() << endl;  // ✅ OK
    // d.setValue(20);             // ❌ Error!
}
```

---

## Const Objects

A **const object** can only call **const member functions**.

```cpp
const Student s1("John", 20);
s1.display();     // ✅ OK (display is const)
// s1.setAge(25); // ❌ Error! (setAge is not const)
```

---

## mutable Keyword

Allows a member to be modified even in const functions.

```cpp
class Counter {
    mutable int accessCount;  // Can be modified in const functions
    int value;
    
public:
    int getValue() const {
        accessCount++;         // ✅ OK because mutable
        return value;
    }
};
```

---

## Summary

| Concept | Key Point |
|---------|-----------|
| **const function** | Append `const` after parameter list |
| **Cannot modify** | Any data member (except mutable) |
| **const objects** | Can only call const functions |
| **mutable** | Exception to const restriction |

---

> **Next Topic:** [24 - Memory Management (Stack vs Heap)](./24_Memory_Management_Stack_vs_Heap.md)
