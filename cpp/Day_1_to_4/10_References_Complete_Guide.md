# 10 - References Complete Guide

## Table of Contents
1. [Introduction](#introduction)
2. [Reference Declaration](#reference-declaration)
3. [How References Work](#how-references-work)
4. [Call by Reference](#call-by-reference)
5. [Reference vs Pointer](#reference-vs-pointer)
6. [Reference to Array and Function](#reference-to-array-and-function)
7. [Function Returning Reference](#function-returning-reference)
8. [Const References](#const-references)
9. [Practice Questions](#practice-questions)
10. [Interview Questions](#interview-questions)

---

## Introduction

### What is a Reference?

A **reference** is an alias (another name) for an existing variable. It's NOT a separate variable.

### Key Properties

| Property | Description |
|----------|-------------|
| **Alias** | Just another name for existing variable |
| **No memory** | Reference is not allocated separate memory |
| **Must initialize** | Must be initialized when declared |
| **Cannot rebind** | Once initialized, cannot refer to another variable |
| **Auto-dereference** | No explicit `*` needed (unlike pointers) |

---

## Reference Declaration

### Syntax

```cpp
datatype& reference_name = variable;
```

### Example

```cpp
int num = 10;
int& ref = num;  // ref is a reference to num

cout << num << endl;  // Prints 10
cout << ref << endl;  // Prints 10 (same memory location)
```

### Line-by-Line Explanation

| Line | Code | What Happens |
|------|------|--------------|
| `int num = 10;` | Create variable `num` with value 10 |
| `int& ref = num;` | Create reference `ref` as alias for `num` |
| `cout << ref;` | Access num through its alias ref |

---

## How References Work

### Compiler Perspective

```cpp
int num = 10;
int& ref = num;
cout << ref << endl;
```

**What the compiler does:**
1. `ref` is treated as an alias for `num`
2. Every use of `ref` is replaced by `num` in generated code
3. `ref` doesn't exist as a separate variable in memory

### Memory Visualization

```
┌───────────────────────────────────────────────────────────────┐
│  REFERENCE IN MEMORY                                          │
├───────────────────────────────────────────────────────────────┤
│                                                               │
│  Memory:                                                      │
│  ┌─────────────┐                                             │
│  │    10       │ ← num (at address 0x1000)                   │
│  │             │ ← ref also refers to same location          │
│  └─────────────┘                                             │
│                                                               │
│  Both 'num' and 'ref' are names for the SAME memory!         │
│  There is NO separate storage for 'ref'.                     │
│                                                               │
└───────────────────────────────────────────────────────────────┘
```

---

## Call by Reference

### Using References for Function Parameters

```cpp
#include <iostream>
using namespace std;

void swap(int& a, int& b) {  // a and b are references
    int c = a;
    a = b;
    b = c;
}

int main() {
    int num1, num2;
    cout << "Enter two numbers:" << endl;
    cin >> num1 >> num2;
    
    cout << "Before call\t" << num1 << "\t" << num2 << endl;
    swap(num1, num2);   // Pass by reference (no & needed in call!)
    cout << "After call\t" << num1 << "\t" << num2 << endl;
    
    return 0;
}
```

### Line-by-Line Explanation

| Line | Code | Explanation |
|------|------|-------------|
| `void swap(int& a, int& b)` | Parameters are references, not copies |
| `swap(num1, num2);` | No `&` needed when calling (unlike pointers) |
| Inside swap | `a` is alias for `num1`, `b` is alias for `num2` |

### Benefits

| Benefit | Description |
|---------|-------------|
| **No copy** | Original variables used directly |
| **Memory efficient** | No extra memory for copies |
| **Clean syntax** | No `*` or `&` in function body |

---

## Reference vs Pointer

### Comparison

| Feature | Reference | Pointer |
|---------|-----------|---------|
| **Declaration** | `int& ref = var;` | `int* ptr = &var;` |
| **Must initialize** | Yes | No |
| **Can be NULL** | No | Yes |
| **Can rebind** | No | Yes |
| **Dereference** | Automatic | Manual with `*` |
| **Arithmetic** | Not possible | Possible |
| **Syntax** | Cleaner | More verbose |

### Code Comparison

```cpp
// Using Pointer
void swapPtr(int* a, int* b) {
    int temp = *a;   // Must dereference
    *a = *b;
    *b = temp;
}
swapPtr(&num1, &num2);  // Must pass addresses

// Using Reference
void swapRef(int& a, int& b) {
    int temp = a;    // Direct access
    a = b;
    b = temp;
}
swapRef(num1, num2);    // Direct pass
```

---

## Reference to Array and Function

### Reference to Array

```cpp
#include <iostream>
using namespace std;

int main() {
    int arr[3] = {10, 20, 30};
    int (&ref)[3] = arr;   // Reference to array of 3 ints
    
    for(int i = 0; i < 3; i++) {
        cout << ref[i] << endl;  // Access through reference
    }
    
    return 0;
}
```

### Reference to Function

```cpp
#include <iostream>
using namespace std;

void disp() {
    cout << "in disp" << endl;
}

int main() {
    void (&ref)() = disp;   // Reference to function
    
    ref();   // Call function through reference
    
    return 0;
}
```

---

## Function Returning Reference

### What Does This Enable?

A function returning a reference can be used on the **left side** of an assignment!

### Example

```cpp
#include <iostream>
using namespace std;

int num = 200;

int& disp() {        // Returns reference to num
    return num;
}

int main() {
    cout << "Before call\t" << num << endl;  // 200
    
    disp() = 400;    // Assign to the returned reference!
    
    cout << "After call\t" << num << endl;   // 400
    
    return 0;
}
```

### Line-by-Line Explanation

| Line | Code | What Happens |
|------|------|--------------|
| `int& disp()` | Function returns a reference (to num) |
| `return num;` | Returns the reference of global num |
| `disp() = 400;` | Assigns 400 to the returned reference (i.e., to num) |

### Execution Flow

```
┌───────────────────────────────────────────────────────────────┐
│ 1. num = 200 (global)                                         │
├───────────────────────────────────────────────────────────────┤
│ 2. disp() called, returns reference to num                    │
├───────────────────────────────────────────────────────────────┤
│ 3. disp() = 400 → assigns 400 to num                         │
├───────────────────────────────────────────────────────────────┤
│ 4. num is now 400                                             │
└───────────────────────────────────────────────────────────────┘
```

> **Warning:** Never return a reference to a local variable!

---

## Const References

### const vs non-const Reference

```cpp
int num = 10;

int& ref = num;         // Non-const: can modify
const int& cref = num;  // Const: read-only

ref = 20;     // OK
cref = 30;    // ❌ Error! Cannot modify through const reference
```

### When to Use Const References

- Pass large objects without copying
- Prevent modification of passed argument
- Commonly used for strings and large structures

```cpp
void display(const string& str) {
    cout << str << endl;
    // str = "new value";  // ❌ Error! Cannot modify
}
```

---

## Practice Questions

### Question 1: Swap using References

```cpp
void swap(int& a, int& b) {
    int temp = a;
    a = b;
    b = temp;
}
```

### Question 2: Why is this wrong?

```cpp
int& func() {
    int local = 10;
    return local;  // ❌ Returning reference to local variable!
}
```

**Answer:** `local` is destroyed when function returns. Reference becomes dangling.

---

## Interview Questions

### Q1: What is a reference?

**Answer:** A reference is an alias (alternative name) for a variable. It must be initialized when declared and cannot be rebound to refer to another variable.

### Q2: Can a reference be NULL?

**Answer:** No. Unlike pointers, references must always refer to a valid object.

### Q3: What is the difference between pass by reference and pass by value?

| Pass by Value | Pass by Reference |
|---------------|-------------------|
| Creates a copy | Uses original |
| Changes don't affect original | Changes affect original |
| Uses more memory | Memory efficient |

### Q4: When should you use references vs pointers?

**Use references when:**
- You always need a valid object
- You don't need to change what you're referring to

**Use pointers when:**
- NULL is a valid value
- You need to change what you're pointing to
- You need pointer arithmetic

---

## Summary

| Concept | Key Points |
|---------|------------|
| **Reference** | Alias for a variable, no separate memory |
| **Must initialize** | Cannot leave uninitialized |
| **Cannot rebind** | Once set, cannot change target |
| **Call by reference** | Efficient parameter passing |
| **Returning reference** | Allows assignment to function call |
| **vs Pointer** | Cleaner syntax, must be valid |

---

> **Next Topic:** [11 - Linkage in C++](./11_Linkage_in_Cpp.md)
