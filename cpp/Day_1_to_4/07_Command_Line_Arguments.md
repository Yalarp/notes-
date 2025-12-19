# 07 - Command Line Arguments

## Table of Contents
1. [Introduction](#introduction)
2. [argc and argv](#argc-and-argv)
3. [Accessing Arguments](#accessing-arguments)
4. [Practical Examples](#practical-examples)
5. [Setting Arguments in Visual Studio](#setting-arguments-in-visual-studio)
6. [Practice Questions](#practice-questions)
7. [Interview Questions](#interview-questions)

---

## Introduction

### What are Command Line Arguments?

**Command Line Arguments** are values passed to a program when it is executed from the command line or terminal.

```bash
# Example command:
myprogram.exe arg1 arg2 arg3

# The program receives:
# argv[0] = "myprogram.exe"
# argv[1] = "arg1"
# argv[2] = "arg2"
# argv[3] = "arg3"
# argc = 4
```

### Why Use Command Line Arguments?

| Use Case | Example |
|----------|---------|
| Configuration | `myapp --config=settings.ini` |
| File processing | `convert input.txt output.pdf` |
| Mode selection | `app --debug` |
| Batch operations | `process file1.txt file2.txt file3.txt` |

---

## argc and argv

### Definition

| Parameter | Type | Description |
|-----------|------|-------------|
| `argc` | `int` | **Argument Count** - Number of arguments (including program name) |
| `argv` | `char *[]` | **Argument Vector** - Array of strings containing arguments |

### main() Signature

```cpp
int main(int argc, char *argv[])
{
    // argc = number of arguments
    // argv = array of argument strings
}

// Alternative syntax (equivalent)
int main(int argc, char **argv)
{
    // Same as above
}
```

### Visual Representation

```
Command: myprogram.exe abc xyz lmn

┌─────────────────────────────────────────────────────────────────┐
│                    COMMAND LINE ARGUMENTS                       │
└─────────────────────────────────────────────────────────────────┘

    argc = 4 (total count of arguments)

    argv:
    ┌───────────────────────────────────────────────────────┐
    │ argv[0]: "myprogram.exe"  ← Program name (always!)   │
    │ argv[1]: "abc"            ← First argument           │
    │ argv[2]: "xyz"            ← Second argument          │
    │ argv[3]: "lmn"            ← Third argument           │
    │ argv[4]: NULL             ← Terminator               │
    └───────────────────────────────────────────────────────┘
```

> **Important:** `argv[0]` is ALWAYS the program name/path, not the first user argument!

---

## Accessing Arguments

### Basic Example

```cpp
#include <iostream>
using namespace std;

int main(int argc, char *argv[]) {
    // Check if enough arguments provided
    if(argc < 2) {
        cout << "Not enough arguments" << endl;
        return 0;
    }
    
    // Display all arguments
    for(int i = 0; i < argc; i++) {
        cout << argv[i] << endl;
    }
    
    return 0;
}
```

### Line-by-Line Explanation

| Line | Code | Explanation |
|------|------|-------------|
| `int main(int argc, char *argv[])` | main receives argument count and array |
| `if(argc < 2)` | Check if user provided at least 1 argument (argc<2 means only program name) |
| `cout << argv[i]` | Print each argument string |

### Execution Example

**Command:** `myprogram.exe abc xyz lmn`

**Output:**
```
D:\temp\myprogram.exe
abc
xyz
lmn
```

### Execution Flow

```
┌───────────────────────────────────────────────────────────────┐
│ User executes: myprogram.exe abc xyz lmn                      │
├───────────────────────────────────────────────────────────────┤
│ 1. OS parses command line into separate tokens               │
│                                                               │
│    Token 0: "D:\...\myprogram.exe"                           │
│    Token 1: "abc"                                            │
│    Token 2: "xyz"                                            │
│    Token 3: "lmn"                                            │
├───────────────────────────────────────────────────────────────┤
│ 2. OS creates argv array with 4 elements                     │
│    argc = 4                                                  │
├───────────────────────────────────────────────────────────────┤
│ 3. main(4, argv) is called                                   │
├───────────────────────────────────────────────────────────────┤
│ 4. Loop prints all 4 arguments                               │
└───────────────────────────────────────────────────────────────┘
```

---

## Practical Examples

### Example 1: Sum of Numbers

```cpp
#include <iostream>
#include <cstdlib>  // For atoi
using namespace std;

int main(int argc, char *argv[]) {
    if(argc < 3) {
        cout << "Usage: sum num1 num2" << endl;
        return 1;
    }
    
    int a = atoi(argv[1]);    // Convert string to int
    int b = atoi(argv[2]);    // Convert string to int
    
    cout << "Sum: " << (a + b) << endl;
    
    return 0;
}
```

**Usage:**
```bash
sum.exe 10 20
# Output: Sum: 30
```

### Example 2: File Name Processing

```cpp
#include <iostream>
#include <fstream>
using namespace std;

int main(int argc, char *argv[]) {
    if(argc != 2) {
        cout << "Usage: reader filename" << endl;
        return 1;
    }
    
    ifstream file(argv[1]);
    
    if(!file) {
        cout << "Cannot open file: " << argv[1] << endl;
        return 1;
    }
    
    string line;
    while(getline(file, line)) {
        cout << line << endl;
    }
    
    file.close();
    return 0;
}
```

**Usage:**
```bash
reader.exe data.txt
# Output: (contents of data.txt)
```

---

## Setting Arguments in Visual Studio

### Steps

```
1. Right-click on your project in Solution Explorer
2. Select "Properties"
3. Navigate to:
   Configuration Properties → Debugging
4. Find "Command Arguments"
5. Enter your arguments:
   arg1 arg2 arg3
6. Click OK

Now when you run/debug, these arguments will be passed.
```

### Visual Guide

```
┌─────────────────────────────────────────────────────────────────┐
│  Project Properties → Debugging                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Command:              $(TargetPath)                            │
│  Command Arguments:    abc xyz lmn                              │
│  Working Directory:    $(ProjectDir)                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Practice Questions

### Question 1: Program that displays each argument on a new line

```cpp
#include <iostream>
using namespace std;

int main(int argc, char *argv[]) {
    cout << "Total arguments: " << argc << endl;
    cout << "---" << endl;
    
    for(int i = 0; i < argc; i++) {
        cout << "Argument " << i << ": " << argv[i] << endl;
    }
    
    return 0;
}
```

### Question 2: Multiply all number arguments

```cpp
#include <iostream>
#include <cstdlib>
using namespace std;

int main(int argc, char *argv[]) {
    if(argc < 2) {
        cout << "Provide numbers to multiply" << endl;
        return 1;
    }
    
    int result = 1;
    for(int i = 1; i < argc; i++) {
        result *= atoi(argv[i]);
    }
    
    cout << "Product: " << result << endl;
    return 0;
}
```

---

## Interview Questions

### Q1: What is argv[0]?

**Answer:** `argv[0]` contains the program name or the full path to the executable. It is always present, so `argc` is always at least 1.

### Q2: What is the type of argv?

**Answer:** `argv` is an array of character pointers (`char *argv[]` or equivalently `char **argv`). Each element is a null-terminated string.

### Q3: How do you convert command line arguments to integers?

**Answer:**
```cpp
int value = atoi(argv[1]);        // C-style
int value = stoi(string(argv[1])); // C++11 style
```

### Q4: What happens if the user provides no arguments?

**Answer:** `argc` will be 1 (only the program name), and `argv` will contain only `argv[0]`.

---

## Summary

| Concept | Key Points |
|---------|------------|
| **argc** | Count of arguments, always >= 1 |
| **argv** | Array of string pointers |
| **argv[0]** | Always the program name |
| **atoi()** | Convert string to integer |
| **Validation** | Always check argc before accessing argv |

---

> **Next Topic:** [08 - Pattern Programs](./08_Pattern_Programs.md)
