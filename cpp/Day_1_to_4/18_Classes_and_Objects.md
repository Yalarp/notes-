# 18 - Classes and Objects

## Table of Contents
1. [Introduction](#introduction)
2. [Class Definition](#class-definition)
3. [Access Specifiers](#access-specifiers)
4. [Creating Objects](#creating-objects)
5. [Member Functions](#member-functions)
6. [Getters and Setters](#getters-and-setters)
7. [The this Pointer](#the-this-pointer)
8. [Summary](#summary)

---

## Introduction

### What is a Class?

A **class** is a blueprint for creating objects. It defines:
- **Data members** (attributes/properties)
- **Member functions** (methods/behaviors)

### What is an Object?

An **object** is an instance of a class - a concrete entity created from the blueprint.

```
Class (Blueprint)          Object (Instance)
─────────────────          ──────────────────
Car                    →   myCar (Honda Civic)
                       →   yourCar (Toyota Camry)
```

---

## Class Definition

### Syntax

```cpp
class ClassName {
private:
    // Data members (hidden)
    type member1;
    type member2;
    
public:
    // Member functions (accessible)
    returnType methodName(parameters);
};  // Semicolon required!
```

### Example

```cpp
#include <iostream>
using namespace std;

class Student {
private:
    char name[20];
    int age;
    
public:
    void setAge(int a) {
        if(a > 0) age = a;
    }
    
    int getAge() {
        return age;
    }
};
```

---

## Access Specifiers

### Three Access Levels

| Specifier | Access |
|-----------|--------|
| `private` | Only within the class |
| `public` | Anywhere |
| `protected` | Class + derived classes |

### Default Access

| Type | Default |
|------|---------|
| `class` | private |
| `struct` | public |

---

## Creating Objects

### Stack Allocation

```cpp
int main() {
    Student s1;           // Create object on stack
    s1.setAge(20);        // Call member function
    cout << s1.getAge();  // 20
}
```

### Heap Allocation

```cpp
int main() {
    Student* s1 = new Student();  // Create on heap
    s1->setAge(20);               // Use arrow operator
    cout << s1->getAge();         // 20
    delete s1;                     // Free memory
}
```

---

## Member Functions

### Inside Class Definition

```cpp
class Student {
public:
    void display() {
        cout << "Display" << endl;
    }
};
```

### Outside Class Definition

```cpp
class Student {
public:
    void display();  // Declaration only
};

// Definition outside using ::
void Student::display() {
    cout << "Display" << endl;
}
```

---

## Getters and Setters

### Purpose

| Type | Purpose |
|------|---------|
| **Getter** | Read private data |
| **Setter** | Write private data (with validation) |

### Example

```cpp
class Student {
private:
    int age;
    
public:
    // Setter with validation
    void setAge(int a) {
        if(a > 0 && a < 150) {
            age = a;
        }
    }
    
    // Getter
    int getAge() {
        return age;
    }
};
```

---

## The this Pointer

### What is `this`?

`this` is a pointer that points to the **current object** (the invoking object).

### Why is it needed?

When multiple objects exist, only ONE copy of member functions exists. `this` tells the function which object to operate on.

```cpp
class Student {
private:
    int age;
    
public:
    void setAge(int age) {
        this->age = age;  // this->age is member, age is parameter
    }
};
```

### Key Points

| Point | Description |
|-------|-------------|
| `this` is a pointer | Access members with `this->member` |
| Not in static members | Static functions don't have `this` |
| Implicitly passed | Compiler adds it automatically |

---

## Summary

| Concept | Key Point |
|---------|-----------|
| **Class** | Blueprint with data + methods |
| **Object** | Instance of class |
| **Access specifiers** | Control visibility |
| **this pointer** | Points to current object |

---

> **Next Topic:** [19 - Constructors](./19_Constructors.md)
