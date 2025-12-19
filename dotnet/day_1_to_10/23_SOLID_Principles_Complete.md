# 🏛️ SOLID Principles Complete Guide

## 📚 Table of Contents
- [Overview](#-overview)
- [S - Single Responsibility](#-single-responsibility-principle)
- [O - Open/Closed](#-openclosed-principle)
- [L - Liskov Substitution](#-liskov-substitution-principle)
- [I - Interface Segregation](#-interface-segregation-principle)
- [D - Dependency Inversion](#-dependency-inversion-principle)
- [Interview Questions](#-interview-questions)

---

## 🎯 Overview

**SOLID** principles are five design principles for writing maintainable, scalable object-oriented code.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SOLID PRINCIPLES                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  S - Single Responsibility   One class = One reason to change     │
│  O - Open/Closed             Open for extension, closed for mod   │
│  L - Liskov Substitution     Subclasses replaceable for base      │
│  I - Interface Segregation   Many small interfaces > one big      │
│  D - Dependency Inversion    Depend on abstractions, not concr    │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔷 Single Responsibility Principle

> **"A class should have only one reason to change"**

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// ❌ BAD: Multiple responsibilities
// ═══════════════════════════════════════════════════════════════════════════

public class Invoice
{
    public decimal Amount { get; set; }
    
    // Responsibility 1: Invoice logic
    public void AddItem(Item item) { /* ... */ }
    
    // Responsibility 2: Database (WRONG!)
    public void SaveToDatabase() { /* ... */ }
    
    // Responsibility 3: Email (WRONG!)
    public void SendEmail() { /* ... */ }
    
    // Responsibility 4: Printing (WRONG!)
    public void Print() { /* ... */ }
}
// This class has 4 reasons to change!

// ═══════════════════════════════════════════════════════════════════════════
// ✅ GOOD: Separate responsibilities into classes
// ═══════════════════════════════════════════════════════════════════════════

// Responsibility 1: Invoice business logic only
public class Invoice
{
    public decimal Amount { get; set; }
    public List<Item> Items { get; set; }
    
    public void AddItem(Item item)
    {
        Items.Add(item);
        Amount += item.Price;
    }
}

// Responsibility 2: Database operations
public class InvoiceRepository
{
    public void Save(Invoice invoice) { /* DB logic */ }
    public Invoice Load(int id) { /* DB logic */ }
}

// Responsibility 3: Email
public class InvoiceEmailer
{
    public void SendInvoice(Invoice invoice, string email) { /* Email */ }
}

// Responsibility 4: Printing
public class InvoicePrinter
{
    public void Print(Invoice invoice) { /* Print */ }
}

// Each class has ONE reason to change
```

---

## 🔶 Open/Closed Principle

> **"Open for extension, closed for modification"**

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// ❌ BAD: Must modify class to add new discount types
// ═══════════════════════════════════════════════════════════════════════════

public class DiscountCalculator
{
    public decimal Calculate(decimal price, string discountType)
    {
        if (discountType == "Regular")
            return price * 0.1m;
        else if (discountType == "Premium")
            return price * 0.2m;
        else if (discountType == "VIP")
            return price * 0.3m;
        // Must modify this class for new discount types!
        return 0;
    }
}

// ═══════════════════════════════════════════════════════════════════════════
// ✅ GOOD: Extend via new classes, no modification needed
// ═══════════════════════════════════════════════════════════════════════════

// Abstract base (closed for modification)
public interface IDiscount
{
    decimal Calculate(decimal price);
}

// Extensions (open for extension)
public class RegularDiscount : IDiscount
{
    public decimal Calculate(decimal price) => price * 0.1m;
}

public class PremiumDiscount : IDiscount
{
    public decimal Calculate(decimal price) => price * 0.2m;
}

public class VIPDiscount : IDiscount
{
    public decimal Calculate(decimal price) => price * 0.3m;
}

// Adding new discount type = new class, no changes to existing code
public class BlackFridayDiscount : IDiscount
{
    public decimal Calculate(decimal price) => price * 0.5m;
}

// Usage
public class Order
{
    public decimal CalculateTotal(decimal price, IDiscount discount)
    {
        return price - discount.Calculate(price);
    }
}
```

---

## 🔵 Liskov Substitution Principle

> **"Subclasses should be substitutable for their base classes"**

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// ❌ BAD: Penguin breaks Bird behavior
// ═══════════════════════════════════════════════════════════════════════════

public class Bird
{
    public virtual void Fly() 
    { 
        Console.WriteLine("Flying!"); 
    }
}

public class Penguin : Bird
{
    public override void Fly()
    {
        throw new NotSupportedException("Penguins can't fly!");
        // Breaks substitution - code expecting Bird.Fly() will crash!
    }
}

void MakeBirdsFly(List<Bird> birds)
{
    foreach (var bird in birds)
        bird.Fly();  // Crashes on Penguin!
}

// ═══════════════════════════════════════════════════════════════════════════
// ✅ GOOD: Separate interfaces for different capabilities
// ═══════════════════════════════════════════════════════════════════════════

public interface IBird
{
    void Eat();
}

public interface IFlyingBird : IBird
{
    void Fly();
}

public class Sparrow : IFlyingBird
{
    public void Eat() => Console.WriteLine("Eating...");
    public void Fly() => Console.WriteLine("Flying!");
}

public class Penguin : IBird
{
    public void Eat() => Console.WriteLine("Eating fish...");
    // No Fly() method - Penguin doesn't pretend to fly
}

void MakeBirdsFly(List<IFlyingBird> birds)
{
    foreach (var bird in birds)
        bird.Fly();  // Only flying birds passed, no crashes
}
```

---

## 🟢 Interface Segregation Principle

> **"Clients should not be forced to depend on methods they don't use"**

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// ❌ BAD: Fat interface forces unused implementations
// ═══════════════════════════════════════════════════════════════════════════

public interface IWorker
{
    void Work();
    void Eat();
    void Sleep();
    void AttendMeeting();
    void WriteReport();
}

public class Robot : IWorker
{
    public void Work() => Console.WriteLine("Working...");
    
    // Robots don't eat or sleep!
    public void Eat() => throw new NotImplementedException();
    public void Sleep() => throw new NotImplementedException();
    public void AttendMeeting() => throw new NotImplementedException();
    public void WriteReport() => throw new NotImplementedException();
}

// ═══════════════════════════════════════════════════════════════════════════
// ✅ GOOD: Segregated interfaces
// ═══════════════════════════════════════════════════════════════════════════

public interface IWorkable
{
    void Work();
}

public interface IEatable
{
    void Eat();
}

public interface ISleepable
{
    void Sleep();
}

public interface IMeetingAttendee
{
    void AttendMeeting();
}

// Human implements all relevant interfaces
public class Human : IWorkable, IEatable, ISleepable, IMeetingAttendee
{
    public void Work() => Console.WriteLine("Working...");
    public void Eat() => Console.WriteLine("Eating...");
    public void Sleep() => Console.WriteLine("Sleeping...");
    public void AttendMeeting() => Console.WriteLine("In meeting...");
}

// Robot only implements what it can do
public class Robot : IWorkable
{
    public void Work() => Console.WriteLine("Working 24/7...");
    // No forced empty implementations!
}
```

---

## 🟣 Dependency Inversion Principle

> **"Depend on abstractions, not concretions"**

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// ❌ BAD: High-level depends on low-level concrete class
// ═══════════════════════════════════════════════════════════════════════════

public class EmailNotification
{
    public void Send(string message) { /* send email */ }
}

public class OrderService
{
    private EmailNotification _notifier = new EmailNotification();
    // Depends on CONCRETE class - can't change to SMS without modifying
    
    public void PlaceOrder(Order order)
    {
        // Process order
        _notifier.Send("Order placed");
    }
}

// ═══════════════════════════════════════════════════════════════════════════
// ✅ GOOD: Both depend on abstraction
// ═══════════════════════════════════════════════════════════════════════════

// Abstraction
public interface INotification
{
    void Send(string message);
}

// Low-level modules implement abstraction
public class EmailNotification : INotification
{
    public void Send(string message) { /* email */ }
}

public class SmsNotification : INotification
{
    public void Send(string message) { /* SMS */ }
}

public class PushNotification : INotification
{
    public void Send(string message) { /* push */ }
}

// High-level module depends on abstraction
public class OrderService
{
    private readonly INotification _notifier;
    
    public OrderService(INotification notifier)  // Injected abstraction
    {
        _notifier = notifier;
    }
    
    public void PlaceOrder(Order order)
    {
        // Process order
        _notifier.Send("Order placed");
    }
}

// Can now use any notification type
var emailOrder = new OrderService(new EmailNotification());
var smsOrder = new OrderService(new SmsNotification());
```

---

## 🎤 Interview Questions

**Q1: What is Single Responsibility?**
> A class should have only one reason to change - one job/responsibility.

**Q2: Explain Open/Closed Principle?**
> Classes should be open for extension (add new behavior) but closed for modification (don't change existing code).

**Q3: What is Liskov Substitution?**
> Subclasses must be substitutable for their base class without breaking the program.

**Q4: Why Interface Segregation?**
> Many specific interfaces are better than one general interface. Clients shouldn't depend on methods they don't use.

**Q5: Dependency Inversion vs Dependency Injection?**
> DI Principle = depend on abstractions. DI Pattern = technique to inject dependencies.

---

## 📝 Summary

| Principle | Rule | Violation Sign |
|-----------|------|----------------|
| **SRP** | One reason to change | Class does too many things |
| **OCP** | Extend, don't modify | if/switch for types |
| **LSP** | Subtypes substitutable | throw NotImplemented |
| **ISP** | Small interfaces | Empty method implementations |
| **DIP** | Depend on abstractions | `new ConcreteClass()` |
