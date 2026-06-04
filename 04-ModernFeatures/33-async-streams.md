# Async Streams

## Introduction (C# 8+)
Async streams allow asynchronous iteration over data that arrives over time, using `IAsyncEnumerable<T>` and `await foreach`.

## Producing Async Streams

### Basic Async Iterator
```csharp
public async IAsyncEnumerable<int> GetNumbersAsync()
{
    for (int i = 0; i < 10; i++)
    {
        await Task.Delay(100);  // Simulated async work
        yield return i;
    }
}
```

### Real-World Example: API Pagination
```csharp
public async IAsyncEnumerable<Customer> GetAllCustomersAsync()
{
    int page = 0;
    bool hasMore = true;

    while (hasMore)
    {
        var response = await _httpClient.GetFromJsonAsync<PageResponse<Customer>>(
            $"api/customers?page={page}");

        if (response?.Items is null || response.Items.Count == 0)
            yield break;

        foreach (var customer in response.Items)
        {
            yield return customer;
        }

        hasMore = response.HasMore;
        page++;
    }
}
```

### Async File Reading
```csharp
public async IAsyncEnumerable<string> ReadLinesAsync(string path)
{
    using var reader = new StreamReader(path);
    string? line;

    while ((line = await reader.ReadLineAsync()) is not null)
    {
        yield return line;
    }
}
```

## Consuming Async Streams

### Basic Consumption
```csharp
await foreach (var number in GetNumbersAsync())
{
    Console.WriteLine(number);
}
```

### With Cancellation
```csharp
await foreach (var item in GetAllCustomersAsync()
    .WithCancellation(cancellationToken))
{
    Process(item);
}
```

### With Break
```csharp
await foreach (var customer in GetAllCustomersAsync())
{
    if (customer.IsPriority)
    {
        ProcessPriority(customer);
    }
    else if (customer.IsInactive)
    {
        break;  // Stop consuming
    }
}
```

## AsyncLINQ Operations
```csharp
// Using System.Linq.Async (NuGet: System.Linq.Async)

await foreach (var customer in GetAllCustomersAsync()
    .Where(c => c.Age > 18)
    .OrderBy(c => c.Name)
    .Take(10))
{
    Console.WriteLine(customer.Name);
}
```

## Error Handling
```csharp
try
{
    await foreach (var data in StreamDataAsync())
    {
        Process(data);
    }
}
catch (HttpRequestException ex)
{
    Console.WriteLine($"Network error: {ex.Message}");
}
catch (OperationCanceledException)
{
    Console.WriteLine("Cancelled");
}
```

## EnumeratorCancellation Attribute
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

// Usage
var cts = new CancellationTokenSource();
cts.CancelAfter(2000);

await foreach (var num in GetNumbersAsync()
    .WithCancellation(cts.Token))
{
    Console.WriteLine(num);
}
```

## ConfigureAwait with Async Streams
```csharp
await foreach (var item in GetDataAsync()
    .ConfigureAwait(false))
{
    Process(item);
}
```

## Converting IEnumerable to IAsyncEnumerable
```csharp
public static async IAsyncEnumerable<T> ToAsyncEnumerable<T>(
    this IEnumerable<T> source,
    [EnumeratorCancellation] CancellationToken ct = default)
{
    foreach (var item in source)
    {
        ct.ThrowIfCancellationRequested();
        await Task.CompletedTask;  // Or some async work
        yield return item;
    }
}

// Or use System.Linq.Async: source.ToAsyncEnumerable()
```

## Performance Considerations
```csharp
// Async streams are pull-based (lazy) - each item fetched on demand
// Good for: large datasets, streaming, pagination
// Not good for: small in-memory collections (use IEnumerable)

// Avoid multiple iterations (async streams aren't cached)
// Bad:
var stream = GetDataAsync();
var count = await stream.CountAsync();  // Iterates once
await foreach (var item in stream)     // Iterates again (empty!)
{
    Process(item);
}

// Good: materialize if you need multiple iterations
var list = await GetDataAsync().ToListAsync();
```

## Retry Pattern with Async Streams
```csharp
public async IAsyncEnumerable<T> WithRetry<T>(
    Func<CancellationToken, IAsyncEnumerable<T>> source,
    int maxRetries = 3,
    [EnumeratorCancellation] CancellationToken ct = default)
{
    int attempts = 0;
    while (attempts < maxRetries)
    {
        try
        {
            await foreach (var item in source(ct))
            {
                yield return item;
            }
            yield break;  // Success
        }
        catch (Exception) when (attempts < maxRetries - 1)
        {
            attempts++;
            await Task.Delay(TimeSpan.FromSeconds(Math.Pow(2, attempts)), ct);
        }
    }
}
```

## Differences: IEnumerable vs IAsyncEnumerable
| Feature | `IEnumerable<T>` | `IAsyncEnumerable<T>` |
|---------|-----------------|---------------------|
| Execution | Synchronous | Asynchronous |
| Return type | `IEnumerator<T>` | `IAsyncEnumerator<T>` |
| foreach | `foreach` | `await foreach` |
| Disposal | `IDisposable` | `IAsyncDisposable` |
| Cancellation | Manual | `CancellationToken` |
| Use case | In-memory data | I/O-bound streaming |
