# Iterators and Yield

## Basic Iterator
The `yield` keyword creates an iterator method that produces values on-demand.

```csharp
public IEnumerable<int> GetNumbers()
{
    yield return 1;
    yield return 2;
    yield return 3;
}

// Usage
foreach (var num in GetNumbers())
{
    Console.WriteLine(num);  // 1, 2, 3
}
```

## Yield with Logic
```csharp
public IEnumerable<int> GetEvenUpTo(int max)
{
    for (int i = 0; i <= max; i += 2)
    {
        yield return i;
    }
}

public IEnumerable<int> Fibonacci(int count)
{
    int prev = 0, current = 1;
    for (int i = 0; i < count; i++)
    {
        yield return prev;
        int next = prev + current;
        prev = current;
        current = next;
    }
}
```

## Yield Break
```csharp
public IEnumerable<int> GetUntilNegative(params int[] numbers)
{
    foreach (var n in numbers)
    {
        if (n < 0)
            yield break;  // Stop iteration
        yield return n;
    }
}

// Usage
var result = GetUntilNegative(1, 2, 3, -1, 4, 5);
// Returns: 1, 2, 3
```

## Deferred Execution
```csharp
public IEnumerable<int> GetNumbers()
{
    Console.WriteLine("Starting iteration");
    for (int i = 0; i < 3; i++)
    {
        Console.WriteLine($"Producing {i}");
        yield return i;
    }
    Console.WriteLine("Ending iteration");
}

// Nothing executes until enumeration starts
var numbers = GetNumbers();
Console.WriteLine("Query defined");

foreach (var n in numbers)  // Execution starts here
{
    Console.WriteLine($"Consuming {n}");
}

// Output:
// Query defined
// Starting iteration
// Producing 0
// Consuming 0
// Producing 1
// Consuming 1
// Producing 2
// Consuming 2
// Ending iteration
```

## Lazy Enumeration Example
```csharp
public static IEnumerable<string> ReadLines(string path)
{
    using var reader = new StreamReader(path);
    string? line;
    while ((line = reader.ReadLine()) is not null)
    {
        yield return line;  // One line at a time
    }
}

// Efficient: doesn't load entire file
foreach (var line in ReadLines("large-file.txt"))
{
    if (line.Contains("error"))
        Console.WriteLine(line);
}
```

## Iterator with State
```csharp
public class PagedEnumerator<T>
{
    private readonly Func<int, int, IEnumerable<T>> _pageLoader;
    private readonly int _pageSize;

    public PagedEnumerator(Func<int, int, IEnumerable<T>> pageLoader, int pageSize)
    {
        _pageLoader = pageLoader;
        _pageSize = pageSize;
    }

    public IEnumerable<T> GetAll()
    {
        int page = 0;
        IEnumerable<T>? currentPage;

        do
        {
            currentPage = _pageLoader(page++, _pageSize);
            foreach (var item in currentPage)
            {
                yield return item;
            }
        } while (currentPage.Any());
    }
}
```

## Multiple Yield in One Method
```csharp
public IEnumerable<string> GetMessages()
{
    yield return "First";
    // Some processing
    yield return "Second";

    if (DateTime.Now.Hour < 12)
        yield return "Morning special";

    yield return "Last";
}
```

## Asynchronous Iterator (C# 8+)
```csharp
public async IAsyncEnumerable<int> GetNumbersAsync()
{
    for (int i = 0; i < 10; i++)
    {
        await Task.Delay(100);
        yield return i;
    }
}

// Consumption
await foreach (var num in GetNumbersAsync())
{
    Console.WriteLine(num);
}
```

## Iterator with Cancellation
```csharp
public async IAsyncEnumerable<int> GetNumbersAsync(
    [EnumeratorCancellation] CancellationToken cancellationToken = default)
{
    for (int i = 0; i < 100; i++)
    {
        cancellationToken.ThrowIfCancellationRequested();
        await Task.Delay(100, cancellationToken);
        yield return i;
    }
}

// Usage with cancellation
var cts = new CancellationTokenSource();
cts.CancelAfter(500);

await foreach (var num in GetNumbersAsync(cts.Token))
{
    Console.WriteLine(num);
}
```

## Returning IAsyncEnumerable
```csharp
public class DataService
{
    public async IAsyncEnumerable<DataItem> StreamData(int count)
    {
        for (int i = 0; i < count; i++)
        {
            var item = await FetchFromApiAsync(i);
            yield return item;
        }
    }
}
```

## Performance Characteristics

### Eager vs Lazy
```csharp
// Eager (List): all items computed now
List<int> eager = GetEvenUpTo(100).ToList();

// Lazy (IEnumerable): items computed on-demand
IEnumerable<int> lazy = GetEvenUpTo(100);

// Lazy can short-circuit
int first = GetEvenUpTo(100).First();  // Only computes first item
bool any = GetEvenUpTo(100).Any();     // Only computes first item
```

### Iterator Implementation
The compiler generates a state machine class implementing `IEnumerable<T>` and `IEnumerator<T>`:

```csharp
// The compiler transforms this:
public IEnumerable<int> GetNumbers()
{
    yield return 1;
    yield return 2;
}

// Into something like:
public IEnumerable<int> GetNumbers()
{
    return new <GetNumbers>d__0(0);  // Generated state machine
}
```

## Best Practices
- Use yield for large/infinite sequences
- Avoid modifying collections inside iterators
- Remember deferred execution (side effects may surprise)
- Use `ToList()`/`ToArray()` to materialize when needed
- `yield return` cannot be used in try-catch (only try-finally)
- `yield break` exits the iterator early
