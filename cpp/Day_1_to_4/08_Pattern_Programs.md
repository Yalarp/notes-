# 08 - Pattern Programs

## Table of Contents
1. [Introduction](#introduction)
2. [Basic Patterns](#basic-patterns)
3. [Number Patterns](#number-patterns)
4. [Advanced Patterns](#advanced-patterns)
5. [Logic Explanation](#logic-explanation)
6. [Practice Questions](#practice-questions)

---

## Introduction

### What are Pattern Programs?

Pattern programs are exercises that print shapes using characters, numbers, or symbols. They help understand:

- Nested loops
- Loop control variables
- Printing logic
- Space and character placement

### General Approach

```cpp
for(int i = 1; i <= rows; i++) {      // Outer loop: controls rows
    for(int j = 1; j <= columns; j++) { // Inner loop: controls columns
        cout << character;              // Print character/number
    }
    cout << endl;                       // Move to next line
}
```

---

## Basic Patterns

### Pattern 1: Right Triangle

```
*
**
***
****
*****
```

```cpp
#include <iostream>
using namespace std;

int main() {
    int n = 5;
    
    for(int i = 1; i <= n; i++) {       // Line 1: i goes from 1 to 5
        for(int j = 1; j <= i; j++) {   // Line 2: j goes from 1 to current i
            cout << "*";                 // Line 3: Print star
        }
        cout << endl;                    // Line 4: Move to next line
    }
    
    return 0;
}
```

**Execution Flow:**
```
i=1: j goes 1 to 1 → prints 1 star  → *
i=2: j goes 1 to 2 → prints 2 stars → **
i=3: j goes 1 to 3 → prints 3 stars → ***
i=4: j goes 1 to 4 → prints 4 stars → ****
i=5: j goes 1 to 5 → prints 5 stars → *****
```

---

### Pattern 2: Inverted Right Triangle

```
*****
****
***
**
*
```

```cpp
int n = 5;
for(int i = n; i >= 1; i--) {       // i goes from 5 down to 1
    for(int j = 1; j <= i; j++) {   // j goes from 1 to current i
        cout << "*";
    }
    cout << endl;
}
```

---

### Pattern 3: Square

```
*****
*****
*****
*****
*****
```

```cpp
int n = 5;
for(int i = 1; i <= n; i++) {
    for(int j = 1; j <= n; j++) {   // Always print n stars
        cout << "*";
    }
    cout << endl;
}
```

---

### Pattern 4: Hollow Square

```
*****
*   *
*   *
*   *
*****
```

```cpp
int n = 5;
for(int i = 1; i <= n; i++) {
    for(int j = 1; j <= n; j++) {
        if(i == 1 || i == n || j == 1 || j == n) {
            cout << "*";
        } else {
            cout << " ";
        }
    }
    cout << endl;
}
```

---

## Number Patterns

### Pattern 5: Number Triangle

```
1
12
123
1234
12345
```

```cpp
int n = 5;
for(int i = 1; i <= n; i++) {
    for(int j = 1; j <= i; j++) {
        cout << j;                   // Print j (current column number)
    }
    cout << endl;
}
```

---

### Pattern 6: Same Number Triangle

```
1
22
333
4444
55555
```

```cpp
int n = 5;
for(int i = 1; i <= n; i++) {
    for(int j = 1; j <= i; j++) {
        cout << i;                   // Print i (row number)
    }
    cout << endl;
}
```

---

### Pattern 7: Floyd's Triangle

```
1
2 3
4 5 6
7 8 9 10
```

```cpp
int n = 4;
int num = 1;                         // Counter starts at 1

for(int i = 1; i <= n; i++) {
    for(int j = 1; j <= i; j++) {
        cout << num << " ";
        num++;                       // Increment for next print
    }
    cout << endl;
}
```

---

## Advanced Patterns

### Pattern 8: Pyramid

```
    *
   ***
  *****
 *******
*********
```

```cpp
int n = 5;
for(int i = 1; i <= n; i++) {
    // Print spaces (n-i spaces)
    for(int j = 1; j <= n - i; j++) {
        cout << " ";
    }
    // Print stars (2*i-1 stars)
    for(int j = 1; j <= 2 * i - 1; j++) {
        cout << "*";
    }
    cout << endl;
}
```

**Logic:**
```
Row 1: 4 spaces, 1 star   (n-1 spaces, 2*1-1 stars)
Row 2: 3 spaces, 3 stars  (n-2 spaces, 2*2-1 stars)
Row 3: 2 spaces, 5 stars  (n-3 spaces, 2*3-1 stars)
Row 4: 1 space,  7 stars  (n-4 spaces, 2*4-1 stars)
Row 5: 0 spaces, 9 stars  (n-5 spaces, 2*5-1 stars)
```

---

### Pattern 9: Inverted Pyramid

```
*********
 *******
  *****
   ***
    *
```

```cpp
int n = 5;
for(int i = n; i >= 1; i--) {
    // Print spaces
    for(int j = 1; j <= n - i; j++) {
        cout << " ";
    }
    // Print stars
    for(int j = 1; j <= 2 * i - 1; j++) {
        cout << "*";
    }
    cout << endl;
}
```

---

### Pattern 10: Diamond

```
    *
   ***
  *****
 *******
*********
 *******
  *****
   ***
    *
```

```cpp
int n = 5;

// Upper half (pyramid)
for(int i = 1; i <= n; i++) {
    for(int j = 1; j <= n - i; j++) cout << " ";
    for(int j = 1; j <= 2 * i - 1; j++) cout << "*";
    cout << endl;
}

// Lower half (inverted pyramid, starting from n-1)
for(int i = n - 1; i >= 1; i--) {
    for(int j = 1; j <= n - i; j++) cout << " ";
    for(int j = 1; j <= 2 * i - 1; j++) cout << "*";
    cout << endl;
}
```

---

### Pattern 11: Right Aligned Triangle

```
    *
   **
  ***
 ****
*****
```

```cpp
int n = 5;
for(int i = 1; i <= n; i++) {
    // Print spaces (n-i spaces)
    for(int j = 1; j <= n - i; j++) {
        cout << " ";
    }
    // Print stars (i stars)
    for(int j = 1; j <= i; j++) {
        cout << "*";
    }
    cout << endl;
}
```

---

## Logic Explanation

### Understanding the Loops

```cpp
for(int i = 1; i <= n; i++) {        // OUTER: Which row are we on?
    for(int j = 1; j <= ???; j++) {  // INNER: How many characters in this row?
        cout << "?";                  // WHAT: What to print?
    }
    cout << endl;                     // NEWLINE: Move to next row
}
```

### Key Questions

| Question | Determines |
|----------|------------|
| How many rows? | Outer loop limit |
| How many chars per row? | Inner loop limit |
| What to print? | cout content |
| Left alignment? | Spaces before stars |

### Common Formulas

| Pattern | Stars per row | Spaces per row |
|---------|---------------|----------------|
| Right triangle | i | 0 |
| Left triangle | i | n - i |
| Pyramid | 2i - 1 | n - i |
| Inverted | 2(n-i+1) - 1 | i - 1 |

---

## Practice Questions

### Question 1: Pascal's Triangle

```
    1
   1 1
  1 2 1
 1 3 3 1
1 4 6 4 1
```

```cpp
int n = 5;
for(int i = 0; i < n; i++) {
    // Print spaces
    for(int j = 0; j < n - i - 1; j++)
        cout << " ";
    
    // Print numbers
    int num = 1;
    for(int j = 0; j <= i; j++) {
        cout << num << " ";
        num = num * (i - j) / (j + 1);
    }
    cout << endl;
}
```

### Question 2: Butterfly Pattern

```
*        *
**      **
***    ***
****  ****
**********
****  ****
***    ***
**      **
*        *
```

**Hint:** Two halves - upper and lower, each with stars-spaces-stars logic.

---

## Summary

| Concept | Key Points |
|---------|------------|
| **Outer Loop** | Controls which row |
| **Inner Loop** | Controls what prints in that row |
| **Spaces** | Usually `n - i` for left alignment |
| **Stars** | Usually `i` for triangle, `2i-1` for pyramid |
| **Practice** | Key is identifying the pattern in each row |

---

> **Next Topic:** [09 - C++ Enhancements Over C](./09_Cpp_Enhancements_Over_C.md)
