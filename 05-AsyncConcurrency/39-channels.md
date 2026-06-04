# Channels (System.Threading.Channels)

## Introduction
Channels provide a producer/consumer pattern for passing data between threads or tasks asynchronously.

## Basic Usage

```csharp
using System.Threading.Channels;

// Create a channel
Channel<int> channel = Channel.CreateUnbounded<int>();
// or Channel.CreateBounded<int>(capacity: 100);

// Writer (producer)
ChannelWriter<int> writer = channel.Writer;

// Reader (consumer)
ChannelReader<int> reader = channel.Reader;
```

## Producer Pattern

```csharp
public async Task ProduceAsync(ChannelWriter<int> writer, CancellationToken ct)
{
    try
    {
        for (int i = 0; i < 100; i++)
        {
            await writer.WriteAsync(i, ct);
            Console.WriteLine($"Produced {i}");
        }
    }
    finally
    {
        writer.Complete();  // Signal no more data
    }
}
```

## Consumer Pattern

```csharp
public async Task ConsumeAsync(ChannelReader<int> reader, CancellationToken ct)
{
    await foreach (var item in reader.ReadAllAsync(ct))
    {
        Console.WriteLine($"Consumed {item}");
    }
}
```

## Complete Example

```csharp
var channel = Channel.CreateUnbounded<int>();

var producer = Task.Run(() => ProduceAsync(channel.Writer));
var consumer = Task.Run(() => ConsumeAsync(channel.Reader));

await Task.WhenAll(producer, consumer);
```

## Bounded Channels

```csharp
// Fixed capacity with back-pressure
var options = new BoundedChannelOptions(10)
{
    FullMode = BoundedChannelFullMode.Wait,      // Wait for space
    // FullMode = BoundedChannelFullMode.DropNewest,  // Drop newest
    // FullMode = BoundedChannelFullMode.DropOldest,   // Drop oldest
    // FullMode = BoundedChannelFullMode.DropWrite,    // Drop written item
    SingleWriter = true,
    SingleReader = false
};

var channel = Channel.CreateBounded<int>(options);
```

## Channel Completion

```csharp
// Signal completion
writer.Complete();

// Signal completion with error
writer.Complete(new InvalidOperationException("Something went wrong"));

// Check completion
var completion = reader.Completion;  // Task that completes when writer completes
await reader.Completion;  // Wait for completion

// TryRead behavior after completion
while (reader.TryRead(out var item))
{
    Console.WriteLine(item);
}
```

## Multiple Producers/Consumers

```csharp
var channel = Channel.CreateUnbounded<int>();

// Multiple producers
var producers = Enumerable.Range(0, 3).Select(i =>
    Task.Run(async () =>
    {
        for (int j = 0; j < 10; j++)
        {
            await channel.Writer.WriteAsync(i * 100 + j);
        }
    }));

// Multiple consumers
var consumers = Enumerable.Range(0, 2).Select(_ =>
    Task.Run(async () =>
    {
        await foreach (var item in channel.Reader.ReadAllAsync())
        {
            Console.WriteLine($"Consumer got {item}");
        }
    }));

// Wait for producers, then signal completion
await Task.WhenAll(producers);
channel.Writer.Complete();

// Wait for consumers
await Task.WhenAll(consumers);
```

## Pipeline Pattern

```csharp
// Stage 1: Read lines from file
Channel<string> linesChannel = Channel.CreateUnbounded<string>();
Task producer = Task.Run(async () =>
{
    await foreach (var line in File.ReadLinesAsync("input.txt"))
    {
        await linesChannel.Writer.WriteAsync(line);
    }
    linesChannel.Writer.Complete();
});

// Stage 2: Process lines
Channel<ProcessedData> processedChannel = Channel.CreateUnbounded<ProcessedData>();
Task processor = Task.Run(async () =>
{
    await foreach (var line in linesChannel.Reader.ReadAllAsync())
    {
        var processed = await ProcessLineAsync(line);
        await processedChannel.Writer.WriteAsync(processed);
    }
    processedChannel.Writer.Complete();
});

// Stage 3: Write results
Task consumer = Task.Run(async () =>
{
    await foreach (var data in processedChannel.Reader.ReadAllAsync())
    {
        await WriteResultAsync(data);
    }
});

await Task.WhenAll(producer, processor, consumer);
```

## Batched Processing

```csharp
public async IAsyncEnumerable<List<int>> BatchReader(
    ChannelReader<int> reader, int batchSize,
    [EnumeratorCancellation] CancellationToken ct = default)
{
    var batch = new List<int>(batchSize);

    await foreach (var item in reader.ReadAllAsync(ct))
    {
        batch.Add(item);
        if (batch.Count >= batchSize)
        {
            yield return batch;
            batch = new List<int>(batchSize);
        }
    }

    if (batch.Count > 0)
        yield return batch;
}
```

## Performance Considerations

### Unbounded vs Bounded
```csharp
// Unbounded: No limit, may use unbounded memory
var unbounded = Channel.CreateUnbounded<int>();

// Bounded: Limits memory, creates back-pressure
var bounded = Channel.CreateBounded<int>(new BoundedChannelOptions(100)
{
    FullMode = BoundedChannelFullMode.Wait
});
```

### Single Reader/Writer Optimization
```csharp
var options = new UnboundedChannelOptions
{
    SingleReader = true,   // Optimize for single consumer
    SingleWriter = true,   // Optimize for single producer
    AllowSynchronousContinuations = false
};
```

## Channel vs Other Patterns

| Feature | Channel | BlockingCollection | BufferBlock<T> |
|---------|---------|-------------------|----------------|
| Async | Yes | Limited | Yes |
| Back-pressure | Yes | Yes | Yes |
| Multiple producers | Yes | Yes | Yes |
| Multiple consumers | Yes | Yes | Yes |
| Completion signaling | Yes | Yes | Yes |
| Memory efficiency | High | Medium | Medium |
| API simplicity | High | Medium | Low |
