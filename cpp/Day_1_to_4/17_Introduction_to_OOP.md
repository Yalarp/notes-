# 17 - Introduction to Object-Oriented Programming

## Table of Contents
1. [What is OOP](#what-is-oop)
2. [Four Pillars of OOP](#four-pillars-of-oop)
3. [Abstraction](#abstraction)
4. [Encapsulation](#encapsulation)
5. [Inheritance](#inheritance)
6. [Polymorphism](#polymorphism)
7. [Thinking in OOP](#thinking-in-oop)
8. [Summary](#summary)

---

## What is OOP

### Definition

**Object-Oriented Programming (OOP)** is a programming paradigm based on the concept of **objects** that contain:
- **Data** (attributes/properties)
- **Code** (methods/functions)

### OOP vs Procedural Programming

| Procedural | OOP |
|------------|-----|
| Focus on functions | Focus on objects |
| Data separate from functions | Data and functions bundled |
| Top-down approach | Bottom-up approach |
| Less modular | Highly modular |

---

## Four Pillars of OOP

```
┌─────────────────────────────────────────────────────────────────┐
│                   FOUR PILLARS OF OOP                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│    ┌────────────┐    ┌────────────────┐                        │
│    │ ABSTRACTION│    │ ENCAPSULATION  │                        │
│    │            │    │                │                        │
│    │ Hide       │    │ Bundle data    │                        │
│    │ complexity │    │ and methods    │                        │
│    └────────────┘    └────────────────┘                        │
│                                                                 │
│    ┌────────────┐    ┌────────────────┐                        │
│    │INHERITANCE │    │ POLYMORPHISM   │                        │
│    │            │    │                │                        │
│    │ Reuse code │    │ Many forms     │                        │
│    │ via parent │    │ of behavior    │                        │
│    └────────────┘    └────────────────┘                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Abstraction

### Definition

**Abstraction** is simplifying a problem by omitting unnecessary details, focusing only on relevant aspects.

### Also Called: Model Building

### Example: Car Brakes

```
User Perspective:      Internal Details (Hidden):
─────────────────      ────────────────────────────
Press brake pedal  →   Hydraulic fluid
                  →   Brake pads
                  →   Rotor friction
                  →   Anti-lock system
```

**User only needs to know:** Pressing pedal stops the car.
**User doesn't need to know:** The internal mechanics.

---

## Encapsulation

### Definition

**Encapsulation** binds together data and functions that manipulate that data, keeping both safe from outside interference.

### Components

| Component | Description |
|-----------|-------------|
| **Contractual Interface** | Public methods (what you CAN do) |
| **Implementation** | Private details (HOW it's done) |

### Example: PC Encapsulation

```
Exposed (Interface):        Hidden (Implementation):
────────────────────        ────────────────────────
• Power button              • Motherboard
• USB ports                 • Graphics card
• LED indicators            • Sound card
• Monitor connection        • Internal wiring
```

### Data Encapsulation = Data Hiding

```cpp
class BankAccount {
private:
    double balance;        // Hidden from outside
    
public:
    void deposit(double amount) {  // Controlled access
        if(amount > 0) {
            balance += amount;
        }
    }
    
    double getBalance() {
        return balance;
    }
};
```

---

## Inheritance

### Definition

**Inheritance** allows a new class (child) to acquire properties and behaviors from an existing class (parent).

### Terminology

| Term | Description |
|------|-------------|
| **Base/Parent class** | Class being inherited from |
| **Derived/Child class** | Class that inherits |
| **IS-A relationship** | Dog IS-A Animal |

### Example

```cpp
class Animal {
public:
    void eat() { cout << "Eating" << endl; }
};

class Dog : public Animal {   // Dog inherits from Animal
public:
    void bark() { cout << "Barking" << endl; }
};

// Dog can eat() (inherited) and bark() (own method)
```

---

## Polymorphism

### Definition

**Polymorphism** means "many forms" - the ability to take different forms.

### Types

| Type | Description | Example |
|------|-------------|---------|
| **Compile-time** | Function/Operator overloading | `add(int)` vs `add(double)` |
| **Runtime** | Virtual functions | Base pointer calling derived method |

---

## Thinking in OOP

### Step-by-Step Approach

1. **Identify the objects** in your problem domain
2. **Define properties** (data) for each object
3. **Define behaviors** (methods) for each object
4. **Establish relationships** between objects
5. **Implement** using classes

### Example: Library System

```
Objects:           Properties:              Behaviors:
───────────        ────────────             ───────────
Book          →    title, author, ISBN  →   borrow(), return()
Member        →    name, memberID       →   borrowBook(), returnBook()
Librarian     →    name, employeeID     →   addBook(), removeBook()
```

---

## Summary

| Pillar | Key Concept | Benefit |
|--------|-------------|---------|
| **Abstraction** | Hide complexity | Simplify usage |
| **Encapsulation** | Bundle data + methods | Data protection |
| **Inheritance** | Reuse parent class | Code reusability |
| **Polymorphism** | Many forms | Flexibility |

---

> **Next Topic:** [18 - Classes and Objects](./18_Classes_and_Objects.md)
