# 14 - Inline Functions

## Table of Contents
1. [Introduction](#introduction)
2. [When to Use Inline](#when-to-use-inline)
3. [Inline vs Macros](#inline-vs-macros)
4. [Limitations](#limitations)
5. [Summary](#summary)

---

## Introduction

### What is an Inline Function?

An **inline function** is a function where the compiler replaces the function call with the actual function code (inlining).

### Syntax

```cpp
inline returntype functionname(parameters) {
    // function body
}
```

### Example

```cpp
inline int square(int x) {
    return x * x;
}

int main() {
    int result = square(5);  // Compiler may replace with: int result = 5 * 5;
}
```

---

## When to Use Inline

### Good Candidates

| Type | Example |
|------|---------|
| Small functions | Getters, setters |
| Frequently called | Loop body helpers |
| Simple operations | Mathematical operations |

### Bad Candidates

| Type | Reason |
|------|--------|
| Large functions | Code bloat |
| Recursive functions | Can't be inlined |
| Functions with loops | Too complex |

---

## Inline vs Macros

| Feature | Inline | Macro |
|---------|--------|-------|
| Type checking | ✅ Yes | ❌ No |
| Debugging | ✅ Possible | ❌ Difficult |
| Side effects | ✅ Safe | ❌ Risky |
| Syntax | Normal function | Preprocessor |

### Macro Problem

```cpp
#define SQR(x) x*x
cout << SQR(2+3);  // Expands to: 2+3*2+3 = 11 (wrong!)

inline int sqr(int x) { return x*x; }
cout << sqr(2+3);  // Correctly computes: 5*5 = 25
```

---

## Limitations

1. **Request, not command** - Compiler may ignore `inline`
2. **Code bloat** - Large inlined functions increase code size
3. **Recursive functions** - Cannot be inlined
4. **Virtual functions** - May not be inlined

---

## Summary

| Concept | Key Point |
|---------|-----------|
| **Inline** | Compiler replaces call with function body |
| **Benefit** | Faster (no function call overhead) |
| **Drawback** | Code bloat if overused |
| **Best for** | Small, frequently called functions |

---

> **Next Topic:** [15 - Const Pointers](./15_Const_Pointers_and_Pointer_to_Const.md)
