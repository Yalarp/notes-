# 19 - Constructors

## Table of Contents
1. [Introduction](#introduction)
2. [Types of Constructors](#types-of-constructors)
3. [Default Constructor](#default-constructor)
4. [Parameterized Constructor](#parameterized-constructor)
5. [Constructor Overloading](#constructor-overloading)
6. [Initializer List](#initializer-list)
7. [Constructor Rules](#constructor-rules)
8. [Summary](#summary)

---

## Introduction

### What is a Constructor?

A **constructor** is a special member function that:
- Has the **same name** as the class
- Is called **automatically** when an object is created
- Used to **initialize** object data members
- Has **no return type** (not even void)

---

## Types of Constructors

| Type | Description |
|------|-------------|
| **Default** | No parameters |
| **Parameterized** | Takes parameters |
| **Copy** | Initializes from another object |

---

## Default Constructor

### Definition

A constructor with **no parameters** or all parameters have **default values**.

```cpp
class Student {
private:
    int age;
    char name[20];
    
public:
    Student() {                    // Default constructor
        age = 0;
        strcpy(name, "Unknown");
        cout << "Constructor called" << endl;
    }
};

int main() {
    Student s1;    // Constructor called automatically
    Student s2;    // Constructor called again
}
```

### Compiler-Generated Default Constructor

If you don't write ANY constructor, compiler provides a default one that:
- Allocates memory for object
- Does NOT initialize primitive members (garbage values)

---

## Parameterized Constructor

### Definition

A constructor that takes **parameters** to initialize object members.

```cpp
class Student {
private:
    int age;
    string name;
    
public:
    Student(string n, int a) {     // Parameterized constructor
        name = n;
        age = a;
    }
    
    void display() {
        cout << name << " is " << age << " years old" << endl;
    }
};

int main() {
    Student s1("John", 20);       // Direct initialization
    Student s2 = Student("Jane", 22);  // Explicit call
    
    s1.display();  // John is 20 years old
    s2.display();  // Jane is 22 years old
}
```

---

## Constructor Overloading

Multiple constructors with different parameters.

```cpp
class Student {
private:
    string name;
    int age;
    
public:
    Student() {                    // Default
        name = "Unknown";
        age = 0;
    }
    
    Student(string n) {            // One parameter
        name = n;
        age = 0;
    }
    
    Student(string n, int a) {     // Two parameters
        name = n;
        age = a;
    }
};

int main() {
    Student s1;                    // Calls default
    Student s2("John");            // Calls one-param
    Student s3("Jane", 22);        // Calls two-param
}
```

---

## Initializer List

### Preferred Way to Initialize Members

```cpp
class Student {
private:
    string name;
    int age;
    
public:
    // Using initializer list (preferred)
    Student(string n, int a) : name(n), age(a) {
        // Body can be empty or contain additional logic
    }
};
```

### When is Initializer List Required?

| Case | Reason |
|------|--------|
| `const` members | Must be initialized at creation |
| Reference members | Must be initialized at creation |
| Member objects without default constructor | Must be initialized explicitly |

---

## Constructor Rules

| Rule | Description |
|------|-------------|
| Same name as class | Mandatory |
| No return type | Not even void |
| Called automatically | When object is created |
| Can be overloaded | Multiple constructors allowed |
| Cannot be virtual | Not allowed |
| Can be private | Prevents direct object creation |

---

## Summary

| Constructor Type | When Called |
|------------------|-------------|
| **Default** | `Student s1;` |
| **Parameterized** | `Student s1("John", 20);` |
| **Copy** | `Student s2 = s1;` |

---

> **Next Topic:** [20 - Destructors](./20_Destructors.md)
