# 05 - Dynamic Memory Allocation

## Table of Contents
1. [Introduction](#introduction)
2. [Stack vs Heap Memory](#stack-vs-heap-memory)
3. [C-Style Memory Functions](#c-style-memory-functions)
4. [C++ new and delete Operators](#c-new-and-delete-operators)
5. [Memory Leak](#memory-leak)
6. [Dangling Pointer](#dangling-pointer)
7. [Best Practices](#best-practices)
8. [Practice Questions](#practice-questions)
9. [Interview Questions](#interview-questions)

---

## Introduction

### What is Dynamic Memory Allocation?

**Dynamic Memory Allocation (DMA)** is the process of allocating memory at **runtime** rather than at compile time. This allows programs to:

- Allocate memory based on user input
- Create data structures of variable size
- Manage memory efficiently

### Why Do We Need DMA?

| Static Allocation | Dynamic Allocation |
|-------------------|-------------------|
| Size fixed at compile time | Size determined at runtime |
| Memory on stack | Memory on heap |
| Automatically released | Must be manually released |
| Limited size | Large memory possible |

---

## Stack vs Heap Memory

### Memory Layout

```
┌─────────────────────────────────────────────────────────────────┐
│                    PROGRAM MEMORY LAYOUT                        │
└─────────────────────────────────────────────────────────────────┘

    HIGH ADDRESS
    ┌───────────────────────────────┐
    │           STACK               │ ← Local variables
    │   (Grows Downward ↓)          │   Function parameters
    │   Auto-managed                │   Limited size (~1-8MB)
    │                               │
    ├───────────────────────────────┤
    │                               │
    │           HEAP                │ ← Dynamic allocation
    │   (Grows Upward ↑)            │   Manual management
    │   Large size available        │   malloc, new, etc.
    │                               │
    ├───────────────────────────────┤
    │      DATA / BSS               │ ← Global, static variables
    ├───────────────────────────────┤
    │           TEXT                │ ← Program code
    └───────────────────────────────┘
    LOW ADDRESS
```

### Comparison

| Feature | Stack | Heap |
|---------|-------|------|
| **Allocation** | Automatic | Manual |
| **Deallocation** | Automatic | Manual |
| **Size** | Limited (1-8 MB) | Large (GBs) |
| **Speed** | Fast | Slower |
| **Fragmentation** | No | Yes |
| **Lifetime** | Until scope ends | Until explicitly freed |

---

## C-Style Memory Functions

### Functions in `<cstdlib>` (previously `<malloc.h>`)

| Function | Purpose | Syntax |
|----------|---------|--------|
| `malloc` | Allocate memory | `void* malloc(size_t size)` |
| `calloc` | Allocate & initialize to 0 | `void* calloc(size_t num, size_t size)` |
| `realloc` | Resize allocation | `void* realloc(void* ptr, size_t new_size)` |
| `free` | Deallocate memory | `void free(void* ptr)` |

---

### malloc Example

```cpp
#include <iostream>
#include <cstdlib>    // For malloc, free
using namespace std;

int main() {
    int *ptr, i, rec;
    
    puts("How many numbers?");
    scanf("%d", &rec);                              // Line 1: Get count from user
    
    ptr = (int*)malloc(rec * sizeof(int));          // Line 2: Allocate memory
    
    if(ptr == NULL) {                               // Line 3: Check allocation
        puts("Memory allocation failed!");
        return 1;
    }
    
    printf("Enter %d values\n", rec);
    for(i = 0; i < rec; i++) {
        scanf("%d", &ptr[i]);                       // Line 4: Read values
    }
    
    puts("Displaying:");
    for(i = 0; i < rec; i++) {
        printf("%d\n", ptr[i]);                     // Line 5: Display values
    }
    
    free(ptr);                                      // Line 6: IMPORTANT: Release memory!
    puts("Done");
    
    return 0;
}
```

### Line-by-Line Explanation

| Line | Code | Explanation |
|------|------|-------------|
| 1 | `scanf("%d", &rec);` | Get number of integers from user |
| 2 | `ptr = (int*)malloc(rec * sizeof(int));` | Allocate `rec × 4` bytes; cast to `int*` |
| 3 | `if(ptr == NULL)` | Check if allocation succeeded |
| 4 | `scanf("%d", &ptr[i]);` | Store values in allocated memory |
| 5 | `printf("%d\n", ptr[i]);` | Access values via pointer |
| 6 | `free(ptr);` | **Critical:** Release memory back to system |

### Execution Flow

```
┌───────────────────────────────────────────────────────────────┐
│ 1. User inputs: rec = 3                                      │
├───────────────────────────────────────────────────────────────┤
│ 2. malloc(3 * 4) = malloc(12 bytes)                          │
│    → Operating system finds 12 bytes on HEAP                 │
│    → Returns address (e.g., 0x7FF8)                          │
│    → ptr = 0x7FF8                                            │
├───────────────────────────────────────────────────────────────┤
│ 3. User enters: 10, 20, 30                                   │
│    → ptr[0] = 10 (at 0x7FF8)                                 │
│    → ptr[1] = 20 (at 0x7FFC)                                 │
│    → ptr[2] = 30 (at 0x8000)                                 │
├───────────────────────────────────────────────────────────────┤
│ 4. Display: 10, 20, 30                                       │
├───────────────────────────────────────────────────────────────┤
│ 5. free(ptr)                                                 │
│    → Memory at 0x7FF8 returned to heap                       │
└───────────────────────────────────────────────────────────────┘
```

---

### malloc for String Example

```cpp
#include <iostream>
#include <cstdlib>
using namespace std;

int main() {
    char *ptr;
    int rec, i;
    
    cout << "How many characters?" << endl;
    cin >> rec;
    
    // Allocate space for characters + null terminator
    ptr = (char*)malloc((rec + 1) * sizeof(char));
    
    for(i = 0; i < rec; i++) {
        cin >> ptr[i];
    }
    ptr[i] = '\0';    // Add null terminator
    
    cout << ptr << endl;
    
    free(ptr);
    cout << "Done" << endl;
    
    return 0;
}
```

---

## C++ new and delete Operators

### Comparison with malloc/free

| Feature | malloc/free | new/delete |
|---------|-------------|------------|
| **Type** | Functions | Operators |
| **Returns** | void* (needs casting) | Typed pointer |
| **sizeof** | Required | Not required |
| **Constructor** | Not called | Called |
| **Destructor** | Not called | Called |
| **Failure** | Returns NULL | Throws exception |

---

### new Operator Characteristics

```cpp
/*
new operator:
a) Similar to malloc or calloc
b) Allocates memory dynamically from heap (free store)
c) It is an OPERATOR, not a function
d) No need to use sizeof
e) new invokes CONSTRUCTOR
*/
```

### delete Operator Characteristics

```cpp
/*
delete operator:
a) Similar to free
b) Deallocates dynamically allocated memory
c) It is an OPERATOR, not a function
d) delete invokes DESTRUCTOR
*/
```

---

### new and delete for Single Variable

```cpp
int *ptr = new int;        // Allocate single int
*ptr = 10;                 // Assign value
cout << *ptr << endl;      // Print: 10
delete ptr;                // Deallocate

// Or with initialization
int *ptr = new int(25);    // Allocate and initialize to 25
cout << *ptr << endl;      // Print: 25
delete ptr;
```

---

### new and delete for Arrays

```cpp
#include <iostream>
using namespace std;

int main() {
    int *ptr, rec, i;
    
    cout << "\nHow many numbers?\n";
    cin >> rec;
    
    ptr = new int[rec];                    // Allocate array
    
    cout << "\nEnter " << rec << " numbers\n";
    for(i = 0; i < rec; i++) {
        cin >> ptr[i];
    }
    
    cout << "\nDisplaying all numbers\n";
    for(i = 0; i < rec; i++) {
        cout << ptr[i] << endl;
    }
    
    // delete(ptr);    // WRONG for arrays!
    // delete ptr;     // WRONG for arrays!
    delete[] ptr;       // CORRECT for arrays!
    
    return 0;
}
```

### Line-by-Line Explanation

| Line | Code | Explanation |
|------|------|-------------|
| `ptr = new int[rec];` | Allocates `rec` integers on heap |
| `cin >> ptr[i];` | Store user input in array |
| `delete[] ptr;` | Deallocate ENTIRE array (must use `[]`) |

> **Critical:** Use `delete[]` for arrays allocated with `new[]`!

---

## Memory Leak

### What is Memory Leak?

A **memory leak** occurs when allocated heap memory is never freed, causing the program to gradually consume more memory.

### Example of Memory Leak

```cpp
void disp() {
    int *ptr = new int(5);    // Allocate memory
    // Function returns without freeing memory!
    // ptr goes out of scope, but the memory remains allocated
}

int main() {
    disp();
    // Memory is LEAKED! Cannot be recovered until program ends
    // ... lots of other stuff
}
```

### Visual Representation

```
┌───────────────────────────────────────────────────────────────┐
│  MEMORY LEAK                                                  │
├───────────────────────────────────────────────────────────────┤
│                                                               │
│  Stack:                  Heap:                                │
│  ┌─────────┐            ┌───────────┐                        │
│  │  ptr    │───────────▶│     5     │                        │
│  │ 0x1000  │            │  @0x1000  │                        │
│  └─────────┘            └───────────┘                        │
│     ↓ (destroyed)              ↑                              │
│                         (still allocated!)                    │
│                                                               │
│  After disp() returns:                                        │
│  • ptr is destroyed (was on stack)                           │
│  • Memory @0x1000 STILL ALLOCATED but unreachable!           │
│  • This memory cannot be freed → MEMORY LEAK                 │
│                                                               │
└───────────────────────────────────────────────────────────────┘
```

### How to Fix

```cpp
void disp() {
    int *ptr = new int(5);
    // ... use ptr ...
    delete ptr;               // FREE memory before function ends!
}
```

---

## Dangling Pointer

### What is a Dangling Pointer?

A **dangling pointer** is a pointer that points to memory that has been freed/deleted but the pointer still holds the old address.

### Example of Dangling Pointer

```cpp
int main() {
    int *ptr = new int(5);    // Allocate memory
    
    // Use the pointer
    cout << *ptr << endl;      // OK: prints 5
    
    delete ptr;                // Free memory
    
    // ptr is now DANGLING! It still holds the address
    // but that memory is no longer valid
    
    cout << *ptr << endl;      // UNDEFINED BEHAVIOR!
    *ptr = 10;                 // CRASH or data corruption!
}
```

### Visual Representation

```
┌───────────────────────────────────────────────────────────────┐
│  DANGLING POINTER                                             │
├───────────────────────────────────────────────────────────────┤
│                                                               │
│  Before delete:              After delete:                    │
│                                                               │
│  Stack:     Heap:            Stack:     Heap:                 │
│  ┌─────────┐ ┌─────────┐     ┌─────────┐ ┌─────────┐         │
│  │ptr=1000 │◀│    5    │     │ptr=1000 │ │ (freed) │         │
│  └─────────┘ └─────────┘     └─────────┘ └─────────┘         │
│      │           ↑               │          ✗                │
│      └───────────┘               └──────────┤                 │
│                                    DANGLING! │                │
│                                                               │
└───────────────────────────────────────────────────────────────┘
```

### How to Fix

```cpp
int main() {
    int *ptr = new int(5);
    
    // Use the pointer
    cout << *ptr << endl;
    
    delete ptr;
    ptr = nullptr;            // Set to nullptr after delete!
    
    if(ptr != nullptr) {
        cout << *ptr << endl; // This won't execute
    }
}
```

---

## Best Practices

### The "Rule of Thumb"

```cpp
// For every new, there must be a delete
int *ptr = new int;
// ... use ...
delete ptr;
ptr = nullptr;

// For every new[], there must be a delete[]
int *arr = new int[10];
// ... use ...
delete[] arr;
arr = nullptr;
```

### Summary of Best Practices

| Practice | Reason |
|----------|--------|
| Always check for NULL after malloc | Allocation can fail |
| Always delete/free allocated memory | Prevent memory leaks |
| Set pointer to nullptr after delete | Prevent dangling pointer |
| Use `delete[]` for arrays | Deallocates entire array |
| Prefer smart pointers (C++11+) | Automatic memory management |

---

## Practice Questions

### Question 1: Allocate and Display Array Using new

```cpp
#include <iostream>
using namespace std;

int main() {
    int n;
    cout << "Enter size: ";
    cin >> n;
    
    int *arr = new int[n];
    
    cout << "Enter " << n << " values: ";
    for(int i = 0; i < n; i++) {
        cin >> arr[i];
    }
    
    cout << "Values: ";
    for(int i = 0; i < n; i++) {
        cout << arr[i] << " ";
    }
    cout << endl;
    
    delete[] arr;
    arr = nullptr;
    
    return 0;
}
```

---

## Interview Questions

### Q1: What is the difference between malloc and new?

| malloc | new |
|--------|-----|
| Function | Operator |
| Returns void* | Returns typed pointer |
| Requires sizeof | No sizeof needed |
| Doesn't call constructor | Calls constructor |
| Returns NULL on failure | Throws bad_alloc exception |

### Q2: What happens if you use delete instead of delete[]?

**Answer:** Undefined behavior! Only the first destructor might be called, and memory may not be properly deallocated. Always use `delete[]` for arrays.

### Q3: How to detect memory leaks?

**Answer:**
- Use Valgrind (Linux)
- Use Visual Studio's Memory Diagnostics
- Use AddressSanitizer
- Use smart pointers (C++11+)

### Q4: What is a memory pool?

**Answer:** A pre-allocated block of memory from which smaller allocations are made. This reduces fragmentation and improves allocation speed.

---

## Summary

| Concept | Key Points |
|---------|------------|
| **DMA** | Allocate memory at runtime |
| **malloc/free** | C-style, returns void*, no constructor |
| **new/delete** | C++ style, type-safe, calls constructor/destructor |
| **new[]/delete[]** | For array allocation |
| **Memory Leak** | Allocated memory never freed |
| **Dangling Pointer** | Pointer to freed memory |
| **Best Practice** | Always delete, set to nullptr |

---

> **Next Topic:** [06 - Static and Dynamic Libraries](./06_Static_and_Dynamic_Libraries.md)
