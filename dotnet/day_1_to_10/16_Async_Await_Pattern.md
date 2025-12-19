# ⏱️ Async/Await Pattern in C#

## 📚 Table of Contents
- [Overview](#-overview)
- [async and await Keywords](#-async-and-await-keywords)
- [Task and Task<T>](#-task-and-taskt)
- [Error Handling](#-error-handling)
- [Cancellation](#-cancellation)
- [Best Practices](#-best-practices)
- [Interview Questions](#-interview-questions)

---

## 🎯 Overview

**Async/await** enables non-blocking asynchronous programming without callback complexity.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SYNC vs ASYNC                                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  SYNCHRONOUS (Blocking)                                             │
│  Main Thread: ════════▶[Wait for I/O]════════▶ Continue             │
│               Thread blocked, doing nothing                         │
│                                                                     │
│  ASYNCHRONOUS (Non-blocking)                                        │
│  Main Thread: ════════▶ Start I/O ═══════════▶ Continue             │
│                              │                        │             │
│  I/O Operation:              └───────[Working]────────┘             │
│                                        ▲                            │
│                                   Callback when done                │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔷 async and await Keywords

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// ASYNC/AWAIT BASICS
// ═══════════════════════════════════════════════════════════════════════════

using System;
using System.Net.Http;
using System.Threading.Tasks;

class AsyncAwaitDemo
{
    static async Task Main()
    // Line 1: async Main - entry point can be async (C# 7.1+)
    {
        Console.WriteLine("Starting...");
        
        // ─────────────────────────────────────────────────────────────────
        // CALLING ASYNC METHOD
        // ─────────────────────────────────────────────────────────────────
        
        string result = await DownloadPageAsync("https://example.com");
        // Line 2: await pauses here until task completes
        //         BUT doesn't block the thread!
        //         Thread is released to do other work
        
        Console.WriteLine($"Downloaded {result.Length} chars");
    }
    
    // ─────────────────────────────────────────────────────────────────────
    // ASYNC METHOD SIGNATURE
    // ─────────────────────────────────────────────────────────────────────
    
    static async Task<string> DownloadPageAsync(string url)
    // Line 3: async modifier enables await inside method
    // Line 4: Task<string> = asynchronously returns string
    {
        using HttpClient client = new HttpClient();
        
        string content = await client.GetStringAsync(url);
        // Line 5: GetStringAsync is async I/O operation
        //         await releases thread during download
        
        return content;
        // Line 6: Return string, automatically wrapped in Task<string>
    }
    
    // ─────────────────────────────────────────────────────────────────────
    // RETURN TYPES
    // ─────────────────────────────────────────────────────────────────────
    
    // Task - no return value (like void)
    static async Task DoWorkAsync()
    {
        await Task.Delay(1000);
        Console.WriteLine("Work done");
    }
    
    // Task<T> - returns value
    static async Task<int> CalculateAsync()
    {
        await Task.Delay(100);
        return 42;
    }
    
    // void - ONLY for event handlers (fire-and-forget)
    async void Button_Click(object sender, EventArgs e)
    // Line 7: async void - can't be awaited, exceptions lost
    //         Use ONLY for event handlers!
    {
        await DoWorkAsync();
    }
}
```

### Execution Flow

```
┌─────────────────────────────────────────────────────────────────────┐
│                    ASYNC/AWAIT EXECUTION FLOW                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Main Thread                                                        │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ 1. Call DownloadPageAsync()                                  │   │
│  │ 2. Start HTTP request                                        │   │
│  │ 3. Hit 'await' → Thread RELEASED (not blocked!)             │   │
│  │                                                               │   │
│  │    ...thread can do other work...                            │   │
│  │                                                               │   │
│  │ 4. HTTP response arrives                                     │   │
│  │ 5. Continuation scheduled                                    │   │
│  │ 6. Resume after await with result                            │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  Key: Thread is NOT blocked during I/O wait                        │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔶 Task and Task<T>

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// TASK CREATION AND USAGE
// ═══════════════════════════════════════════════════════════════════════════

using System;
using System.Threading.Tasks;

class TaskDemo
{
    static async Task Main()
    {
        // ─────────────────────────────────────────────────────────────────
        // Task.Run - Execute CPU-bound work on thread pool
        // ─────────────────────────────────────────────────────────────────
        
        Task<int> task = Task.Run(() =>
        {
            // Line 1: Runs on thread pool thread
            return ExpensiveCalculation();
        });
        
        int result = await task;
        // Line 2: Wait for calculation without blocking
        
        // ─────────────────────────────────────────────────────────────────
        // Task.Delay - Non-blocking wait
        // ─────────────────────────────────────────────────────────────────
        
        await Task.Delay(1000);
        // Line 3: Waits 1 second without blocking thread
        //         Unlike Thread.Sleep which blocks!
        
        // ─────────────────────────────────────────────────────────────────
        // Multiple Tasks - Run concurrently
        // ─────────────────────────────────────────────────────────────────
        
        Task<string> task1 = DownloadAsync("url1");
        Task<string> task2 = DownloadAsync("url2");
        Task<string> task3 = DownloadAsync("url3");
        // Line 4: All three start immediately (concurrent)
        
        // Wait for all
        string[] results = await Task.WhenAll(task1, task2, task3);
        // Line 5: WhenAll completes when ALL tasks complete
        //         Returns array of results
        
        // Wait for any (first to complete)
        Task<string> firstComplete = await Task.WhenAny(task1, task2, task3);
        // Line 6: WhenAny returns first completed task
        
        // ─────────────────────────────────────────────────────────────────
        // Task.FromResult - Already completed task
        // ─────────────────────────────────────────────────────────────────
        
        Task<int> cached = Task.FromResult(42);
        // Line 7: Creates already-completed task with value
        //         Useful for caching or mock implementations
        
        // ─────────────────────────────────────────────────────────────────
        // Task Status
        // ─────────────────────────────────────────────────────────────────
        
        Task t = Task.Run(() => { });
        
        bool isComplete = t.IsCompleted;
        bool isFaulted = t.IsFaulted;      // Exception occurred
        bool isCanceled = t.IsCanceled;
        TaskStatus status = t.Status;
    }
    
    static int ExpensiveCalculation() => 42;
    static async Task<string> DownloadAsync(string url) 
    {
        await Task.Delay(100);
        return url;
    }
}
```

---

## 🔵 Error Handling

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// ASYNC ERROR HANDLING
// ═══════════════════════════════════════════════════════════════════════════

using System;
using System.Threading.Tasks;

class AsyncErrorDemo
{
    static async Task Main()
    {
        // ─────────────────────────────────────────────────────────────────
        // Try-Catch with await
        // ─────────────────────────────────────────────────────────────────
        
        try
        {
            await MightFailAsync();
        }
        catch (InvalidOperationException ex)
        // Line 1: Exception is unwrapped and caught directly
        {
            Console.WriteLine($"Caught: {ex.Message}");
        }
        
        // ─────────────────────────────────────────────────────────────────
        // Multiple tasks - aggregate exceptions
        // ─────────────────────────────────────────────────────────────────
        
        Task t1 = Task.Run(() => throw new Exception("Error 1"));
        Task t2 = Task.Run(() => throw new Exception("Error 2"));
        
        try
        {
            await Task.WhenAll(t1, t2);
        }
        catch (Exception ex)
        {
            // Line 2: Only first exception caught with await
            Console.WriteLine(ex.Message);
        }
        
        // To get all exceptions:
        Task allTasks = Task.WhenAll(t1, t2);
        try { await allTasks; }
        catch
        {
            AggregateException all = allTasks.Exception;
            // Line 3: Task.Exception contains AggregateException
            foreach (var inner in all.InnerExceptions)
            {
                Console.WriteLine(inner.Message);
            }
        }
    }
    
    static async Task MightFailAsync()
    {
        await Task.Delay(100);
        throw new InvalidOperationException("Something failed");
    }
}
```

---

## 🟢 Cancellation

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// CANCELLATION TOKENS
// ═══════════════════════════════════════════════════════════════════════════

using System;
using System.Threading;
using System.Threading.Tasks;

class CancellationDemo
{
    static async Task Main()
    {
        // Line 1: Create cancellation token source
        CancellationTokenSource cts = new CancellationTokenSource();
        
        // Start long-running task
        Task task = LongRunningAsync(cts.Token);
        
        // Cancel after 2 seconds
        await Task.Delay(2000);
        cts.Cancel();
        // Line 2: Request cancellation
        
        try
        {
            await task;
        }
        catch (OperationCanceledException)
        // Line 3: Task throws when cancelled
        {
            Console.WriteLine("Task was cancelled");
        }
        
        // ─────────────────────────────────────────────────────────────────
        // Auto-cancel after timeout
        // ─────────────────────────────────────────────────────────────────
        
        var ctsWithTimeout = new CancellationTokenSource(TimeSpan.FromSeconds(5));
        // Line 4: Automatically cancels after 5 seconds
    }
    
    static async Task LongRunningAsync(CancellationToken token)
    {
        for (int i = 0; i < 100; i++)
        {
            // Check cancellation periodically
            token.ThrowIfCancellationRequested();
            // Line 5: Throws OperationCanceledException if cancelled
            
            await Task.Delay(100, token);
            // Line 6: Pass token to delay - throws if cancelled during wait
            
            Console.WriteLine($"Progress: {i}%");
        }
    }
}
```

---

## ✅ Best Practices

### 1. async All the Way
```csharp
// ✅ GOOD: async throughout the call chain
async Task<Data> GetDataAsync()
{
    return await repository.GetAsync();
}

// ❌ BAD: Blocking on async (deadlock risk!)
Data GetData()
{
    return repository.GetAsync().Result;  // BLOCKS!
}
```

### 2. Avoid async void
```csharp
// ✅ GOOD: Return Task
async Task DoWorkAsync() { }

// ❌ BAD: async void (except event handlers)
async void DoWork() { }  // Can't await, exceptions lost
```

### 3. Use ConfigureAwait(false) in Libraries
```csharp
// In library code - don't capture sync context
await SomeAsync().ConfigureAwait(false);
```

---

## 🎤 Interview Questions

**Q1: What does async do?**
> Enables await keyword in method. Method returns Task/Task<T> and can be paused/resumed.

**Q2: What does await do?**
> Asynchronously waits for Task completion. Releases thread during wait, resumes when done.

**Q3: Task vs Thread?**
> Task is higher-level abstraction using thread pool. Thread is OS thread. Task is preferred.

**Q4: Why avoid async void?**
> Can't be awaited, exceptions can't be caught normally. Only use for event handlers.

**Q5: What is Task.WhenAll?**
> Waits for all tasks to complete. Returns when all done. Collects all results.

---

## 📝 Summary

| Concept | Purpose |
|---------|---------|
| `async` | Enable await in method |
| `await` | Non-blocking wait for task |
| `Task` | Async operation (no result) |
| `Task<T>` | Async operation with result |
| `Task.Run` | Run on thread pool |
| `Task.WhenAll` | Wait for all tasks |
| `CancellationToken` | Cancel async operations |
