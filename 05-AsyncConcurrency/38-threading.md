# Threading and Synchronization

## Thread Basics

```csharp
// Creating a thread
Thread thread = new Thread(() =>
{
    Console.WriteLine("Running in thread");
});
thread.Start();
thread.Join();  // Wait for completion

// Parameterized thread
Thread thread = new Thread(param =>
{
    int value = (int)param;
    Console.WriteLine($"Parameter: {value}");
});
thread.Start(42);

// Named thread
Thread thread = new Thread(DoWork)
{
    Name = "WorkerThread",
    IsBackground = true,  // App can exit without waiting
    Priority = ThreadPriority.Normal
};
```

## Thread Pool

```csharp
// Queue work to thread pool
ThreadPool.QueueUserWorkItem(state =>
{
    Console.WriteLine("Thread pool work");
});

// With parameter
ThreadPool.QueueUserWorkItem(param =>
{
    int id = (int)param;
    Process(id);
}, 42);

// Configure thread pool
ThreadPool.SetMinThreads(4, 4);
ThreadPool.SetMaxThreads(50, 50);
ThreadPool.GetAvailableThreads(out int workers, out int io);
```

## Synchronization

### lock Statement
```csharp
private readonly object _lock = new();
private int _counter;

public void Increment()
{
    lock (_lock)
    {
        _counter++;
    }
}

// Lock is equivalent to:
Monitor.Enter(_lock);
try
{
    _counter++;
}
finally
{
    Monitor.Exit(_lock);
}
```

### Monitor (Explicit Lock)
```csharp
private readonly object _lock = new();

public bool TryProcess()
{
    if (Monitor.TryEnter(_lock, TimeSpan.FromSeconds(1)))
    {
        try
        {
            // Critical section
            Process();
            return true;
        }
        finally
        {
            Monitor.Exit(_lock);
        }
    }
    return false;  // Couldn't acquire lock
}
```

### ReaderWriterLockSlim
```csharp
private readonly ReaderWriterLockSlim _rwLock = new();
private int _data;

public int Read()
{
    _rwLock.EnterReadLock();
    try { return _data; }
    finally { _rwLock.ExitReadLock(); }
}

public void Write(int value)
{
    _rwLock.EnterWriteLock();
    try { _data = value; }
    finally { _rwLock.ExitWriteLock(); }
}
```

### Mutex (Cross-Process)
```csharp
using (var mutex = new Mutex(false, "Global\\MyAppMutex"))
{
    if (!mutex.WaitOne(TimeSpan.FromSeconds(3)))
    {
        Console.WriteLine("Another instance running");
        return;
    }

    try
    {
        // Run application
        Run();
    }
    finally
    {
        mutex.ReleaseMutex();
    }
}
```

### Semaphore / SemaphoreSlim
```csharp
// Limit concurrent access (e.g., 3 threads)
private static readonly SemaphoreSlim _semaphore = new(3, 3);

public async Task AccessResourceAsync()
{
    await _semaphore.WaitAsync();
    try
    {
        await UseResourceAsync();
    }
    finally
    {
        _semaphore.Release();
    }
}
```

### AutoResetEvent / ManualResetEvent
```csharp
private readonly AutoResetEvent _event = new(false);

// Waiting thread
Task.Run(() =>
{
    Console.WriteLine("Waiting...");
    _event.WaitOne();  // Blocks until Set()
    Console.WriteLine("Proceeding");
});

// Signaling thread
Task.Run(() =>
{
    Thread.Sleep(1000);
    _event.Set();  // Release one waiting thread
});
```

## Thread-Safe Collections

```csharp
using System.Collections.Concurrent;

// ConcurrentDictionary
var dict = new ConcurrentDictionary<string, int>();
dict.TryAdd("key", 1);
dict.TryUpdate("key", 2, 1);
dict.AddOrUpdate("key", 1, (k, v) => v + 1);
dict.GetOrAdd("key", k => Compute(k));

// ConcurrentQueue (FIFO)
var queue = new ConcurrentQueue<int>();
queue.Enqueue(1);
bool success = queue.TryDequeue(out int result);

// ConcurrentStack (LIFO)
var stack = new ConcurrentStack<int>();
stack.Push(1);
bool success = stack.TryPop(out int result);

// ConcurrentBag (unordered)
var bag = new ConcurrentBag<int>();
bag.Add(1);
bool success = bag.TryTake(out int result);

// BlockingCollection (bounded producer/consumer)
var bc = new BlockingCollection<int>(boundedCapacity: 10);
// Producer
Task.Run(() => { bc.Add(1); bc.CompleteAdding(); });
// Consumer
foreach (var item in bc.GetConsumingEnumerable())
{
    Process(item);
}
```

## ThreadLocal<T>

```csharp
private static readonly ThreadLocal<int> _threadId = new(() =>
{
    return Thread.CurrentThread.ManagedThreadId;
});

// Each thread gets its own value
Parallel.For(0, 10, i =>
{
    Console.WriteLine($"Thread {_threadId.Value} processing {i}");
});
```

## Interlocked Operations

```csharp
private int _counter;

public void AtomicIncrement()
{
    Interlocked.Increment(ref _counter);
}

public void AtomicAdd(int value)
{
    Interlocked.Add(ref _counter, value);
}

public int AtomicExchange(int newValue)
{
    return Interlocked.Exchange(ref _counter, newValue);
}

public bool AtomicCompareExchange(int expected, int newValue)
{
    return Interlocked.CompareExchange(ref _counter, newValue, expected) == expected;
}
```

## Volatile

```csharp
private volatile bool _isRunning = true;

// Volatile ensures:
// 1. Fresh reads (no cached value)
// 2. All writes are visible to other threads

public void Worker()
{
    while (_isRunning)
    {
        Process();
    }
}
```

## Thread Safety Guidelines

| Pattern | Use When | Example |
|---------|----------|---------|
| `lock` | Simple critical sections | Counter increment |
| `Monitor.TryEnter` | Non-blocking lock attempt | Try-acquire |
| `ReaderWriterLockSlim` | Read-heavy, write-light | Cache |
| `SemaphoreSlim` | Resource pool, throttling | Connection pool |
| `ConcurrentDictionary` | Thread-safe dictionary | Shared cache |
| `ConcurrentQueue` | Producer-consumer | Work queue |
| `Interlocked` | Simple atomic operations | Counter |
| `volatile` | Simple status flags | Cancellation flag |
| `Mutex` | Cross-process sync | Single instance |
