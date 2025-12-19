# 12 - Structs in C++

## Table of Contents
1. [Introduction](#introduction)
2. [C++ Struct Enhancements](#c-struct-enhancements)
3. [Struct with Default Values](#struct-with-default-values)
4. [Access Specifiers in Struct](#access-specifiers-in-struct)
5. [Struct vs Class](#struct-vs-class)
6. [Practice Questions](#practice-questions)

---

## Introduction

### What is a Struct?

A **struct** is a user-defined data type that groups related data together.

### C++ Enhancements to Struct

| Feature | C Struct | C++ Struct |
|---------|----------|------------|
| Methods | ❌ No | ✅ Yes |
| Access specifiers | ❌ No | ✅ Yes |
| Default member values | ❌ No | ✅ Yes |
| `struct` keyword in declaration | Required | Optional |

---

## C++ Struct Enhancements

### Struct with Default Values

```cpp
#include <iostream>
using namespace std;

struct Student {
    char name[20] = "noname";   // Default value
    int age = 0;                // Default value
};

int main() {
    Student s1;   // No 'struct' keyword needed in C++
    cout << s1.name << "\t" << s1.age << endl;  // noname    0
    
    s1.age = -5;  // Problem: No validation, data corruption risk!
    cout << s1.name << "\t" << s1.age << endl;  // noname    -5
}
```

### Line-by-Line Explanation

| Line | Code | Explanation |
|------|------|-------------|
| `char name[20] = "noname";` | Default initialization in C++ struct |
| `Student s1;` | No `struct` keyword needed (unlike C) |
| `s1.age = -5;` | Direct access allows invalid data! |

---

## Access Specifiers in Struct

### Making Members Private

```cpp
#include <iostream>
using namespace std;

struct Student {
private:                      // Access specifier
    char name[20] = "noname";
    int age = 0;
public:
    void setAge(int a) {
        if(a > 0) age = a;    // Validation
    }
    int getAge() { return age; }
};

int main() {
    Student s1;
    // s1.age = -5;  // ❌ Error! age is private
    s1.setAge(-5);   // Ignored by validation
    s1.setAge(20);   // Works!
    cout << s1.getAge() << endl;  // 20
}
```

---

## Struct vs Class

### The Only Difference

| Feature | struct | class |
|---------|--------|-------|
| **Default access** | `public` | `private` |
| Everything else | Same | Same |

### Example

```cpp
struct MyStruct {
    int x;  // public by default
};

class MyClass {
    int x;  // private by default
};
```

---

## Summary

| Concept | Key Point |
|---------|-----------|
| **C++ struct** | Can have methods, access specifiers, default values |
| **Default access** | public (unlike class which is private) |
| **Use case** | Simple data grouping, POD types |

---

> **Next Topic:** [13 - new and delete Operators](./13_New_and_Delete_Operators.md)
