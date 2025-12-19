# 📦 Collections in C# - Complete Guide

## 📚 Table of Contents
- [Overview](#-overview)
- [Array vs Collections](#-array-vs-collections)
- [List<T>](#-listt)
- [Dictionary<TKey, TValue>](#-dictionarytkey-tvalue)
- [Queue<T> and Stack<T>](#-queuet-and-stackt)
- [HashSet<T>](#-hashsett)
- [Performance Comparison](#-performance-comparison)
- [Interview Questions](#-interview-questions)

---

## 🎯 Overview

Collections are dynamic data structures that can grow/shrink at runtime, unlike fixed-size arrays.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    COLLECTION TYPES                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  List<T>       - Dynamic array, indexed access                      │
│  Dictionary    - Key-value pairs, fast lookup                       │
│  Queue<T>      - FIFO (First In, First Out)                        │
│  Stack<T>      - LIFO (Last In, First Out)                         │
│  HashSet<T>    - Unique elements, fast contains check              │
│  LinkedList<T> - Doubly-linked list, fast insert/remove            │
│  SortedList    - Sorted key-value pairs                            │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔷 Array vs Collections

| Feature | Array | List<T> |
|---------|-------|---------|
| Size | Fixed at creation | Can grow/shrink |
| Syntax | `int[] arr = new int[5]` | `List<int> list = new()` |
| Add/Remove | Not possible | `Add()`, `Remove()` |
| Performance | Slightly faster | Slightly slower |
| Memory | Contiguous | Contiguous (internally) |

---

## 🔶 List<T>

The most commonly used collection - a dynamic array.

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// LIST<T> - DYNAMIC ARRAY
// ═══════════════════════════════════════════════════════════════════════════

using System;
using System.Collections.Generic;

class ListDemo
{
    static void Main()
    {
        // ─────────────────────────────────────────────────────────────────
        // CREATING LISTS
        // ─────────────────────────────────────────────────────────────────
        
        // Empty list
        List<int> numbers = new List<int>();
        
        // With initial capacity (performance optimization)
        List<int> bigList = new List<int>(1000);
        // Line 1: Pre-allocates space for 1000 items
        //         Reduces reallocations as list grows
        
        // Collection initializer
        List<string> fruits = new List<string> { "Apple", "Banana", "Cherry" };
        // Line 2: Initialize with values
        
        // From array
        int[] arr = { 1, 2, 3 };
        List<int> fromArray = new List<int>(arr);
        
        // ─────────────────────────────────────────────────────────────────
        // ADDING ELEMENTS
        // ─────────────────────────────────────────────────────────────────
        
        List<string> names = new List<string>();
        
        names.Add("Alice");
        // Line 3: Add single item to end
        
        names.AddRange(new[] { "Bob", "Charlie" });
        // Line 4: Add multiple items to end
        
        names.Insert(1, "Diana");
        // Line 5: Insert at specific index
        //         Shifts existing elements right
        // List: Alice, Diana, Bob, Charlie
        
        // ─────────────────────────────────────────────────────────────────
        // ACCESSING ELEMENTS
        // ─────────────────────────────────────────────────────────────────
        
        string first = names[0];
        // Line 6: Access by index (0-based)
        
        string last = names[^1];
        // Line 7: C# 8 index from end (^1 = last element)
        
        List<string> subset = names.GetRange(1, 2);
        // Line 8: Get range: start at index 1, take 2 items
        
        // ─────────────────────────────────────────────────────────────────
        // SEARCHING
        // ─────────────────────────────────────────────────────────────────
        
        bool hasAlice = names.Contains("Alice");
        // Line 9: Check if item exists
        
        int index = names.IndexOf("Bob");
        // Line 10: Get index of first occurrence (-1 if not found)
        
        int lastIndex = names.LastIndexOf("Bob");
        // Get index of last occurrence
        
        string found = names.Find(n => n.StartsWith("D"));
        // Line 11: Find first matching item using predicate
        // found = "Diana"
        
        List<string> allMatches = names.FindAll(n => n.Length > 4);
        // Line 12: Find all matching items
        
        bool exists = names.Exists(n => n == "Charlie");
        // Line 13: Check if any match predicate
        
        // ─────────────────────────────────────────────────────────────────
        // REMOVING ELEMENTS
        // ─────────────────────────────────────────────────────────────────
        
        names.Remove("Diana");
        // Line 14: Remove first occurrence of value
        
        names.RemoveAt(0);
        // Line 15: Remove at specific index
        
        names.RemoveRange(0, 2);
        // Remove range: start at 0, remove 2 items
        
        names.RemoveAll(n => n.Length < 4);
        // Line 16: Remove all matching predicate
        
        names.Clear();
        // Line 17: Remove all elements
        
        // ─────────────────────────────────────────────────────────────────
        // SORTING AND REVERSING
        // ─────────────────────────────────────────────────────────────────
        
        List<int> nums = new List<int> { 5, 2, 8, 1, 9 };
        
        nums.Sort();
        // Line 18: Sort ascending: 1, 2, 5, 8, 9
        
        nums.Sort((a, b) => b.CompareTo(a));
        // Line 19: Custom comparison - descending: 9, 8, 5, 2, 1
        
        nums.Reverse();
        // Line 20: Reverse order in place
        
        // ─────────────────────────────────────────────────────────────────
        // CONVERSION
        // ─────────────────────────────────────────────────────────────────
        
        int[] array = nums.ToArray();
        // Line 21: Convert to array
        
        List<string> stringNums = nums.ConvertAll(n => n.ToString());
        // Line 22: Transform each element
        
        // ─────────────────────────────────────────────────────────────────
        // PROPERTIES
        // ─────────────────────────────────────────────────────────────────
        
        int count = nums.Count;
        // Line 23: Number of elements
        
        int capacity = nums.Capacity;
        // Line 24: Current internal array size
        
        nums.TrimExcess();
        // Line 25: Reduce capacity to match count
    }
}
```

---

## 🔵 Dictionary<TKey, TValue>

Key-value pair collection with fast O(1) lookup.

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// DICTIONARY<TKEY, TVALUE> - KEY-VALUE PAIRS
// ═══════════════════════════════════════════════════════════════════════════

using System;
using System.Collections.Generic;

class DictionaryDemo
{
    static void Main()
    {
        // ─────────────────────────────────────────────────────────────────
        // CREATING DICTIONARIES
        // ─────────────────────────────────────────────────────────────────
        
        Dictionary<string, int> ages = new Dictionary<string, int>();
        
        // Collection initializer
        Dictionary<string, string> capitals = new Dictionary<string, string>
        {
            { "USA", "Washington" },
            { "UK", "London" },
            { "France", "Paris" }
        };
        // Line 1: Initialize with key-value pairs
        
        // C# 6+ simplified syntax
        var scores = new Dictionary<string, int>
        {
            ["Alice"] = 95,
            ["Bob"] = 87,
            ["Charlie"] = 92
        };
        // Line 2: Index initializer syntax
        
        // ─────────────────────────────────────────────────────────────────
        // ADDING AND UPDATING
        // ─────────────────────────────────────────────────────────────────
        
        ages["Alice"] = 25;
        // Line 3: Add or update using indexer
        //         If key exists: updates value
        //         If key doesn't exist: adds new entry
        
        ages.Add("Bob", 30);
        // Line 4: Add new entry
        //         Throws exception if key already exists!
        
        bool added = ages.TryAdd("Charlie", 28);
        // Line 5: Add if key doesn't exist
        //         Returns false if key exists (no exception)
        
        // ─────────────────────────────────────────────────────────────────
        // ACCESSING VALUES
        // ─────────────────────────────────────────────────────────────────
        
        int aliceAge = ages["Alice"];
        // Line 6: Get value by key
        //         Throws KeyNotFoundException if not found!
        
        // Safe access
        if (ages.TryGetValue("David", out int davidAge))
        // Line 7: TryGetValue - safe retrieval
        //         Returns true if found, value in out parameter
        //         Returns false if not found
        {
            Console.WriteLine($"David is {davidAge}");
        }
        else
        {
            Console.WriteLine("David not found");
        }
        
        // GetValueOrDefault (C# 7.1+)
        int eveAge = ages.GetValueOrDefault("Eve", 0);
        // Line 8: Returns 0 if key not found
        
        // ─────────────────────────────────────────────────────────────────
        // CHECKING EXISTENCE
        // ─────────────────────────────────────────────────────────────────
        
        bool hasAlice = ages.ContainsKey("Alice");
        // Line 9: Check if key exists
        
        bool has25 = ages.ContainsValue(25);
        // Line 10: Check if value exists (slower - O(n))
        
        // ─────────────────────────────────────────────────────────────────
        // REMOVING
        // ─────────────────────────────────────────────────────────────────
        
        ages.Remove("Bob");
        // Line 11: Remove by key
        
        bool removed = ages.Remove("NonExistent");
        // Returns false if key didn't exist
        
        ages.Clear();
        // Remove all entries
        
        // ─────────────────────────────────────────────────────────────────
        // ITERATING
        // ─────────────────────────────────────────────────────────────────
        
        // Iterate key-value pairs
        foreach (KeyValuePair<string, int> kvp in scores)
        {
            Console.WriteLine($"{kvp.Key}: {kvp.Value}");
        }
        
        // Deconstruct (C# 7+)
        foreach (var (name, score) in scores)
        // Line 12: Deconstruct KeyValuePair
        {
            Console.WriteLine($"{name}: {score}");
        }
        
        // Just keys
        foreach (string name in scores.Keys)
        {
            Console.WriteLine(name);
        }
        
        // Just values
        foreach (int score in scores.Values)
        {
            Console.WriteLine(score);
        }
        
        // ─────────────────────────────────────────────────────────────────
        // PROPERTIES
        // ─────────────────────────────────────────────────────────────────
        
        int count = scores.Count;
        ICollection<string> keys = scores.Keys;
        ICollection<int> values = scores.Values;
    }
}
```

---

## 🟢 Queue<T> and Stack<T>

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// QUEUE<T> - FIFO (FIRST IN, FIRST OUT)
// ═══════════════════════════════════════════════════════════════════════════

using System;
using System.Collections.Generic;

class QueueDemo
{
    static void Main()
    {
        Queue<string> line = new Queue<string>();
        
        // Add to queue (end)
        line.Enqueue("First");
        line.Enqueue("Second");
        line.Enqueue("Third");
        // Line 1: Enqueue adds to back
        // Queue: First -> Second -> Third
        
        // Remove from queue (front)
        string served = line.Dequeue();
        // Line 2: Dequeue removes from front
        // served = "First"
        // Queue: Second -> Third
        
        // Peek at front without removing
        string next = line.Peek();
        // Line 3: Peek returns front item without removing
        // next = "Second"
        
        // Safe operations
        if (line.TryDequeue(out string item))
        // Line 4: TryDequeue - safe removal
        {
            Console.WriteLine($"Processed: {item}");
        }
        
        if (line.TryPeek(out string frontItem))
        {
            Console.WriteLine($"Next: {frontItem}");
        }
        
        int count = line.Count;
        bool hasItems = line.Count > 0;
        line.Clear();
    }
}

// ═══════════════════════════════════════════════════════════════════════════
// STACK<T> - LIFO (LAST IN, FIRST OUT)
// ═══════════════════════════════════════════════════════════════════════════

class StackDemo
{
    static void Main()
    {
        Stack<string> history = new Stack<string>();
        
        // Add to stack (top)
        history.Push("Page1");
        history.Push("Page2");
        history.Push("Page3");
        // Line 5: Push adds to top
        // Stack: Page3 (top) -> Page2 -> Page1 (bottom)
        
        // Remove from stack (top)
        string current = history.Pop();
        // Line 6: Pop removes from top
        // current = "Page3"
        // Stack: Page2 (top) -> Page1 (bottom)
        
        // Peek at top without removing
        string top = history.Peek();
        // Line 7: Peek returns top item
        // top = "Page2"
        
        // Safe operations
        if (history.TryPop(out string page))
        {
            Console.WriteLine($"Back to: {page}");
        }
        
        // Use case: Undo functionality
        Stack<string> undoStack = new Stack<string>();
        undoStack.Push("Action1");
        undoStack.Push("Action2");
        // To undo: undoStack.Pop()
    }
}
```

---

## 🟣 HashSet<T>

Collection of unique elements with O(1) contains check.

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// HASHSET<T> - UNIQUE ELEMENTS
// ═══════════════════════════════════════════════════════════════════════════

using System;
using System.Collections.Generic;

class HashSetDemo
{
    static void Main()
    {
        HashSet<int> numbers = new HashSet<int>();
        
        // ─────────────────────────────────────────────────────────────────
        // ADDING ELEMENTS
        // ─────────────────────────────────────────────────────────────────
        
        numbers.Add(1);
        numbers.Add(2);
        numbers.Add(3);
        bool added = numbers.Add(2);  // Returns false - already exists!
        // Line 1: Add returns false if item already exists
        //         No duplicates allowed
        
        // ─────────────────────────────────────────────────────────────────
        // CHECKING EXISTENCE
        // ─────────────────────────────────────────────────────────────────
        
        bool has2 = numbers.Contains(2);
        // Line 2: O(1) lookup - very fast!
        
        // ─────────────────────────────────────────────────────────────────
        // SET OPERATIONS
        // ─────────────────────────────────────────────────────────────────
        
        HashSet<int> setA = new HashSet<int> { 1, 2, 3, 4, 5 };
        HashSet<int> setB = new HashSet<int> { 4, 5, 6, 7, 8 };
        
        // Union - all elements from both sets
        HashSet<int> union = new HashSet<int>(setA);
        union.UnionWith(setB);
        // Line 3: {1, 2, 3, 4, 5, 6, 7, 8}
        
        // Intersection - common elements
        HashSet<int> intersection = new HashSet<int>(setA);
        intersection.IntersectWith(setB);
        // Line 4: {4, 5}
        
        // Difference - in A but not in B
        HashSet<int> difference = new HashSet<int>(setA);
        difference.ExceptWith(setB);
        // Line 5: {1, 2, 3}
        
        // Symmetric difference - in A or B but not both
        HashSet<int> symDiff = new HashSet<int>(setA);
        symDiff.SymmetricExceptWith(setB);
        // Line 6: {1, 2, 3, 6, 7, 8}
        
        // ─────────────────────────────────────────────────────────────────
        // SET COMPARISONS
        // ─────────────────────────────────────────────────────────────────
        
        HashSet<int> subset = new HashSet<int> { 1, 2 };
        
        bool isSubset = subset.IsSubsetOf(setA);
        // Line 7: True - all elements of subset are in setA
        
        bool isSuperset = setA.IsSupersetOf(subset);
        // True - setA contains all elements of subset
        
        bool overlaps = setA.Overlaps(setB);
        // True - sets share at least one element
        
        bool equals = setA.SetEquals(new HashSet<int> { 5, 4, 3, 2, 1 });
        // True - same elements (order doesn't matter)
    }
}
```

---

## 📊 Performance Comparison

| Operation | List | Dictionary | HashSet | Queue | Stack |
|-----------|------|------------|---------|-------|-------|
| **Add** | O(1)* | O(1) | O(1) | O(1) | O(1) |
| **Remove** | O(n) | O(1) | O(1) | O(1) | O(1) |
| **Access by Index** | O(1) | N/A | N/A | N/A | N/A |
| **Access by Key** | N/A | O(1) | N/A | N/A | N/A |
| **Contains** | O(n) | O(1) | O(1) | O(n) | O(n) |
| **Search** | O(n) | O(1) | O(1) | O(n) | O(n) |

*O(1) amortized, O(n) when resizing

---

## 🎤 Interview Questions

**Q1: When would you use List vs Dictionary?**
> Use List for ordered collections with index-based access. Use Dictionary when you need fast key-based lookup.

**Q2: What's the difference between Queue and Stack?**
> Queue is FIFO (First-In-First-Out) - like a line. Stack is LIFO (Last-In-First-Out) - like a stack of plates.

**Q3: Why use HashSet?**
> For unique elements with O(1) contains check and set operations (union, intersection, etc.).

**Q4: What happens if you add duplicate key to Dictionary?**
> `Add()` throws `ArgumentException`. Indexer `dict[key] = value` overwrites existing value.

**Q5: What is the time complexity of List.Contains vs HashSet.Contains?**
> List.Contains is O(n) - must check each element. HashSet.Contains is O(1) - uses hash-based lookup.

---

## 📝 Summary

| Collection | Use Case | Key Methods |
|------------|----------|-------------|
| **List<T>** | Dynamic array | Add, Remove, Insert, Find |
| **Dictionary** | Key-value lookup | Add, TryGetValue, ContainsKey |
| **Queue** | FIFO processing | Enqueue, Dequeue, Peek |
| **Stack** | LIFO/Undo | Push, Pop, Peek |
| **HashSet** | Unique items | Add, Contains, UnionWith |
