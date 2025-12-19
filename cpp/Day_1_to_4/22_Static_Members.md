# 22 - Static Members

## Table of Contents
1. [Introduction](#introduction)
2. [Static Data Members](#static-data-members)
3. [Static Member Functions](#static-member-functions)
4. [Key Differences](#key-differences)
5. [Use Cases](#use-cases)
6. [Summary](#summary)

---

## Introduction

**Static members** belong to the **class** rather than to any specific object.

| Type | Belongs To |
|------|------------|
| Instance members | Each object (copy per object) |
| Static members | Class (shared by all objects) |

---

## Static Data Members

### Declaration and Definition

```cpp
class Counter {
private:
    static int count;    // Declaration inside class
    
public:
    Counter() {
        count++;
    }
    
    static int getCount() {
        return count;
    }
};

// Definition OUTSIDE class (required!)
int Counter::count = 0;

int main() {
    Counter c1;
    Counter c2;
    Counter c3;
    
    cout << Counter::getCount() << endl;  // 3
}
```

### Memory Visualization

```
┌─────────────────────────────────────────────────────────────────┐
│                    STATIC vs INSTANCE                           │
└─────────────────────────────────────────────────────────────────┘

Instance Members:                    Static Members:
(One per object)                    (One for class)

   c1          c2          c3            Counter::count
┌──────┐   ┌──────┐   ┌──────┐         ┌──────────────┐
│ data │   │ data │   │ data │         │     3        │
└──────┘   └──────┘   └──────┘         └──────────────┘
                                              ↑
                            Shared by c1, c2, c3
```

---

## Static Member Functions

### Characteristics

| Property | Value |
|----------|-------|
| Access | Via class name (`ClassName::func()`) |
| `this` pointer | Not available |
| Can access | Only static members |
| Cannot access | Instance members directly |

### Example

```cpp
class Math {
public:
    static int add(int a, int b) {
        return a + b;
    }
    
    static int multiply(int a, int b) {
        return a * b;
    }
};

int main() {
    // Call without creating object
    cout << Math::add(5, 3) << endl;       // 8
    cout << Math::multiply(5, 3) << endl;  // 15
}
```

---

## Key Differences

| Instance | Static |
|----------|--------|
| Accessed via object | Accessed via class |
| Has `this` pointer | No `this` pointer |
| Created per object | Created once |
| Memory with object | Memory at program start |

---

## Use Cases

| Use Case | Example |
|----------|---------|
| Object counting | Track how many objects exist |
| Shared data | Configuration settings |
| Utility functions | Math operations |
| Singleton pattern | Single instance control |

---

## Summary

| Concept | Key Point |
|---------|-----------|
| **Static data** | Shared by all objects |
| **Static function** | No `this`, access via class |
| **Definition** | Required outside class |
| **Access** | `ClassName::member` |

---

> **Next Topic:** [23 - Const Member Functions](./23_Const_Member_Functions.md)
