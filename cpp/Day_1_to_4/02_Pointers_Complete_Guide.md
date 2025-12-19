# 02 - Pointers Complete Guide

## Table of Contents
1. [Introduction to Pointers](#introduction-to-pointers)
2. [Pointer Declaration and Initialization](#pointer-declaration-and-initialization)
3. [Pointer Operators](#pointer-operators)
4. [Pointer Arithmetic](#pointer-arithmetic)
5. [Pointers and Functions](#pointers-and-functions)
6. [Pointer to Function](#pointer-to-function)
7. [Pointers and Arrays](#pointers-and-arrays)
8. [Array of Pointers](#array-of-pointers)
9. [Double Pointer](#double-pointer)
10. [Dynamic Memory Allocation](#dynamic-memory-allocation)
11. [Advanced Pointer Expressions](#advanced-pointer-expressions)
12. [Practice Questions](#practice-questions)
13. [Interview Questions](#interview-questions)

---

## Introduction to Pointers

### What is a Pointer?

A **pointer** is a variable that holds the **memory address** of another variable.

```
┌─────────────────────────────────────────────────────────────────┐
│                    POINTER CONCEPT                              │
└─────────────────────────────────────────────────────────────────┘

    Variable                    Pointer
    ┌──────────┐              ┌──────────────┐
    │   num1   │              │     ptr1     │
    │    20    │◄─────────────│   0x1000     │
    │          │              │              │
    └──────────┘              └──────────────┘
    Address: 0x1000           Holds address 0x1000
```

### Why Use Pointers?

| Purpose | Description |
|---------|-------------|
| **Dynamic Memory** | Allocate memory at runtime |
| **Efficient Parameters** | Pass large data by reference |
| **Data Structures** | Implement linked lists, trees |
| **Hardware Access** | Direct memory manipulation |
| **Function Pointers** | Callbacks and dynamic dispatch |

### Pointer Size

> **Important:** Any type of pointer will occupy **8 bytes** inside a 64-bit memory address space (4 bytes in 32-bit systems).

```cpp
int* ptr1;      // 8 bytes on 64-bit system
char* ptr2;     // 8 bytes on 64-bit system
float* ptr3;    // 8 bytes on 64-bit system
double* ptr4;   // 8 bytes on 64-bit system
```

---

## Pointer Declaration and Initialization

### Syntax

```cpp
datatype *pointervariable;
```

### Declaration Examples

```cpp
int *ptr1;      // ptr1 is a pointer variable which can point to int
char *ptr2;     // ptr2 is a pointer variable which can point to char
float *ptr3;    // ptr3 is a pointer variable which can point to float
```

### Line-by-Line Explanation

```cpp
int *ptr1;
```

| Part | Meaning |
|------|---------|
| `int` | The datatype that ptr1 will point to |
| `*` | Indicates this is a pointer variable |
| `ptr1` | The name of the pointer variable |

### Initialization Example

```cpp
int num1 = 20;      // Line 1: Declare and initialize integer
int *ptr1;          // Line 2: Declare pointer
ptr1 = &num1;       // Line 3: Assign address of num1 to ptr1
```

**Or in one line:**
```cpp
int num1 = 20;
int *ptr1 = &num1;  // Initialization of pointer
```

---

## Pointer Operators

### Two Essential Operators

| Operator | Name | Description |
|----------|------|-------------|
| `&` | Address-of | Returns the memory address of a variable |
| `*` | Dereference | Returns the value at the address contained by pointer |

### Code Example

```cpp
#include <iostream>
using namespace std;

int main() {
    int num1 = 20;          // Line 1: num1 contains value 20
    int *ptr1;              // Line 2: Declare pointer ptr1
    ptr1 = &num1;           // Line 3: ptr1 now holds address of num1
    
    printf("%d\n", num1);   // Line 4: Prints 20 (direct access)
    printf("%d\n", *ptr1);  // Line 5: Prints 20 (via pointer)
    
    return 0;
}
```

### Line-by-Line Explanation

| Line | Code | What Happens |
|------|------|--------------|
| 1 | `int num1 = 20;` | Variable `num1` is created in memory with value 20 |
| 2 | `int *ptr1;` | Pointer `ptr1` is declared (currently contains garbage) |
| 3 | `ptr1 = &num1;` | `&num1` gets address of num1, stores in ptr1 |
| 4 | `printf("%d\n", num1);` | Directly prints value of num1 → 20 |
| 5 | `printf("%d\n", *ptr1);` | `*ptr1` dereferences pointer, gets value at that address → 20 |

### Memory Visualization

```
Memory Layout:
══════════════════════════════════════════════════════════════

Address       Variable    Value
─────────────────────────────────────
0x1000        num1        20
0x1008        ptr1        0x1000  (holds address of num1)

══════════════════════════════════════════════════════════════

When we say *ptr1:
1. Look at ptr1 → contains 0x1000
2. Go to address 0x1000
3. Read value there → 20
```

---

## Pointer Arithmetic

### Rules

1. Pointers can only be **incremented or decremented**
2. Two pointers can be **subtracted** (must be same type)
3. Pointer arithmetic is **scaled** by the size of the datatype

### The Formula

```
New Address = pointer_value ± (specified_number × scale_factor)
```

Where **scale factor** = bytes occupied by the datatype

| Datatype | Scale Factor (64-bit) |
|----------|----------------------|
| char | 1 byte |
| int | 4 bytes |
| float | 4 bytes |
| double | 8 bytes |
| pointer | 8 bytes |

### Code Example

```cpp
int num1 = 50;
int *ptr1 = &num1;      // ptr1 points to num1

printf("%d\n", *ptr1);  // Prints 50

ptr1++;                 // Pointer arithmetic!
                        // ptr1 now points 4 bytes ahead
```

### Visual Explanation

```
Before ptr1++:
┌─────────────────────────────────────────────────────────────────┐
│  Address    │ 0x1000 │ 0x1004 │ 0x1008 │ 0x100C │ 0x1010 │      │
│  Value      │   50   │   ??   │   ??   │   ??   │   ??   │      │
│             │   ↑    │        │        │        │        │      │
│             │ ptr1   │        │        │        │        │      │
└─────────────────────────────────────────────────────────────────┘

After ptr1++ (int pointer, so +4 bytes):
┌─────────────────────────────────────────────────────────────────┐
│  Address    │ 0x1000 │ 0x1004 │ 0x1008 │ 0x100C │ 0x1010 │      │
│  Value      │   50   │   ??   │   ??   │   ??   │   ??   │      │
│             │        │   ↑    │        │        │        │      │
│             │        │ ptr1   │        │        │        │      │
└─────────────────────────────────────────────────────────────────┘

New address = 0x1000 + (1 × 4) = 0x1004
```

> **Note:** Pointer arithmetic is most useful in case of **arrays only**.

### Pointer Subtraction

Two pointers of the **same type** can be subtracted:

```cpp
int *ptr1, *ptr2;
// ... assign values ...

int diff = ptr1 - ptr2;   // Gives number of elements between them
```

**Formula:**
```
Result = (first_pointer - second_pointer) / scale_factor
```

---

## Pointers and Functions

### Call by Value vs Call by Address

#### Call by Value (Default)

```cpp
#include <iostream>
using namespace std;

void swap(int x, int y) {    // Line 1: Receives COPIES
    int temp = x;            // Line 2: temp = copy of num1
    x = y;                   // Line 3: x = copy of num2
    y = temp;                // Line 4: y = temp (swaps copies only!)
}

int main() {
    int num1 = 5, num2 = 8;
    
    printf("Before %d\t%d\n", num1, num2);  // 5    8
    swap(num1, num2);
    printf("After %d\t%d\n", num1, num2);   // 5    8 (NO CHANGE!)
    
    return 0;
}
```

**Execution Flow:**
```
┌──────────────────────────────────────────────────────────────┐
│ 1. main(): num1=5, num2=8                                   │
├──────────────────────────────────────────────────────────────┤
│ 2. swap(num1, num2) called                                  │
│    → COPIES of 5 and 8 are created in x and y               │
├──────────────────────────────────────────────────────────────┤
│ 3. Inside swap(): x and y are swapped (5↔8)                │
│    → Only the COPIES are modified!                          │
├──────────────────────────────────────────────────────────────┤
│ 4. swap() returns → copies are destroyed                    │
├──────────────────────────────────────────────────────────────┤
│ 5. Original num1 and num2 are UNCHANGED!                    │
└──────────────────────────────────────────────────────────────┘
```

#### Call by Address (Using Pointers)

```cpp
#include <iostream>
using namespace std;

void swap(int *x, int *y) {  // Line 1: Receives ADDRESSES
    int temp = *x;           // Line 2: temp = value at x's address
    *x = *y;                 // Line 3: value at x = value at y
    *y = temp;               // Line 4: value at y = temp
}

int main() {
    int num1 = 5, num2 = 8;
    
    printf("Before %d\t%d\n", num1, num2);  // 5    8
    swap(&num1, &num2);                      // Pass ADDRESSES
    printf("After %d\t%d\n", num1, num2);   // 8    5 (SWAPPED!)
    
    return 0;
}
```

### Line-by-Line Explanation

| Line | Code | Explanation |
|------|------|-------------|
| 1 | `void swap(int *x, int *y)` | Function receives two pointer parameters (addresses) |
| 2 | `int temp = *x;` | `*x` gets value at address x (which is 5), stores in temp |
| 3 | `*x = *y;` | Value at address x = value at address y (num1 becomes 8) |
| 4 | `*y = temp;` | Value at address y = temp (num2 becomes 5) |

**Execution Flow:**
```
┌──────────────────────────────────────────────────────────────┐
│ 1. main(): num1=5 at addr 0x1000, num2=8 at addr 0x1008    │
├──────────────────────────────────────────────────────────────┤
│ 2. swap(&num1, &num2): x=0x1000, y=0x1008                  │
├──────────────────────────────────────────────────────────────┤
│ 3. temp = *x → temp = value at 0x1000 → temp = 5           │
├──────────────────────────────────────────────────────────────┤
│ 4. *x = *y → value at 0x1000 = value at 0x1008 → [5→8]     │
├──────────────────────────────────────────────────────────────┤
│ 5. *y = temp → value at 0x1008 = 5 → [8→5]                 │
├──────────────────────────────────────────────────────────────┤
│ 6. ORIGINAL variables are modified!                         │
└──────────────────────────────────────────────────────────────┘
```

#### Mixed: Call by Value and Call by Address

```cpp
void change(int x, int *y) {  // x: by value, y: by address
    x *= -1;                   // Modifies LOCAL copy only
    *y *= -1;                  // Modifies ORIGINAL variable
}

int main() {
    int num1 = -5, num2 = -8;
    
    printf("Before %d\t%d\n", num1, num2);  // -5   -8
    change(num1, &num2);
    printf("After %d\t%d\n", num1, num2);   // -5   8
    
    return 0;
}
```

**Result:** Only `num2` changes because it was passed by address!

---

## Pointer to Function

### What is a Pointer to Function?

A pointer that points to a function (stores the address of a function).

### Syntax

```cpp
returntype (*pointervariable)(arguments);
```

### Examples

```cpp
int (*ptr)(int);
// ptr is a pointer to a function that accepts int and returns int

void (*ptr)(int, char);
// ptr is a pointer to a function that accepts int and char, returns nothing

int (*ptr)();
// ptr is a pointer to a function that accepts nothing and returns int

void (*ptr)();
// ptr is a pointer to a function that accepts nothing and returns nothing
```

### Complete Example

```cpp
#include <iostream>
using namespace std;

int sqr(int k) {              // Line 1: Function definition
    return k * k;             // Line 2: Returns square
}

int main() {
    int sqr(int);             // Line 3: Function declaration
    int num = 5, res;         // Line 4: Variables
    int (*ptr)(int);          // Line 5: Function pointer declaration
    
    ptr = sqr;                // Line 6: Assign function address to pointer
    
    // Three ways to call the function:
    res = sqr(num);           // Line 7: Direct call
    // OR
    res = ptr(num);           // Line 8: Using pointer (same syntax)
    // OR
    res = (*ptr)(num);        // Line 9: Explicit dereference
    
    printf("%d\n", res);      // Line 10: Prints 25
    
    return 0;
}
```

### Line-by-Line Explanation

| Line | Code | Explanation |
|------|------|-------------|
| 5 | `int (*ptr)(int);` | Declare pointer to function that takes int, returns int |
| 6 | `ptr = sqr;` | Function name IS its address; assign to pointer |
| 7 | `res = sqr(num);` | Normal function call |
| 8 | `res = ptr(num);` | Call using pointer (shorthand) |
| 9 | `res = (*ptr)(num);` | Call using explicit dereference |

---

## Pointers and Arrays

### Key Relationship

**Array name is also a kind of pointer!** (But a constant pointer)

```cpp
int arr[4] = {10, 20, 30, 40};
int *ptr;

ptr = arr;        // Valid! arr is address of first element
// OR
ptr = &arr[0];    // Same thing, more explicit
```

### Accessing Array Elements

There are **four equivalent ways** to access array elements:

```cpp
int arr[4] = {10, 20, 30, 40};
int *ptr = arr;

for(int i = 0; i < 4; i++) {
    printf("%d\n", arr[i]);     // Method 1: Array subscript
    printf("%d\n", *(arr + i)); // Method 2: Array pointer arithmetic
    printf("%d\n", ptr[i]);     // Method 3: Pointer subscript
    printf("%d\n", *(ptr + i)); // Method 4: Pointer arithmetic
}
```

### Visual Memory Layout

```
Array: arr[4] = {10, 20, 30, 40}

Index:     [0]        [1]        [2]        [3]
Address:   0x1000     0x1004     0x1008     0x100C
Value:     10         20         30         40
           ↑
         arr/ptr

arr[0] = *(arr+0) = ptr[0] = *(ptr+0) = 10
arr[1] = *(arr+1) = ptr[1] = *(ptr+1) = 20
arr[2] = *(arr+2) = ptr[2] = *(ptr+2) = 30
arr[3] = *(arr+3) = ptr[3] = *(ptr+3) = 40
```

### Difference: Array Name vs Pointer

```cpp
int arr[4];
int *ptr = arr;

arr++;    // ❌ ERROR! Array name is CONSTANT pointer
ptr++;    // ✅ Works! Pointer can be modified
```

### Traversing with While Loop

```cpp
int arr[4] = {10, 20, 30, 40};
int *ptr = arr;

while(ptr < arr + 4) {
    printf("%d\n", *ptr++);  // Print value, then increment pointer
}

ptr = arr;  // Reset pointer to beginning
```

### Passing Array to Function

```cpp
void disp(int *ptr) {            // Receives pointer
    for(int i = 0; i < 4; i++) {
        printf("%d\n", ptr[i]);  // Access via subscript
        // OR
        printf("%d\n", *ptr++);  // Access and move forward
    }
}

int main() {
    int arr[4] = {10, 20, 30, 40};
    
    disp(arr);        // Pass array name (address of first element)
    // OR
    disp(&arr[0]);    // Explicitly pass address of first element
    
    return 0;
}
```

### Returning Array from Function

```cpp
// WRONG WAY - Undefined Behavior!
int* fun() {
    int arr[4] = {10, 20, 30, 40};  // Local array
    return arr;      // Returns address of local array (DESTROYED after return!)
}

// CORRECT WAY - Use static
int* fun() {
    static int arr[4] = {10, 20, 30, 40};  // Static persists!
    return arr;
}

int main() {
    int* ptr = fun();
    for(int i = 0; i < 4; i++) {
        printf("%d\n", ptr[i]);  // Safe to access
    }
    return 0;
}
```

---

## Array of Pointers

### What is an Array of Pointers?

An array that contains addresses instead of regular values.

### Example

```cpp
int a = 10, b = 20, c = 30;
int* arr[] = {&a, &b, &c};  // Array of pointers

for(int i = 0; i < 3; i++) {
    cout << *arr[i] << endl;  // Dereference each pointer
}
```

### Memory Visualization

```
Variables:
┌────────────┬────────────┬────────────┐
│   a = 10   │   b = 20   │   c = 30   │
│  @0x1000   │  @0x1004   │  @0x1008   │
└────────────┴────────────┴────────────┘

Array of Pointers:
┌────────────┬────────────┬────────────┐
│  arr[0]    │  arr[1]    │  arr[2]    │
│  0x1000    │  0x1004    │  0x1008    │
│     ↓      │     ↓      │     ↓      │
│    10      │    20      │    30      │
└────────────┴────────────┴────────────┘
```

---

## Double Pointer

### What is a Double Pointer?

A pointer that points to another pointer.

### Example

```cpp
int num1 = 300;
int *ptr1 = &num1;    // ptr1 points to num1
int **ptr2 = &ptr1;   // ptr2 points to ptr1 (double pointer!)

cout << num1 << endl;    // 300 (direct access)
cout << *ptr1 << endl;   // 300 (one dereference)
cout << **ptr2 << endl;  // 300 (two dereferences)
```

### Memory Visualization

```
┌─────────────────────────────────────────────────────────────────┐
│                    DOUBLE POINTER                               │
└─────────────────────────────────────────────────────────────────┘

    num1           ptr1            ptr2
   ┌──────┐      ┌──────┐       ┌──────┐
   │ 300  │◄─────│0x1000│◄──────│0x1008│
   └──────┘      └──────┘       └──────┘
   @0x1000       @0x1008        @0x1010

   *ptr1 = 300        (go to 0x1000, get 300)
   *ptr2 = 0x1000     (go to 0x1008, get 0x1000)
   **ptr2 = 300       (go to 0x1008→0x1000, get 300)
```

---

## Dynamic Memory Allocation

### Functions in malloc.h

| Function | Purpose |
|----------|---------|
| `malloc` | Allocate new memory |
| `calloc` | Allocate and initialize to zero |
| `realloc` | Resize already allocated memory |
| `free` | Deallocate memory |

### Complete Example with malloc

```cpp
#include <iostream>
#include <cstdlib>    // For malloc, free
using namespace std;

int main() {
    int *ptr, i, rec;
    
    puts("How many numbers?");
    scanf("%d", &rec);
    
    // Allocate memory for 'rec' integers
    ptr = (int*)malloc(rec * sizeof(int));
    
    printf("Enter %d values\n", rec);
    for(i = 0; i < rec; i++) {
        scanf("%d", &ptr[i]);
    }
    
    puts("Displaying:");
    for(i = 0; i < rec; i++) {
        printf("%d\n", ptr[i]);
    }
    
    free(ptr);  // IMPORTANT: Release memory!
    puts("Done");
    
    return 0;
}
```

### Line-by-Line Explanation

| Line | Code | Explanation |
|------|------|-------------|
| `ptr = (int*)malloc(rec * sizeof(int))` | Allocates `rec × 4` bytes on heap, returns pointer |
| `free(ptr)` | Releases allocated memory back to system |

---

## Advanced Pointer Expressions

### Understanding Operator Precedence

| Expression | Operation Order | Result |
|------------|-----------------|--------|
| `*ptr++` | Dereference, then increment pointer | Get value, move pointer |
| `(*ptr)++` | Dereference, then increment value | Get value, increase it after |
| `++*ptr` | Dereference, pre-increment value | Increase value, return new |
| `*++ptr` | Increment pointer, then dereference | Move pointer, get value |

### Example 1: `*ptr++`

```cpp
char name[20] = "Rajesh";
char* ptr = name;
cout << *ptr++ << "\t" << *ptr << endl;

// Output: R    a
```

**Explanation:**
1. `*ptr` gives 'R' (printed first)
2. `ptr++` moves pointer to next character 'a' (printed second)

### Example 2: `(*ptr)++`

```cpp
int arr[4] = {10, 20, 30, 40};
int* ptr = arr;
cout << (*ptr)++ << "\t" << *ptr << endl;

// Output: 10    11
```

**Explanation:**
1. `*ptr` gives 10 (printed)
2. `(*ptr)++` increments the VALUE at ptr from 10 to 11

### Example 3: `++*ptr`

```cpp
int arr[4] = {10, 20, 30, 40};
int* ptr = arr;
cout << ++(*ptr) << "\t" << *ptr << endl;

// Output: 11    11
```

**Explanation:**
1. `*ptr` is 10, `++` makes it 11 BEFORE printing
2. Both outputs show 11

### Example 4: `++*ptr++`

```cpp
int arr[4] = {10, 20, 30, 40};
int* ptr = arr;
cout << ++*ptr++ << "\t" << *ptr << endl;

// Output: 11    20
```

**Explanation:**
1. `*ptr` gets 10
2. `++` (left) increments value to 11 (printed)
3. `++` (right) moves pointer to next element (20)

### Example 5: Complex Expression

```cpp
int missile(int *ip) {
    return ++++(*ip) * (*ip);  // Increment twice, then multiply
}

int main() {
    int i = 4;
    cout << i << ":" << missile(&i) << ":" << *(&i) << endl;
}

// Output: 4:36:6
```

**Step-by-step:**
1. `i = 4` initially, printed first
2. In `missile`: `++++(*ip)` increments i twice: 4→5→6
3. `6 * 6 = 36` (returned)
4. Final value of i is 6

---

## Practice Questions

### Question 1: Swap Two Numbers Using Pointers

```cpp
#include <iostream>
using namespace std;

void swap(int *ptr1, int *ptr2) {
    int temp = *ptr1;
    *ptr1 = *ptr2;
    *ptr2 = temp;
}

int main() {
    int num1, num2;
    cout << "Enter 2 numbers: ";
    cin >> num1 >> num2;
    
    cout << "Before: " << num1 << " " << num2 << endl;
    swap(&num1, &num2);
    cout << "After: " << num1 << " " << num2 << endl;
    
    return 0;
}
```

### Question 2: Toggle Case of String

```cpp
void toggle(char *ptr) {
    for(int i = 0; ptr[i] != '\0'; i++) {
        if(ptr[i] >= 65 && ptr[i] <= 90) {      // Uppercase
            ptr[i] += 32;                        // Convert to lowercase
        }
        else if(ptr[i] >= 97 && ptr[i] <= 122) { // Lowercase
            ptr[i] -= 32;                        // Convert to uppercase
        }
    }
}

int main() {
    char str[30];
    cout << "Enter your name: ";
    gets_s(str);
    
    cout << "Before: " << str << endl;
    toggle(str);
    cout << "After: " << str << endl;
    
    return 0;
}
```

---

## Interview Questions

### Q1: What is the difference between `*ptr++` and `(*ptr)++`?

| Expression | Meaning | Effect |
|------------|---------|--------|
| `*ptr++` | Get value, then move pointer | Pointer moves to next element |
| `(*ptr)++` | Get value, then increment value | Value at current location increases |

### Q2: Why can't we do `arr++` for an array?

**Answer:** An array name is a **constant pointer**. Its address is fixed at compile time and cannot be changed. You can do `ptr++` on a pointer variable, but not on an array name.

### Q3: What is a null pointer?

**Answer:** A pointer that doesn't point to any valid memory location. It's initialized to `NULL` or `nullptr` (C++11):
```cpp
int *ptr = NULL;      // C style
int *ptr = nullptr;   // C++11 style
```

### Q4: What is a dangling pointer?

**Answer:** A pointer that points to memory that has been freed or gone out of scope:
```cpp
int* ptr = new int(5);
delete ptr;           // Memory freed
// ptr is now dangling! Using it causes undefined behavior
```

### Q5: What is the difference between `int *p` and `int* p`?

**Answer:** They are **exactly the same**. The placement of `*` doesn't matter. However, be careful with multiple declarations:
```cpp
int *p1, p2;   // p1 is pointer, p2 is just int!
int *p1, *p2;  // Both are pointers
```

---

## Summary

| Concept | Key Points |
|---------|------------|
| **Pointer** | Variable holding memory address |
| **& operator** | Gets address of variable |
| **\* operator** | Gets value at address |
| **Call by Value** | Copies data, original unchanged |
| **Call by Address** | Passes address, original can change |
| **Pointer Arithmetic** | Scales by datatype size |
| **Array = Pointer** | Array name is constant pointer to first element |
| **Double Pointer** | Pointer to pointer (`**`) |
| **DMA** | malloc/calloc/realloc/free |

---

> **Next Topic:** [03 - Storage Classes](./03_Storage_Classes.md)
