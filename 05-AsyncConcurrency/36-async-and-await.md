# Async and Await

## Introduction
Async/await provides a structured way to write asynchronous code that reads like synchronous code, without blocking threads.

## Basic Async Method

```csharp
// Synchronous version
public string DownloadData(string url)
{
    using var client = new WebClient();
    return client.DownloadString(url);  // Blocks thread!
}

// Asynchronous version
public async Task<string> DownloadDataAsync(string url)
{
    using var client = new HttpClient();
    return await client.GetStringAsync(url);  // Non-blocking
}
```

## Async Method Signatures

### Return Types
```csharp
// Task<T> - returns a value
public async Task<string> GetDataAsync() { /* ... */ }

// Task - no return value (void equivalent)
public async Task SaveDataAsync() { /* ... */ }

// void - only for event handlers
public async void Button_Click(object sender, EventArgs e) { /* ... */ }

// ValueTask<T> - performance optimization (struct)
public async ValueTask<int> GetCountAsync() { /* ... */ }
```

### Async Method Naming Convention
```csharp
public Task<User> GetUserAsync(int id);
public Task SaveChangesAsync();
public ValueTask<int> GetCountAsync();
```

## Await Pattern

### Awaiting Tasks
```csharp
public async Task ProcessAsync()
{
    // Await single operation
    string data = await DownloadDataAsync("https://example.com");

    // Await multiple sequentially
    var user = await GetUserAsync(1);
    var orders = await GetOrdersAsync(user.Id);

    // Await with configuration
    await Task.Delay(100).ConfigureAwait(false);
}
```

### Concurrent Operations
```csharp
public async Task ProcessConcurrentlyAsync()
{
    // Start both tasks
    Task<string> task1 = DownloadDataAsync("url1");
    Task<string> task2 = DownloadDataAsync("url2");

    // Wait for both to complete
    await Task.WhenAll(task1, task2);

    // Or start and await individually
    var result1 = await task1;
    var result2 = await task2;
}
```

## Error Handling

```csharp
public async Task ProcessWithErrorHandlingAsync()
{
    try
    {
        await ProcessAsync();
    }
    catch (HttpRequestException ex)
    {
        Console.WriteLine($"HTTP error: {ex.Message}");
    }
    catch (TaskCanceledException)
    {
        Console.WriteLine("Operation cancelled");
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Unexpected: {ex.Message}");
        throw;  // Re-throw
    }
    finally
    {
        await CleanupAsync();
    }
}
```

## Cancellation

```csharp
public async Task ProcessWithCancellationAsync(CancellationToken ct)
{
    ct.ThrowIfCancellationRequested();

    await Task.Delay(1000, ct);  // Task.Delay supports cancellation

    await DownloadDataAsync(url).WaitAsync(ct);  // Timeout/cancellation

    // Periodic check
    for (int i = 0; i < 100; i++)
    {
        ct.ThrowIfCancellationRequested();
        await ProcessItemAsync(i);
    }
}

// Usage
using var cts = new CancellationTokenSource(TimeSpan.FromSeconds(30));
try
{
    await ProcessWithCancellationAsync(cts.Token);
}
catch (OperationCanceledException)
{
    Console.WriteLine("Cancelled");
}
```

## Async All the Way

### Don't Block on Async
```csharp
// BAD - deadlock risk, blocks thread
public string GetData()
{
    return DownloadDataAsync(url).Result;       // Deadlock!
    return DownloadDataAsync(url).GetAwaiter().GetResult();  // Deadlock risk
}

// GOOD - async all the way
public async Task<string> GetDataAsync()
{
    return await DownloadDataAsync(url);
}
```

## Async in Libraries

### Async with Streams
```csharp
public async Task ProcessFileAsync(string path)
{
    using var stream = new FileStream(path, FileMode.Open);
    byte[] buffer = new byte[4096];
    int bytesRead = await stream.ReadAsync(buffer, 0, buffer.Length);
    // Process buffer
}
```

### Async Disposal (IAsyncDisposable)
```csharp
public class AsyncResource : IAsyncDisposable
{
    public async ValueTask DisposeAsync()
    {
        await CleanupAsync();
        GC.SuppressFinalize(this);
    }
}

// Usage
await using var resource = new AsyncResource();
```

## ConfigureAwait

```csharp
// Library code - don't need context
public async Task<string> GetDataAsync()
{
    var data = await httpClient.GetStringAsync(url)
        .ConfigureAwait(false);  // Don't capture context

    return await ProcessAsync(data)
        .ConfigureAwait(false);
}

// UI code - need context
public async void LoadButton_Click(object sender, EventArgs e)
{
    // Continue on UI thread after await
    var data = await GetDataAsync();  // No ConfigureAwait
    textBox.Text = data;  // Back on UI thread
}
```

## Progress Reporting

```csharp
public async Task ProcessWithProgressAsync(IProgress<int> progress)
{
    for (int i = 0; i <= 100; i += 10)
    {
        await Task.Delay(100);
        progress?.Report(i);
    }
}

// Usage
var progress = new Progress<int>(percent =>
{
    Console.WriteLine($"{percent}% complete");
});
await ProcessWithProgressAsync(progress);
```

## Common Async Patterns

### Retry Pattern
```csharp
public async Task<T> RetryAsync<T>(Func<Task<T>> operation, int maxRetries = 3)
{
    for (int attempt = 1; attempt <= maxRetries; attempt++)
    {
        try
        {
            return await operation();
        }
        catch (Exception) when (attempt < maxRetries)
        {
            await Task.Delay(TimeSpan.FromSeconds(Math.Pow(2, attempt)));
        }
    }
    return await operation(); // Last attempt
}
```

### Timeout Pattern
```csharp
public async Task<T> WithTimeout<T>(Task<T> task, TimeSpan timeout)
{
    var timeoutTask = Task.Delay(timeout);
    var completed = await Task.WhenAny(task, timeoutTask);

    if (completed == timeoutTask)
        throw new TimeoutException();

    return await task;
}
```
