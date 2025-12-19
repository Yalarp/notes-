# 30 - Type Conversion

## Table of Contents
1. [Introduction](#introduction)
2. [Implicit Conversion](#implicit-conversion)
3. [Explicit Conversion](#explicit-conversion)
4. [Conversion Operators](#conversion-operators)
5. [Converting Constructors](#converting-constructors)
6. [Summary](#summary)

---

## Introduction

### Types of Conversion

| Type | Description |
|------|-------------|
| Implicit | Automatic by compiler |
| Explicit | Using cast operators |
| User-defined | Custom conversions for classes |

---

## Implicit Conversion

### Built-in Types

```cpp
int i = 10;
double d = i;   // Implicit: int → double
```

### Promotion Rules

```cpp
char → int → long → float → double
```

---

## Explicit Conversion

### C-Style Cast

```cpp
double d = 3.14;
int i = (int)d;   // C-style cast
```

### C++ Cast Operators

| Operator | Purpose |
|----------|---------|
| `static_cast` | Compile-time type conversion |
| `dynamic_cast` | Runtime polymorphic conversion |
| `const_cast` | Remove/add const |
| `reinterpret_cast` | Low-level bit reinterpretation |

```cpp
double d = 3.14;
int i = static_cast<int>(d);   // Preferred in C++
```

---

## Conversion Operators

Allow class to convert to another type.

### Syntax

```cpp
operator targetType() const {
    return value;
}
```

### Example

```cpp
class Distance {
    int meters;
public:
    Distance(int m) : meters(m) {}
    
    // Convert Distance to int
    operator int() const {
        return meters;
    }
    
    // Convert Distance to double (in kilometers)
    operator double() const {
        return meters / 1000.0;
    }
};

int main() {
    Distance d(1500);
    
    int m = d;        // 1500 (uses operator int)
    double km = d;    // 1.5 (uses operator double)
}
```

---

## Converting Constructors

Allow other types to convert to class type.

### Example

```cpp
class Complex {
    double real, imag;
public:
    // Converting constructor (int → Complex)
    Complex(int r) : real(r), imag(0) {}
    
    // Regular constructor
    Complex(double r, double i) : real(r), imag(i) {}
};

int main() {
    Complex c = 5;    // int 5 converted to Complex(5, 0)
}
```

### explicit Keyword

Prevent implicit conversion:

```cpp
class Complex {
public:
    explicit Complex(int r) : real(r), imag(0) {}
};

int main() {
    // Complex c = 5;        // ❌ Error! implicit not allowed
    Complex c(5);            // ✅ OK
    Complex c = Complex(5);  // ✅ OK
}
```

---

## Summary

| Conversion | Direction | Method |
|------------|-----------|--------|
| To basic type | Class → int/double | Conversion operator |
| From basic type | int/double → Class | Converting constructor |
| Prevent implicit | - | `explicit` keyword |

---

> **Next Topic:** [31 - Friend Functions](./31_Friend_Functions.md)
