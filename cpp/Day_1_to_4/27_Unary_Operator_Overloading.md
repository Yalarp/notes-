# 27 - Unary Operator Overloading

## Table of Contents
1. [Introduction](#introduction)
2. [Prefix Increment/Decrement](#prefix-incrementdecrement)
3. [Postfix Increment/Decrement](#postfix-incrementdecrement)
4. [Unary Minus](#unary-minus)
5. [Complete Example](#complete-example)
6. [Summary](#summary)

---

## Introduction

### Unary Operators

Operators that work on **one operand**.

| Operator | Example |
|----------|---------|
| `++` | `++obj` or `obj++` |
| `--` | `--obj` or `obj--` |
| `-` | `-obj` |
| `!` | `!obj` |

---

## Prefix Increment/Decrement

### Syntax

```cpp
ClassName& operator++() {      // Prefix ++
    // Increment
    return *this;
}
```

### Example

```cpp
class Counter {
    int value;
public:
    Counter(int v = 0) : value(v) {}
    
    // Prefix ++ (increment then return)
    Counter& operator++() {
        value++;
        return *this;
    }
    
    int getValue() { return value; }
};

int main() {
    Counter c(5);
    cout << (++c).getValue();  // 6 (incremented first)
}
```

---

## Postfix Increment/Decrement

### Syntax (Note the dummy int parameter)

```cpp
ClassName operator++(int) {    // Postfix ++ (int is dummy)
    ClassName temp = *this;
    // Increment
    return temp;               // Return OLD value
}
```

### Example

```cpp
class Counter {
    int value;
public:
    Counter(int v = 0) : value(v) {}
    
    // Postfix ++ (return then increment)
    Counter operator++(int) {
        Counter temp = *this;   // Save current
        value++;
        return temp;            // Return old value
    }
    
    int getValue() { return value; }
};

int main() {
    Counter c(5);
    cout << (c++).getValue();  // 5 (old value returned)
    cout << c.getValue();      // 6 (now incremented)
}
```

---

## Unary Minus

```cpp
class Number {
    int val;
public:
    Number(int v) : val(v) {}
    
    Number operator-() {
        return Number(-val);
    }
    
    int getVal() { return val; }
};

int main() {
    Number n(10);
    Number neg = -n;
    cout << neg.getVal();  // -10
}
```

---

## Complete Example

```cpp
class Counter {
    int value;
public:
    Counter(int v = 0) : value(v) {}
    
    // Prefix ++
    Counter& operator++() {
        ++value;
        return *this;
    }
    
    // Postfix ++
    Counter operator++(int) {
        Counter temp = *this;
        value++;
        return temp;
    }
    
    // Prefix --
    Counter& operator--() {
        --value;
        return *this;
    }
    
    // Postfix --
    Counter operator--(int) {
        Counter temp = *this;
        value--;
        return temp;
    }
    
    int getValue() { return value; }
};
```

---

## Summary

| Type | Signature | Returns |
|------|-----------|---------|
| Prefix `++` | `operator++()` | Reference (new value) |
| Postfix `++` | `operator++(int)` | Copy (old value) |

---

> **Next Topic:** [28 - Binary Operator Overloading](./28_Binary_Operator_Overloading.md)
