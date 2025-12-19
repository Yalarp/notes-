# 📦 Inversion of Control (IOC) and Container Patterns

## 📚 Table of Contents
- [What is IOC?](#-what-is-ioc)
- [IOC vs DI](#-ioc-vs-di)
- [IOC Container](#-ioc-container)
- [Manual DI vs Container](#-manual-di-vs-container)
- [Interview Questions](#-interview-questions)

---

## 🎯 What is IOC?

**Inversion of Control** means inverting the flow of control - instead of your code controlling dependencies, a framework/container controls them.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    NORMAL CONTROL                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Your Class ────────▶ Creates ────────▶ Dependencies              │
│   (OrderService)        (new)            (EmailService)            │
│                                                                     │
│   class OrderService {                                              │
│       EmailService email = new EmailService();  // You control     │
│   }                                                                 │
│                                                                     │
├─────────────────────────────────────────────────────────────────────┤
│                    INVERTED CONTROL                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Container ────────▶ Creates ────────▶ Your Class                 │
│   (Framework)          (injects)         + Dependencies            │
│                                                                     │
│   var service = container.Resolve<OrderService>();                 │
│   // Container creates OrderService AND its dependencies           │
│                                                                     │
│   Control is INVERTED: Framework controls object creation          │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔷 IOC vs DI

```
┌─────────────────────────────────────────────────────────────────────┐
│                    IOC vs DI RELATIONSHIP                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   IOC = PRINCIPLE (The "What")                                     │
│   "Invert control of object creation"                              │
│                                                                     │
│   DI = TECHNIQUE (The "How")                                       │
│   "Inject dependencies via constructor/property/method"           │
│                                                                     │
│              ┌────────────────────────────────┐                    │
│              │           IOC                  │                    │
│              │  (Inversion of Control)        │                    │
│              │                                │                    │
│              │   ┌──────────────────────┐    │                    │
│              │   │         DI           │    │                    │
│              │   │ (Dependency Injection)│    │                    │
│              │   └──────────────────────┘    │                    │
│              │                                │                    │
│              │   Service Locator  ◄── Another IoC implementation  │
│              │   Factory Pattern  ◄── Another IoC implementation  │
│              └────────────────────────────────┘                    │
│                                                                     │
│   DI is the most common way to implement IOC                       │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔶 IOC Container

An IOC Container is a framework that manages object creation and dependency resolution.

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// POPULAR IOC CONTAINERS
// ═══════════════════════════════════════════════════════════════════════════

// 1. Built-in .NET Core DI
using Microsoft.Extensions.DependencyInjection;

// 2. Autofac
using Autofac;

// 3. Ninject
using Ninject;

// 4. Unity
using Unity;

// ═══════════════════════════════════════════════════════════════════════════
// SIMPLE IOC CONTAINER IMPLEMENTATION
// ═══════════════════════════════════════════════════════════════════════════

public class SimpleContainer
{
    private Dictionary<Type, Type> _registrations = new();
    private Dictionary<Type, object> _singletons = new();
    
    // Register interface to implementation
    public void Register<TInterface, TImplementation>() where TImplementation : TInterface
    {
        _registrations[typeof(TInterface)] = typeof(TImplementation);
    }
    
    // Register singleton
    public void RegisterSingleton<TInterface, TImplementation>() where TImplementation : TInterface
    {
        Register<TInterface, TImplementation>();
        _singletons[typeof(TInterface)] = null;  // Mark as singleton
    }
    
    // Resolve (create) instance
    public T Resolve<T>()
    {
        return (T)Resolve(typeof(T));
    }
    
    private object Resolve(Type type)
    {
        // Check if singleton exists
        if (_singletons.ContainsKey(type) && _singletons[type] != null)
            return _singletons[type];
        
        // Get implementation type
        Type implementationType = _registrations.ContainsKey(type) 
            ? _registrations[type] 
            : type;
        
        // Get constructor with most parameters (greedy)
        var constructor = implementationType.GetConstructors()
            .OrderByDescending(c => c.GetParameters().Length)
            .First();
        
        // Recursively resolve constructor parameters
        var parameters = constructor.GetParameters()
            .Select(p => Resolve(p.ParameterType))
            .ToArray();
        
        // Create instance
        var instance = Activator.CreateInstance(implementationType, parameters);
        
        // Store if singleton
        if (_singletons.ContainsKey(type))
            _singletons[type] = instance;
        
        return instance;
    }
}

// Usage
var container = new SimpleContainer();
container.Register<IEmailService, EmailService>();
container.Register<IOrderRepository, SqlOrderRepository>();
container.Register<OrderService, OrderService>();

// Container creates OrderService AND injects its dependencies
var orderService = container.Resolve<OrderService>();
```

---

## 🔵 Manual DI vs Container

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// MANUAL DI (Without Container)
// ═══════════════════════════════════════════════════════════════════════════

// You manually wire up dependencies
class Program
{
    static void Main()
    {
        // Create dependencies manually
        ILogger logger = new FileLogger();
        IDatabase db = new SqlDatabase(logger);
        IEmailService email = new EmailService(logger);
        IOrderRepository orderRepo = new OrderRepository(db, logger);
        
        // Create service with all dependencies
        var orderService = new OrderService(orderRepo, email, logger);
        
        // Use service
        orderService.PlaceOrder(new Order());
    }
}

// Problems with manual DI:
// - Tedious for many dependencies
// - Must maintain creation order
// - Hard to manage lifetimes

// ═══════════════════════════════════════════════════════════════════════════
// CONTAINER DI (With Container)
// ═══════════════════════════════════════════════════════════════════════════

class Program
{
    static void Main()
    {
        // Register once
        var services = new ServiceCollection();
        services.AddSingleton<ILogger, FileLogger>();
        services.AddScoped<IDatabase, SqlDatabase>();
        services.AddScoped<IEmailService, EmailService>();
        services.AddScoped<IOrderRepository, OrderRepository>();
        services.AddScoped<OrderService>();
        
        var provider = services.BuildServiceProvider();
        
        // Container resolves entire dependency graph
        var orderService = provider.GetRequiredService<OrderService>();
        // Container automatically created:
        // 1. FileLogger (singleton)
        // 2. SqlDatabase (with logger)
        // 3. EmailService (with logger)
        // 4. OrderRepository (with db and logger)
        // 5. OrderService (with repo, email, logger)
    }
}
```

### Container Benefits

| Aspect | Manual DI | Container DI |
|--------|-----------|--------------|
| Setup | Verbose | Declarative |
| Lifetime | Manual tracking | Automatic |
| Deep graphs | Complex | Automatic resolution |
| Testing | Same effort | Same effort |
| Performance | Slightly faster | Slightly slower |

---

## 🟢 Service Locator Pattern (Anti-Pattern)

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// SERVICE LOCATOR - GENERALLY CONSIDERED ANTI-PATTERN
// ═══════════════════════════════════════════════════════════════════════════

// Service Locator: Class asks for dependencies
public class OrderService
{
    private readonly IEmailService _email;
    
    public OrderService()
    {
        // ❌ BAD: Hides dependencies, hard to test
        _email = ServiceLocator.Get<IEmailService>();
    }
}

public static class ServiceLocator
{
    private static IServiceProvider _provider;
    
    public static void SetProvider(IServiceProvider provider) 
        => _provider = provider;
    
    public static T Get<T>() => _provider.GetService<T>();
}

// Problems:
// 1. Hidden dependencies (not visible in constructor)
// 2. Hard to test (must setup global state)
// 3. Violates explicit dependencies principle

// ✅ BETTER: Constructor Injection
public class OrderService
{
    private readonly IEmailService _email;
    
    public OrderService(IEmailService email)  // Dependencies visible!
    {
        _email = email;
    }
}
```

---

## 🎤 Interview Questions

**Q1: What is IOC?**
> Inversion of Control - transferring control of object creation from your code to a framework/container.

**Q2: IOC vs DI?**
> IOC is the principle (invert control). DI is the technique (inject dependencies via constructor).

**Q3: What is an IOC Container?**
> Framework that manages object creation and dependency resolution automatically.

**Q4: Why is Service Locator an anti-pattern?**
> Hides dependencies, makes testing harder, violates explicit dependencies principle.

**Q5: Benefits of IOC Container over manual DI?**
> Automatic dependency resolution, lifetime management, cleaner code for complex graphs.

---

## 📝 Summary

| Concept | Description |
|---------|-------------|
| **IOC** | Principle of inverting control |
| **DI** | Technique to implement IOC |
| **IOC Container** | Framework managing dependencies |
| **Register** | Tell container about types |
| **Resolve** | Ask container for instance |
| **Service Locator** | Anti-pattern, avoid |
