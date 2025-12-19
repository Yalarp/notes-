# 🧵 Threading Fundamentals in C#

## 📚 Table of Contents
- [Overview](#-overview)
- [Creating Threads](#-creating-threads)
- [Thread Synchronization](#-thread-synchronization)
- [Thread Pool](#-thread-pool)
- [Best Practices](#-best-practices)
- [Interview Questions](#-interview-questions)

---

## 🎯 Overview

**Threading** allows concurrent execution of code, improving performance for CPU-bound and I/O-bound operations.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    THREADING CONCEPTS                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Process                                                           │
│   ┌─────────────────────────────────────────────────────────┐      │
│   │ Main Thread ─────────▶ Executes Program.Main()          │      │
│   │                                                          │      │
│   │ Worker Thread 1 ────▶ Background task                   │      │
│   │                                                          │      │
│   │ Worker Thread 2 ────▶ Another background task           │      │
│   └─────────────────────────────────────────────────────────┘      │
│                                                                     │
│   All threads share: Heap memory, static variables                 │
│   Each thread has: Own stack, local variables                      │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔷 Creating Threads

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// CREATING AND STARTING THREADS
// ═══════════════════════════════════════════════════════════════════════════

using System;
using System.Threading;

class ThreadDemo
{
    static void Main()
    {
        // ─────────────────────────────────────────────────────────────────
        // METHOD 1: Using ThreadStart delegate (no parameters)
        // ─────────────────────────────────────────────────────────────────
        
        Thread thread1 = new Thread(DoWork);
        // Line 1: Create thread with method reference
        
        thread1.Start();
        // Line 2: Start thread execution (asynchronous)
        
        // ─────────────────────────────────────────────────────────────────
        // METHOD 2: Using lambda expression
        // ─────────────────────────────────────────────────────────────────
        
        Thread thread2 = new Thread(() =>
        {
            Console.WriteLine("Thread 2: Lambda execution");
        });
        thread2.Start();
        
        // ─────────────────────────────────────────────────────────────────
        // METHOD 3: With parameters (ParameterizedThreadStart)
        // ─────────────────────────────────────────────────────────────────
        
        Thread thread3 = new Thread(DoWorkWithParam);
        thread3.Start("Hello from parameter");
        // Line 3: Pass parameter to thread method
        
        // ─────────────────────────────────────────────────────────────────
        // THREAD PROPERTIES
        // ─────────────────────────────────────────────────────────────────
        
        Thread current = Thread.CurrentThread;
        // Line 4: Get currently executing thread
        
        Console.WriteLine($"Thread ID: {current.ManagedThreadId}");
        Console.WriteLine($"Thread Name: {current.Name}");
        Console.WriteLine($"Is Background: {current.IsBackground}");
        Console.WriteLine($"Is Alive: {current.IsAlive}");
        Console.WriteLine($"Priority: {current.Priority}");
        
        // Set thread name for debugging
        thread1.Name = "Worker-1";
        
        // Set priority (Normal, AboveNormal, BelowNormal, Highest, Lowest)
        thread1.Priority = ThreadPriority.AboveNormal;
        
        // Background thread - doesn't keep app running
        thread1.IsBackground = true;
        // Line 5: Background threads terminate when main thread ends
        
        // ─────────────────────────────────────────────────────────────────
        // WAITING FOR THREAD
        // ─────────────────────────────────────────────────────────────────
        
        thread1.Join();
        // Line 6: Block until thread1 completes
        
        thread2.Join(1000);
        // Line 7: Wait maximum 1000ms, then continue
        
        Console.WriteLine("All threads completed");
    }
    
    static void DoWork()
    {
        for (int i = 0; i < 5; i++)
        {
            Console.WriteLine($"Thread 1: Working {i}");
            Thread.Sleep(100);
            // Line 8: Pause thread for 100ms
        }
    }
    
    static void DoWorkWithParam(object param)
    {
        string message = (string)param;
        Console.WriteLine($"Thread 3: {message}");
    }
}
```

---

## 🔶 Thread Synchronization

Prevent race conditions when multiple threads access shared resources.

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// RACE CONDITION PROBLEM
// ═══════════════════════════════════════════════════════════════════════════

class RaceConditionDemo
{
    private static int _counter = 0;
    
    static void Main()
    {
        Thread t1 = new Thread(Increment);
        Thread t2 = new Thread(Increment);
        
        t1.Start();
        t2.Start();
        
        t1.Join();
        t2.Join();
        
        Console.WriteLine($"Counter: {_counter}");
        // Expected: 20000, Actual: Random value < 20000!
        // Race condition: both threads read/write _counter simultaneously
    }
    
    static void Increment()
    {
        for (int i = 0; i < 10000; i++)
        {
            _counter++;
            // Line 1: NOT atomic! Read -> Add -> Write
            // Thread A reads 5
            // Thread B reads 5
            // Thread A writes 6
            // Thread B writes 6  <- Overwrites A's work!
        }
    }
}

// ═══════════════════════════════════════════════════════════════════════════
// SOLUTION 1: lock STATEMENT
// ═══════════════════════════════════════════════════════════════════════════

class LockDemo
{
    private static int _counter = 0;
    private static readonly object _lockObject = new object();
    // Line 2: Lock object - use dedicated object for locking
    
    static void Main()
    {
        Thread t1 = new Thread(SafeIncrement);
        Thread t2 = new Thread(SafeIncrement);
        
        t1.Start();
        t2.Start();
        t1.Join();
        t2.Join();
        
        Console.WriteLine($"Counter: {_counter}");
        // Always 20000 now!
    }
    
    static void SafeIncrement()
    {
        for (int i = 0; i < 10000; i++)
        {
            lock (_lockObject)
            // Line 3: Only one thread can enter lock block at a time
            {
                _counter++;
                // Critical section - protected from concurrent access
            }
            // Lock released when exiting block
        }
    }
}

// ═══════════════════════════════════════════════════════════════════════════
// SOLUTION 2: Interlocked CLASS (for simple operations)
// ═══════════════════════════════════════════════════════════════════════════

class InterlockedDemo
{
    private static int _counter = 0;
    
    static void AtomicIncrement()
    {
        for (int i = 0; i < 10000; i++)
        {
            Interlocked.Increment(ref _counter);
            // Line 4: Atomic increment - thread-safe without lock
            
            // Other Interlocked methods:
            // Interlocked.Decrement(ref value);
            // Interlocked.Add(ref value, amount);
            // Interlocked.Exchange(ref location, newValue);
            // Interlocked.CompareExchange(ref location, newValue, comparand);
        }
    }
}

// ═══════════════════════════════════════════════════════════════════════════
// SOLUTION 3: Monitor CLASS (advanced lock)
// ═══════════════════════════════════════════════════════════════════════════

class MonitorDemo
{
    private static object _lock = new object();
    
    static void MonitorExample()
    {
        bool lockTaken = false;
        try
        {
            Monitor.Enter(_lock, ref lockTaken);
            // Line 5: Acquire lock, set lockTaken flag
            
            // Critical section
        }
        finally
        {
            if (lockTaken)
            {
                Monitor.Exit(_lock);
                // Line 6: Release lock
            }
        }
        
        // TryEnter with timeout
        if (Monitor.TryEnter(_lock, TimeSpan.FromSeconds(1)))
        {
            try { /* work */ }
            finally { Monitor.Exit(_lock); }
        }
        else
        {
            Console.WriteLine("Could not acquire lock");
        }
    }
}
```

---

## 🔵 Thread Pool

Reusable pool of threads managed by .NET runtime.

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// THREADPOOL - MANAGED THREAD REUSE
// ═══════════════════════════════════════════════════════════════════════════

using System;
using System.Threading;

class ThreadPoolDemo
{
    static void Main()
    {
        // ─────────────────────────────────────────────────────────────────
        // Queue work to thread pool
        // ─────────────────────────────────────────────────────────────────
        
        ThreadPool.QueueUserWorkItem(DoPoolWork);
        // Line 1: Queue work item to thread pool
        //         Thread from pool executes the method
        
        ThreadPool.QueueUserWorkItem(DoPoolWork, "Parameter");
        // Line 2: With parameter
        
        // ─────────────────────────────────────────────────────────────────
        // Thread pool configuration
        // ─────────────────────────────────────────────────────────────────
        
        int workerThreads, completionPortThreads;
        ThreadPool.GetMinThreads(out workerThreads, out completionPortThreads);
        Console.WriteLine($"Min workers: {workerThreads}");
        
        ThreadPool.GetMaxThreads(out workerThreads, out completionPortThreads);
        Console.WriteLine($"Max workers: {workerThreads}");
        
        // Wait for pool work to complete
        Thread.Sleep(1000);
    }
    
    static void DoPoolWork(object state)
    {
        int threadId = Thread.CurrentThread.ManagedThreadId;
        Console.WriteLine($"Thread pool thread {threadId}: {state}");
    }
}

// Thread Pool vs Manual Thread:
// ┌─────────────────────────────────────────────────────────────────┐
// │ Manual Thread              │ Thread Pool                       │
// ├────────────────────────────┼───────────────────────────────────┤
// │ Create new thread each time│ Reuses existing threads           │
// │ More control (name, priority)│ Limited control                │
// │ Expensive to create        │ Cheaper (reused)                  │
// │ Can be foreground          │ Always background                 │
// │ You manage lifecycle       │ Runtime manages                   │
// └────────────────────────────┴───────────────────────────────────┘
```

---

## ✅ Best Practices

### 1. Prefer Task over Thread
```csharp
// ✅ MODERN: Use Task (builds on ThreadPool)
Task.Run(() => DoWork());

// ❌ OLD: Manual thread creation
new Thread(DoWork).Start();
```

### 2. Lock Rules
```csharp
// ✅ GOOD: Dedicated lock object
private readonly object _lock = new object();

// ❌ BAD: Locking on 'this' or public objects
lock (this) { }  // Other code can lock same object!
lock ("string") { }  // String interning issues
```

### 3. Keep Lock Duration Short
```csharp
// ✅ GOOD: Lock only critical section
lock (_lock)
{
    _counter++;  // Just the critical operation
}

// ❌ BAD: Long operations in lock
lock (_lock)
{
    DoExpensiveWork();  // Blocks other threads
}
```

---

## 🎤 Interview Questions

**Q1: What is thread?**
> Smallest unit of execution within a process. Threads share heap memory but have own stack.

**Q2: Difference between foreground and background threads?**
> Foreground threads keep app running until complete. Background threads terminate when all foreground threads end.

**Q3: What is race condition?**
> Multiple threads access shared resource simultaneously causing unpredictable results.

**Q4: How does lock work?**
> Ensures only one thread enters critical section at a time. Uses Monitor.Enter/Exit internally.

**Q5: Thread vs ThreadPool?**
> Thread creates new thread (expensive). ThreadPool reuses threads (efficient for short tasks).

---

## 📝 Summary

| Concept | Purpose |
|---------|---------|
| `Thread` | Manual thread creation |
| `Thread.Start()` | Begin execution |
| `Thread.Join()` | Wait for completion |
| `lock` | Mutual exclusion |
| `Interlocked` | Atomic operations |
| `ThreadPool` | Managed thread reuse |
