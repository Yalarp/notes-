# 13 - new and delete Operators

## Table of Contents
1. [Introduction](#introduction)
2. [new Operator](#new-operator)
3. [delete Operator](#delete-operator)
4. [new vs malloc](#new-vs-malloc)
5. [Arrays with new/delete](#arrays-with-newdelete)
6. [Common Mistakes](#common-mistakes)
7. [Summary](#summary)

---

## Introduction

C++ provides `new` and `delete` operators for dynamic memory allocation, replacing C's malloc/free.

---

## new Operator

### Characteristics

```cpp
/*
new operator:
a) Similar to malloc or calloc
b) Allocates memory dynamically from heap (free store)
c) It is an OPERATOR, not a function
d) No need to use sizeof
e) new invokes CONSTRUCTOR (for objects)
*/
```

### Syntax

```cpp
type* ptr = new type;          // Single element
type* ptr = new type(value);   // With initialization
type* ptr = new type[size];    // Array
```

### Examples

```cpp
int* p1 = new int;        // Uninitialized int
int* p2 = new int(25);    // Initialized to 25
int* arr = new int[10];   // Array of 10 ints
```

---

## delete Operator

### Characteristics

```cpp
/*
delete operator:
a) Similar to free
b) Deallocates dynamically allocated memory
c) It is an OPERATOR, not a function
d) delete invokes DESTRUCTOR (for objects)
*/
```

### Syntax

```cpp
delete ptr;      // Single element
delete[] ptr;    // Array
```

---

## new vs malloc

| Feature | new | malloc |
|---------|-----|--------|
| Type | Operator | Function |
| Returns | Typed pointer | void* (needs cast) |
| sizeof | Not needed | Required |
| Constructor | Called | Not called |
| Destructor | Called by delete | Not called |
| Failure | throws bad_alloc | Returns NULL |

---

## Arrays with new/delete

### Complete Example

```cpp
#include <iostream>
using namespace std;

int main() {
    int *ptr, rec, i;
    
    cout << "\nHow many numbers?\n";
    cin >> rec;
    
    ptr = new int[rec];   // Allocate array
    
    cout << "\nEnter " << rec << " numbers\n";
    for(i = 0; i < rec; i++) {
        cin >> ptr[i];
    }
    
    cout << "\nDisplaying:\n";
    for(i = 0; i < rec; i++) {
        cout << ptr[i] << endl;
    }
    
    delete[] ptr;         // MUST use delete[] for arrays!
    return 0;
}
```

---

## Common Mistakes

### Mistake 1: Wrong delete for arrays

```cpp
int* arr = new int[10];
delete arr;      // ❌ WRONG!
delete[] arr;    // ✅ CORRECT!
```

### Mistake 2: Using after delete

```cpp
delete ptr;
cout << *ptr;    // ❌ Undefined behavior!
ptr = nullptr;   // ✅ Good practice
```

---

## Summary

| Concept | Key Point |
|---------|-----------|
| **new** | Allocates memory, calls constructor |
| **delete** | Frees memory, calls destructor |
| **new[]** | For array allocation |
| **delete[]** | MUST use for arrays |

---

> **Next Topic:** [14 - Inline Functions](./14_Inline_Functions.md)
