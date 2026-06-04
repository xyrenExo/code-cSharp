# Indexers

## Basic Indexer
Indexers allow objects to be indexed like arrays.

```csharp
public class StringCollection
{
    private List<string> _items = new();

    // Indexer definition
    public string this[int index]
    {
        get => _items[index];
        set => _items[index] = value;
    }

    public void Add(string item) => _items.Add(item);
    public int Count => _items.Count;
}

// Usage
var collection = new StringCollection();
collection.Add("First");
collection.Add("Second");
Console.WriteLine(collection[0]);  // "First"
collection[1] = "Modified";
```

## Multiple Parameters
```csharp
public class Matrix
{
    private double[,] _data;

    public Matrix(int rows, int cols)
    {
        _data = new double[rows, cols];
    }

    // Indexer with multiple parameters
    public double this[int row, int col]
    {
        get => _data[row, col];
        set => _data[row, col] = value;
    }

    public int Rows => _data.GetLength(0);
    public int Columns => _data.GetLength(1);
}

// Usage
var matrix = new Matrix(3, 3);
matrix[0, 0] = 1.0;
matrix[1, 2] = 5.0;
double value = matrix[0, 0];  // 1.0
```

## Read-Only and Write-Only Indexers
```csharp
public class ReadOnlyList<T>
{
    private readonly T[] _items;

    public ReadOnlyList(T[] items) => _items = items;

    // Read-only indexer
    public T this[int index] => _items[index];

    public int Length => _items.Length;
}

public class WriteOnlyBuffer
{
    private byte[] _buffer = new byte[1024];

    // Write-only indexer (rare)
    public byte this[int index]
    {
        set => _buffer[index] = value;
    }
}
```

## String-Based Indexers
```csharp
public class Configuration
{
    private Dictionary<string, string> _settings = new();

    public string this[string key]
    {
        get => _settings.TryGetValue(key, out var value) ? value : null;
        set => _settings[key] = value;
    }

    // Overloaded with different parameter type
    public string this[string section, string key]
    {
        get => this[$"{section}:{key}"];
        set => this[$"{section}:{key}"] = value;
    }
}

// Usage
var config = new Configuration();
config["database:connection"] = "Server=localhost;";
config["app", "name"] = "MyApp";
Console.WriteLine(config["app:name"]);
```

## Indexer Overloading
```csharp
public class MultiIndexCollection
{
    private List<string> _items = new();

    // By index
    public string this[int index]
    {
        get => _items[index];
        set => _items[index] = value;
    }

    // By value (find first match)
    public int this[string value]
    {
        get => _items.IndexOf(value);
    }

    // By range
    public IEnumerable<string> this[Range range]  // C# 8+
    {
        get => _items[range];
    }
}
```

## Generic Indexers
```csharp
public class Repository<T>
{
    private readonly Dictionary<Guid, T> _items = new();

    public T this[Guid id]
    {
        get => _items[id];
        set => _items[id] = value;
    }

    public T this[string key]
    {
        get => this[Guid.Parse(key)];
        set => this[Guid.Parse(key)] = value;
    }
}
```

## Interface Indexers
```csharp
public interface IIndexed<TKey, TValue>
{
    TValue this[TKey key] { get; set; }
}

public class MyDictionary<TKey, TValue> : IIndexed<TKey, TValue>
{
    private Dictionary<TKey, TValue> _data = new();

    public TValue this[TKey key]
    {
        get => _data[key];
        set => _data[key] = value;
    }
}
```

## Virtual Indexers
```csharp
public abstract class BaseCollection
{
    public abstract object this[int index] { get; set; }
}

public class IntCollection : BaseCollection
{
    private int[] _items = new int[10];

    public override object this[int index]
    {
        get => _items[index];
        set => _items[index] = (int)value;
    }
}
```

## Real-World Examples

### HTTP Headers
```csharp
public class HttpHeaders
{
    private Dictionary<string, string> _headers = new(StringComparer.OrdinalIgnoreCase);

    public string this[string name]
    {
        get => _headers.TryGetValue(name, out var value) ? value : null;
        set => _headers[name] = value;
    }
}
```

### Data Record
```csharp
public class DataRecord
{
    private Dictionary<string, object?> _fields = new();

    public object? this[string fieldName]
    {
        get => _fields.GetValueOrDefault(fieldName);
        set => _fields[fieldName] = value;
    }

    public T? GetValue<T>(string fieldName)
    {
        var value = this[fieldName];
        return value is T typed ? typed : default;
    }
}
```

### Bit Array
```csharp
public class BitArray
{
    private byte[] _bits;

    public BitArray(int length)
    {
        _bits = new byte[(length + 7) / 8];
        Length = length;
    }

    public int Length { get; }

    public bool this[int index]
    {
        get => (_bits[index / 8] & (1 << (index % 8))) != 0;
        set
        {
            if (value)
                _bits[index / 8] |= (byte)(1 << (index % 8));
            else
                _bits[index / 8] &= (byte)~(1 << (index % 8));
        }
    }
}

var bits = new BitArray(100);
bits[5] = true;
Console.WriteLine(bits[5]);  // True
```

## Indexers vs Properties
| Feature | Property | Indexer |
|---------|----------|---------|
| Name | Named | `this` |
| Parameters | None | One or more |
| Overloading | By name | By parameter types |
| Static | Yes | No |
| Auto-implemented | Yes | No |
| Expression-bodied | Yes | C# 7+ |
