# 26 - Operator Overloading Introduction

## Table of Contents
1. [Introduction](#introduction)
2. [Syntax](#syntax)
3. [Rules](#rules)
4. [Operators That Can Be Overloaded](#operators-that-can-be-overloaded)
5. [Operators That Cannot Be Overloaded](#operators-that-cannot-be-overloaded)
6. [Summary](#summary)

---

## Introduction

### What is Operator Overloading?

**Operator overloading** allows you to redefine the behavior of operators for user-defined types (classes).

```cpp
Complex c1(3, 4);
Complex c2(1, 2);
Complex c3 = c1 + c2;   // + operator overloaded for Complex class
```

### Why Overload Operators?

| Benefit | Example |
|---------|---------|
| Natural syntax | `c1 + c2` instead of `c1.add(c2)` |
| Intuitive code | `str1 + str2` for string concatenation |
| Consistency | Use same operators as built-in types |

---

## Syntax

### Member Function Syntax

```cpp
returnType operator symbol (parameters) {
    // Implementation
}
```

### Example

```cpp
class Complex {
private:
    double real, imag;
    
public:
    Complex(double r = 0, double i = 0) : real(r), imag(i) {}
    
    // Overload + operator
    Complex operator+(const Complex& other) {
        return Complex(real + other.real, imag + other.imag);
    }
};
```

---

## Rules

| Rule | Description |
|------|-------------|
| Cannot create new operators | Only existing operators |
| Cannot change precedence | Maintains original precedence |
| Cannot change associativity | Maintains original associativity |
| At least one operand must be user-defined | Cannot override for primitives |

---

## Operators That Can Be Overloaded

```
Arithmetic:     +  -  *  /  %
Comparison:     ==  !=  <  >  <=  >=
Logical:        &&  ||  !
Bitwise:        &  |  ^  ~  <<  >>
Assignment:     =  +=  -=  *=  /=  %=
Increment:      ++  --
Subscript:      []
Function call:  ()
Member access:  ->
Stream:         <<  >>
```

---

## Operators That Cannot Be Overloaded

| Operator | Name |
|----------|------|
| `::` | Scope resolution |
| `.` | Member access |
| `.*` | Pointer to member |
| `?:` | Ternary conditional |
| `sizeof` | Size of |
| `typeid` | Type identification |

---

## Summary

| Concept | Key Point |
|---------|-----------|
| **Purpose** | Redefine operators for classes |
| **Syntax** | `returnType operator symbol (params)` |
| **Cannot** | Create new operators or change precedence |

---

> **Next Topic:** [27 - Unary Operator Overloading](./27_Unary_Operator_Overloading.md)
