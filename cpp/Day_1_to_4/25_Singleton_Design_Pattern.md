# 25 - Singleton Design Pattern

## Table of Contents
1. [Introduction](#introduction)
2. [Implementation](#implementation)
3. [Thread-Safe Singleton](#thread-safe-singleton)
4. [Use Cases](#use-cases)
5. [Summary](#summary)

---

## Introduction

### What is Singleton?

**Singleton** ensures a class has **only one instance** and provides global access to it.

### Key Elements

| Element | Purpose |
|---------|---------|
| Private constructor | Prevent direct instantiation |
| Private static instance | Store the single instance |
| Public static method | Provide access to instance |

---

## Implementation

### Basic Singleton

```cpp
class Singleton {
private:
    static Singleton* instance;   // Pointer to single instance
    
    // Private constructor
    Singleton() {
        cout << "Singleton created" << endl;
    }
    
public:
    // Delete copy and assignment
    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;
    
    // Public access method
    static Singleton* getInstance() {
        if(instance == nullptr) {
            instance = new Singleton();
        }
        return instance;
    }
    
    void doSomething() {
        cout << "Doing something" << endl;
    }
};

// Initialize static member
Singleton* Singleton::instance = nullptr;

int main() {
    // Singleton* s1 = new Singleton();  // ❌ Error! Constructor private
    
    Singleton* s1 = Singleton::getInstance();  // ✅ First call creates
    Singleton* s2 = Singleton::getInstance();  // ✅ Same instance
    
    cout << (s1 == s2) << endl;  // 1 (true - same instance!)
}
```

---

## Thread-Safe Singleton

### Meyer's Singleton (C++11)

```cpp
class Singleton {
private:
    Singleton() {}
    
public:
    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;
    
    static Singleton& getInstance() {
        static Singleton instance;   // Thread-safe in C++11
        return instance;
    }
};
```

---

## Use Cases

| Use Case | Example |
|----------|---------|
| Configuration manager | Single config instance |
| Logger | One logging instance |
| Database connection pool | Shared pool |
| Hardware access | Single driver instance |

---

## Summary

| Concept | Key Point |
|---------|-----------|
| **Purpose** | Ensure single instance |
| **Private constructor** | Prevent new instances |
| **Static getInstance** | Access point |
| **C++11 local static** | Thread-safe, simple |

---

> **Next Topic:** [26 - Operator Overloading Introduction](./26_Operator_Overloading_Introduction.md)
