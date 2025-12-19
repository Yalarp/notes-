# 28 - Binary Operator Overloading

## Table of Contents
1. [Introduction](#introduction)
2. [Arithmetic Operators](#arithmetic-operators)
3. [Comparison Operators](#comparison-operators)
4. [Assignment Operator](#assignment-operator)
5. [Stream Operators](#stream-operators)
6. [Summary](#summary)

---

## Introduction

### Binary Operators

Operators that work on **two operands**.

```cpp
obj1 + obj2    // + is binary operator
obj1 == obj2   // == is binary operator
```

---

## Arithmetic Operators

### Example: Complex Number Addition

```cpp
class Complex {
    double real, imag;
public:
    Complex(double r = 0, double i = 0) : real(r), imag(i) {}
    
    // Overload + as member function
    Complex operator+(const Complex& other) const {
        return Complex(real + other.real, imag + other.imag);
    }
    
    // Overload - 
    Complex operator-(const Complex& other) const {
        return Complex(real - other.real, imag - other.imag);
    }
    
    void display() {
        cout << real << " + " << imag << "i" << endl;
    }
};

int main() {
    Complex c1(3, 4);
    Complex c2(1, 2);
    Complex c3 = c1 + c2;  // 4 + 6i
    c3.display();
}
```

---

## Comparison Operators

```cpp
class Point {
    int x, y;
public:
    Point(int x, int y) : x(x), y(y) {}
    
    bool operator==(const Point& other) const {
        return x == other.x && y == other.y;
    }
    
    bool operator!=(const Point& other) const {
        return !(*this == other);
    }
    
    bool operator<(const Point& other) const {
        return (x * x + y * y) < (other.x * other.x + other.y * other.y);
    }
};

int main() {
    Point p1(3, 4);
    Point p2(3, 4);
    Point p3(1, 1);
    
    cout << (p1 == p2);  // 1 (true)
    cout << (p1 < p3);   // 0 (false, p1 farther from origin)
}
```

---

## Assignment Operator

### Copy Assignment

```cpp
class String {
    char* data;
public:
    // Copy assignment operator
    String& operator=(const String& other) {
        if(this != &other) {           // Self-assignment check
            delete[] data;              // Free old memory
            data = new char[strlen(other.data) + 1];
            strcpy(data, other.data);
        }
        return *this;
    }
};
```

### Key Points

| Point | Description |
|-------|-------------|
| Self-assignment check | `if(this != &other)` |
| Free old resources | Before allocating new |
| Return `*this` | For chaining: `a = b = c` |

---

## Stream Operators

### Must Be Non-Member (Friend)

```cpp
class Point {
    int x, y;
public:
    Point(int x, int y) : x(x), y(y) {}
    
    // Friend function for <<
    friend ostream& operator<<(ostream& os, const Point& p);
    friend istream& operator>>(istream& is, Point& p);
};

ostream& operator<<(ostream& os, const Point& p) {
    os << "(" << p.x << ", " << p.y << ")";
    return os;
}

istream& operator>>(istream& is, Point& p) {
    is >> p.x >> p.y;
    return is;
}

int main() {
    Point p(3, 4);
    cout << p << endl;  // (3, 4)
    
    Point p2(0, 0);
    cin >> p2;          // Input: 5 6
    cout << p2;         // (5, 6)
}
```

---

## Summary

| Operator Type | Example | Implementation |
|---------------|---------|----------------|
| Arithmetic | `+`, `-`, `*` | Member or non-member |
| Comparison | `==`, `<` | Returns bool |
| Assignment | `=` | Return `*this` |
| Stream | `<<`, `>>` | Friend function |

---

> **Next Topic:** [29 - Smart Pointers](./29_Smart_Pointers.md)
