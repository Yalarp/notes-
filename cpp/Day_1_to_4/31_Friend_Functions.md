# 31 - Friend Functions

## Table of Contents
1. [Introduction](#introduction)
2. [Friend Function Syntax](#friend-function-syntax)
3. [When to Use Friend](#when-to-use-friend)
4. [Friend with Operator Overloading](#friend-with-operator-overloading)
5. [Key Points](#key-points)
6. [Summary](#summary)

---

## Introduction

### What is a Friend Function?

A **friend function** is a non-member function that can access the **private and protected members** of a class.

### Why Use Friend?

| Scenario | Reason |
|----------|--------|
| Operator overloading | `<<` and `>>` operators |
| Bridge between classes | Access multiple classes' private data |
| When member function not suitable | Left operand is not the class |

---

## Friend Function Syntax

### Declaration

```cpp
class MyClass {
private:
    int data;
    
public:
    MyClass(int d) : data(d) {}
    
    // Declare friend function
    friend void display(const MyClass& obj);
};

// Define friend function (NOT a member, no MyClass::)
void display(const MyClass& obj) {
    cout << obj.data << endl;   // Can access private data!
}

int main() {
    MyClass m(10);
    display(m);   // 10
}
```

---

## When to Use Friend

### Stream Operators (`<<` and `>>`)

The left operand is `ostream`/`istream`, not our class.

```cpp
class Point {
    int x, y;
public:
    Point(int x, int y) : x(x), y(y) {}
    
    // Must be friend because left operand is ostream
    friend ostream& operator<<(ostream& os, const Point& p);
};

ostream& operator<<(ostream& os, const Point& p) {
    os << "(" << p.x << ", " << p.y << ")";
    return os;
}

int main() {
    Point p(3, 4);
    cout << p;   // (3, 4)
}
```

---

## Friend with Operator Overloading

### Binary Operator (Left operand is not class)

```cpp
class Number {
    int val;
public:
    Number(int v) : val(v) {}
    
    // 5 + obj (left operand is int, not Number)
    friend Number operator+(int n, const Number& obj);
};

Number operator+(int n, const Number& obj) {
    return Number(n + obj.val);
}

int main() {
    Number n(10);
    Number result = 5 + n;  // Works because of friend
}
```

---

## Key Points

| Point | Description |
|-------|-------------|
| Not a member | Defined outside class (no `ClassName::`) |
| Access private | Can access private/protected members |
| Declared in class | `friend` keyword inside class |
| Not inherited | Friendship is not inherited |
| Not symmetric | If A is friend of B, B is NOT friend of A |
| Not transitive | Friend of friend is NOT a friend |

---

## Summary

| Concept | Key Point |
|---------|-----------|
| **friend function** | Non-member with private access |
| **Syntax** | `friend returnType funcName(params);` |
| **Common use** | Stream operators, mixed-type operations |
| **Break encapsulation?** | Minimally, when necessary |

---

> **Next Topic:** [32 - Friend Classes](./32_Friend_Classes.md)
