# Task Parallel Library (TPL)

## Task Basics

```csharp
// Create and start a task
Task task = Task.Run(() => DoWork());

// Task with return value
Task<int> task = Task.Run(() => CalculateSum(100));

// Wait for result
int result = task.Result;  // Blocks! Prefer await
int result = await task;   // Non-blocking

// Status checking
Console.WriteLine(task.Status);  // Created, WaitingForActivation, RanToCompletion, Faulted, Canceled
Console.WriteLine(task.IsCompleted);
Console.WriteLine(task.IsFaulted);
Console.WriteLine(task.IsCanceled);
```

## Starting Tasks

```csharp
// Task.Run (preferred)
Task task = Task.Run(() => Process());
Task<int> task = Task.Run(() => Compute());

// Task.Factory.StartNew (more control)
Task task = Task.Factory.StartNew(
    () => Process(),
    CancellationToken.None,
    TaskCreationOptions.LongRunning,  // Hint: dedicated thread
    TaskScheduler.Default
);

// Task constructor (must manually Start)
var task = new Task(() => Process());
task.Start();
```

## Waiting on Tasks

```csharp
// Wait for single task
task.Wait();                         // Blocks
task.Wait(TimeSpan.FromSeconds(5));  // With timeout
bool completed = task.Wait(1000);    // Returns bool

// Wait for multiple tasks
Task.WaitAll(task1, task2, task3);    // All complete
Task.WaitAny(task1, task2, task3);    // Any complete

// Async equivalents
await Task.WhenAll(task1, task2);      // All complete
var first = await Task.WhenAny(t1, t2); // Any complete
```

## Continuations

```csharp
// Simple continuation
Task.Run(() => Process())
    .ContinueWith(previous => Cleanup());

// Conditional continuation
Task.Run(() => Process())
    .ContinueWith(t => HandleSuccess(), TaskContinuationOptions.OnlyOnRanToCompletion)
    .ContinueWith(t => HandleError(), TaskContinuationOptions.OnlyOnFaulted);
```

## Parallel Class

```csharp
// Parallel.For
Parallel.For(0, 100, i =>
{
    ProcessItem(i);
});

// Parallel.ForEach
Parallel.ForEach(items, item =>
{
    Process(item);
});

// Parallel.Invoke (multiple actions in parallel)
Parallel.Invoke(
    () => DoWork1(),
    () => DoWork2(),
    () => DoWork3()
);

// With degree of parallelism
var options = new ParallelOptions
{
    MaxDegreeOfParallelism = 4
};
Parallel.ForEach(items, options, item => Process(item));
```

## PLINQ (Parallel LINQ)

```csharp
// Sequential
var result = items.Where(x => ExpensiveCheck(x)).ToList();

// Parallel
var result = items.AsParallel()
    .Where(x => ExpensiveCheck(x))
    .ToList();

// Controlling parallelism
var result = items.AsParallel()
    .WithDegreeOfParallelism(4)
    .WithExecutionMode(ParallelExecutionMode.ForceParallelism)
    .Where(x => ExpensiveCheck(x))
    .OrderBy(x => x)  // Requires merging
    .ToList();
```

## Task Factory and Schedulers

```csharp
// SynchronizationContext scheduler (UI)
var uiScheduler = TaskScheduler.FromCurrentSynchronizationContext();

Task.Factory.StartNew(() => Compute())
    .ContinueWith(t => UpdateUI(t.Result), uiScheduler);

// Custom options
var factory = new TaskFactory(
    CancellationToken.None,
    TaskCreationOptions.AttachedToParent,
    TaskContinuationOptions.None,
    TaskScheduler.Default
);
```

## Parent and Child Tasks

```csharp
// Attached child tasks
Task parent = Task.Factory.StartNew(() =>
{
    Console.WriteLine("Parent starts");

    Task child = Task.Factory.StartNew(() =>
    {
        Console.WriteLine("Child starts");
        Thread.Sleep(1000);
        Console.WriteLine("Child ends");
    }, TaskCreationOptions.AttachedToParent);

    Console.WriteLine("Parent ends");
});  // Waits for child

// Detached (default) - parent doesn't wait
```

## TaskCompletionSource

```csharp
// Bridge async/await with non-Task operations
public Task<int> GetResultAsync()
{
    var tcs = new TaskCompletionSource<int>();

    // Some non-Task async operation
    SomeEventBasedApi.StartOperation(result =>
    {
        tcs.TrySetResult(result);
    }, error =>
    {
        tcs.TrySetException(error);
    });

    return tcs.Task;
}

// Cancellation support
public Task<int> GetResultAsync(CancellationToken ct)
{
    var tcs = new TaskCompletionSource<int>();
    ct.Register(() => tcs.TrySetCanceled());
    return tcs.Task;
}
```

## Data Flow (TransformBlock, etc.)
```csharp
using System.Threading.Tasks.Dataflow;

// Pipeline pattern
var downloadBlock = new TransformBlock<string, string>(async url =>
{
    using var client = new HttpClient();
    return await client.GetStringAsync(url);
});

var processBlock = new TransformBlock<string, int>(data =>
{
    return data.Length;
});

var outputBlock = new ActionBlock<int>(length =>
{
    Console.WriteLine($"Length: {length}");
});

// Link blocks
downloadBlock.LinkTo(processBlock);
processBlock.LinkTo(outputBlock);

// Post data
downloadBlock.Post("https://example.com");
downloadBlock.Complete();
await outputBlock.Completion;
```
