# 32 - Friend Classes

## Table of Contents
1. [Introduction](#introduction)
2. [Friend Class Syntax](#friend-class-syntax)
3. [Example](#example)
4. [Properties](#properties)
5. [Summary](#summary)

---

## Introduction

### What is a Friend Class?

A **friend class** can access all **private and protected members** of the class that declares it as a friend.

```cpp
class A {
    friend class B;   // B can access A's private members
private:
    int secret;
};
```

---

## Friend Class Syntax

```cpp
class ClassA {
private:
    int data;
    
public:
    ClassA(int d) : data(d) {}
    
    friend class ClassB;   // ClassB is a friend
};

class ClassB {
public:
    void showData(const ClassA& a) {
        cout << a.data << endl;   // Can access private!
    }
};
```

---

## Example

### Linked List with Iterator

```cpp
class LinkedList {
private:
    struct Node {
        int data;
        Node* next;
    };
    Node* head;
    
    friend class Iterator;   // Iterator can access private Node
    
public:
    // LinkedList methods...
};

class Iterator {
    LinkedList::Node* current;   // Can use private Node!
    
public:
    Iterator(LinkedList& list) : current(list.head) {}
    
    int getValue() { return current->data; }
    void next() { current = current->next; }
    bool hasMore() { return current != nullptr; }
};
```

---

## Properties

| Property | Value |
|----------|-------|
| **Friendship not mutual** | A declares B friend ≠ B declares A friend |
| **Not inherited** | Derived classes don't inherit friendship |
| **Not transitive** | Friend of friend is not a friend |
| **Access level** | Can access all private/protected |

### Example: Not Mutual

```cpp
class A {
    friend class B;   // B can access A's private
};

class B {
    // A CANNOT access B's private (not mutual)
};
```

---

## Summary

| Concept | Key Point |
|---------|-----------|
| **friend class** | Full access to private members |
| **Syntax** | `friend class ClassName;` |
| **Not mutual** | One-way relationship |
| **Use case** | Tightly coupled helper classes |

---

## Complete C++ Notes Summary

Congratulations! You have completed all 32 comprehensive C++ study notes:

### Day 1: C++ Fundamentals
- 01-08: Introduction, Pointers, Storage Classes, Preprocessor, DMA, Libraries, Command Line, Patterns

### Day 2: C++ Enhancements
- 09-16: Enhancements, References, Linkage, Structs, new/delete, Inline, Const Pointers, Overloading

### Day 3: Object-Oriented Programming
- 17-25: OOP, Classes, Constructors, Destructors, Copy Constructor, Static, Const Functions, Memory, Singleton

### Day 4: Advanced C++
- 26-32: Operator Overloading, Unary/Binary, Smart Pointers, Type Conversion, Friend Functions/Classes

---

> **Happy Learning! 🎉**
