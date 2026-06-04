# Spans and Memory

## Introduction
`Span<T>` and `Memory<T>` provide type-safe, memory-safe views over contiguous memory without allocation.

## Span<T>

### Creating Spans
```csharp
// From arrays
int[] array = { 1, 2, 3, 4, 5 };
Span<int> span = array.AsSpan();
Span<int> slice = array.AsSpan(1, 3);  // { 2, 3, 4 }

// From strings
string text = "Hello World";
ReadOnlySpan<char> charSpan = text.AsSpan();
ReadOnlySpan<char> wordSpan = text.AsSpan(0, 5);  // "Hello"

// From stack memory
Span<int> stackSpan = stackalloc int[] { 1, 2, 3, 4, 5 };

// From unmanaged memory
Span<byte> nativeSpan;
unsafe
{
    nativeSpan = new Span<byte>(Pointer, length);
}
```

### Span Operations
```csharp
Span<int> numbers = new[] { 5, 3, 1, 4, 2 }.AsSpan();

// Slicing
Span<int> first = numbers[..2];     // { 5, 3 }
Span<int> last = numbers[^2..];      // { 4, 2 }
Span<int> middle = numbers[1..4];    // { 3, 1, 4 }

// Modification
numbers[0] = 10;                // Modifies original
numbers.Fill(0);                // All zeros
numbers.Clear();                 // Set to default
numbers.CopyTo(otherSpan);      // Copy to another span

// Searching and comparison
int index = numbers.IndexOf(3);
bool contains = numbers.Contains(3);
bool startsWith = numbers.StartsWith(sequence);
bool equals = numbers.SequenceEqual(other);
int[] sorted = numbers.ToArray();
```

### ReadOnlySpan<T>
```csharp
// Preferred for read-only scenarios
public static int CountDigits(ReadOnlySpan<char> text)
{
    int count = 0;
    foreach (char c in text)
    {
        if (char.IsDigit(c))
            count++;
    }
    return count;
}

// Usage - no allocation!
string data = "abc123def456";
int digits = CountDigits(data.AsSpan());
```

## Memory<T>

### Creating Memory
```csharp
// From arrays
Memory<int> memory = new int[] { 1, 2, 3, 4, 5 };
Memory<int> slice = memory.Slice(1, 3);

// From arrays with ownership
using IMemoryOwner<int> owner = MemoryPool<int>.Shared.Rent(100);
Memory<int> pooled = owner.Memory;

// String
ReadOnlyMemory<char> textMemory = "Hello".AsMemory();
```

### Memory vs Span
```csharp
// Span<T> is stack-only (ref struct)
Span<int> span = stackalloc int[5];  // OK - on stack
// Span<int> field;  // ERROR - can't be class field

// Memory<T> is heap-allocatable
Memory<int> memory = new int[5];     // OK - on heap
Memory<int> field;                    // OK - can be class field

// Memory<T> can produce Span<T>
Span<int> span = memory.Span;        // OK for synchronous use

// Memory<T> can be used in async methods
async Task ProcessAsync(Memory<int> memory)
{
    Span<int> span = memory.Span;    // OK: synchronous part
    await Task.Delay(100);
    // Don't use span here (stack may be gone)
}
```

## String Performance with Span
```csharp
// Without Span - allocates substrings
string url = "/api/users/123";
string segment = url.Substring(5, 5);  // Allocates!
string id = url.Substring(11);         // Allocates!

// With Span - no allocation
ReadOnlySpan<char> urlSpan = url.AsSpan();
ReadOnlySpan<char> segment = urlSpan.Slice(5, 5);
ReadOnlySpan<char> idSpan = urlSpan.Slice(11);

// Parse from span
if (int.TryParse(idSpan, out int id))
    Console.WriteLine(id);
```

## Parsing with Spans
```csharp
// Date parsing without allocation
ReadOnlySpan<char> dateStr = "2024-01-15".AsSpan();
if (DateOnly.TryParseExact(dateStr, "yyyy-MM-dd", out var date))
{
    Console.WriteLine(date);
}

// CSV parsing
public static void ParseCsvLine(ReadOnlySpan<char> line)
{
    int start = 0;
    for (int i = 0; i <= line.Length; i++)
    {
        if (i == line.Length || line[i] == ',')
        {
            var field = line[start..i];
            ProcessField(field);
            start = i + 1;
        }
    }
}
```

## MemoryPool<T>
```csharp
// Rent temporary buffers
using var owner = MemoryPool<byte>.Shared.Rent(1024);
Memory<byte> buffer = owner.Memory;

// Use buffer
int bytesRead = await stream.ReadAsync(buffer);
var data = buffer[..bytesRead];
```

## Stackalloc with Span
```csharp
// Stack allocation (fast, no heap)
Span<int> temp = stackalloc int[256];  // Stack limit ~1MB

// Conditional stack/heap
Span<int> buffer = array.Length <= 256
    ? stackalloc int[array.Length]
    : new int[array.Length];
```

## Performance Comparison
```csharp
// Allocation-heavy approach
string ExtractValue(string data)
{
    int start = data.IndexOf(':') + 1;
    return data.Substring(start).Trim();  // 2 allocations
}

// Allocation-free approach
ReadOnlySpan<char> ExtractValueSpan(string data)
{
    int start = data.AsSpan().IndexOf(':') + 1;
    return data.AsSpan()[start..].Trim();  // 0 allocations!
}
```

## Rules and Limitations
- `Span<T>` is a `ref struct` - stack only
- Cannot be boxed (no `object`, `dynamic`, `IEnumerable`)
- Cannot be a field in a class (only ref structs)
- Cannot be used in `async` methods
- Can be used in synchronous methods, iterators, lambdas (limited)
- `Memory<T>` is a regular struct - heap safe
- Use `Memory<T>` for async and class fields
- Convert `Memory<T>.Span` for synchronous operations
