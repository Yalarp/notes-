# 29 - Smart Pointers

## Table of Contents
1. [Introduction](#introduction)
2. [unique_ptr](#unique_ptr)
3. [shared_ptr](#shared_ptr)
4. [weak_ptr](#weak_ptr)
5. [Comparison](#comparison)
6. [Summary](#summary)

---

## Introduction

### Problems with Raw Pointers

| Problem | Description |
|---------|-------------|
| Memory leaks | Forgetting to delete |
| Dangling pointers | Using after delete |
| Double delete | Deleting twice |
| Exception safety | Leak if exception thrown |

### Solution: Smart Pointers (C++11)

Smart pointers automatically manage memory using **RAII** (Resource Acquisition Is Initialization).

```cpp
#include <memory>   // Required header
```

---

## unique_ptr

### Characteristics

| Feature | Description |
|---------|-------------|
| Exclusive ownership | Only one owner |
| Cannot copy | Only move |
| Auto-delete | When goes out of scope |

### Usage

```cpp
#include <memory>
using namespace std;

int main() {
    // Create unique_ptr
    unique_ptr<int> ptr1 = make_unique<int>(10);
    
    cout << *ptr1 << endl;  // 10
    
    // unique_ptr<int> ptr2 = ptr1;  // ❌ Error! Cannot copy
    
    unique_ptr<int> ptr2 = move(ptr1);  // ✅ Move ownership
    // ptr1 is now nullptr
    
}   // ptr2 automatically deleted here
```

### For Arrays

```cpp
unique_ptr<int[]> arr = make_unique<int[]>(5);
arr[0] = 10;
arr[1] = 20;
// Automatically deleted when scope ends
```

---

## shared_ptr

### Characteristics

| Feature | Description |
|---------|-------------|
| Shared ownership | Multiple owners allowed |
| Reference counting | Tracks owner count |
| Auto-delete | When last owner destroyed |

### Usage

```cpp
#include <memory>
using namespace std;

int main() {
    shared_ptr<int> ptr1 = make_shared<int>(10);
    cout << ptr1.use_count() << endl;  // 1
    
    {
        shared_ptr<int> ptr2 = ptr1;   // Share ownership
        cout << ptr1.use_count() << endl;  // 2
    }   // ptr2 destroyed, count = 1
    
    cout << ptr1.use_count() << endl;  // 1
}   // ptr1 destroyed, memory freed
```

### Memory Visualization

```
ptr1 ──┐
       ├──▶ [Control Block] ──▶ [Data: 10]
ptr2 ──┘     (count = 2)
```

---

## weak_ptr

### Characteristics

| Feature | Description |
|---------|-------------|
| Non-owning reference | Doesn't increase count |
| Prevents cycles | Breaks circular references |
| Must lock to use | Convert to shared_ptr first |

### Usage

```cpp
#include <memory>
using namespace std;

int main() {
    shared_ptr<int> shared = make_shared<int>(10);
    weak_ptr<int> weak = shared;   // Doesn't increase count
    
    cout << shared.use_count() << endl;  // 1 (not 2!)
    
    // To use weak_ptr, lock it first
    if(auto locked = weak.lock()) {
        cout << *locked << endl;  // 10
    }
    
    shared.reset();   // Memory freed
    
    if(weak.expired()) {
        cout << "Pointer expired!" << endl;
    }
}
```

### Use Case: Breaking Circular References

```cpp
class Node {
public:
    shared_ptr<Node> next;
    weak_ptr<Node> prev;   // weak to prevent cycle!
};
```

---

## Comparison

| Feature | unique_ptr | shared_ptr | weak_ptr |
|---------|------------|------------|----------|
| Ownership | Exclusive | Shared | None |
| Copy | ❌ | ✅ | ✅ |
| Move | ✅ | ✅ | ✅ |
| Overhead | Minimal | Reference count | Minimal |
| Use case | Single owner | Multiple owners | Observers |

---

## Summary

| Smart Pointer | When to Use |
|---------------|-------------|
| `unique_ptr` | Single ownership, best performance |
| `shared_ptr` | Multiple owners need same resource |
| `weak_ptr` | Observe without owning, break cycles |

```cpp
// Modern C++ style - prefer smart pointers
unique_ptr<Widget> w = make_unique<Widget>();
shared_ptr<Data> d = make_shared<Data>();
```

---

> **Next Topic:** [30 - Type Conversion](./30_Type_Conversion.md)
