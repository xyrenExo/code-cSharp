# Static Members and Classes

## Static Fields
```csharp
public class Counter
{
    // Instance field
    private int _instanceCount;

    // Static field - shared across all instances
    private static int _totalCount;

    public Counter()
    {
        _instanceCount++;
        _totalCount++;
    }

    public void ShowCounts()
    {
        Console.WriteLine($"Instance: {_instanceCount}, Total: {_totalCount}");
    }

    // Static method to access static field
    public static int GetTotalCount() => _totalCount;
}
```

## Static Methods
```csharp
public class MathHelper
{
    public static double CalculateCircleArea(double radius)
    {
        return Math.PI * radius * radius;
    }

    public static bool IsEven(int number)
    {
        return number % 2 == 0;
    }

    // Cannot access instance members
    // public void InstanceMethod() => StaticField++;  // OK
    // public static void StaticMethod() => InstanceField++;  // ERROR!
}
```

## Static Classes
```csharp
public static class StringHelper
{
    // All members must be static
    public static string Reverse(string input)
    {
        char[] chars = input.ToCharArray();
        Array.Reverse(chars);
        return new string(chars);
    }

    public static bool IsPalindrome(string input)
    {
        var reversed = Reverse(input);
        return string.Equals(input, reversed, StringComparison.OrdinalIgnoreCase);
    }

    // Cannot have instance constructors
    // Cannot be inherited
    // Cannot be instantiated
}
```

## Static Constructor
```csharp
public class Configuration
{
    public static string ConnectionString { get; }
    public static string AppName { get; }
    public static DateTime LoadedAt { get; }

    // Static constructor - called once, before any static member is accessed
    static Configuration()
    {
        ConnectionString = Environment.GetEnvironmentVariable("CONN_STR") ?? "default";
        AppName = "MyApplication";
        LoadedAt = DateTime.Now;

        Console.WriteLine("Configuration initialized");
    }
}
```

## Static Properties
```csharp
public class AppSettings
{
    private static string _basePath = AppDomain.CurrentDomain.BaseDirectory;

    public static string BasePath
    {
        get => _basePath;
        set => _basePath = value ?? throw new ArgumentNullException(nameof(value));
    }

    public static string LogPath => Path.Combine(BasePath, "logs");
    public static string DataPath => Path.Combine(BasePath, "data");

    // Static readonly property
    public static string Version { get; } = "1.0.0";
}
```

## Static Initialization Order
```csharp
public class StaticDemo
{
    // Field initializers execute first
    public static int A = 1;
    public static int B = A + 1;  // 2

    // Static constructor executes after field initializers
    static StaticDemo()
    {
        C = A + B;  // Uses field values
    }

    public static int C;

    // But this is wrong - order matters!
    public static int D = E;    // E is still null!
    public static string E = "Initialized";
}
```

## Extension Methods (Static Methods in Static Classes)
```csharp
public static class StringExtensions
{
    public static bool IsNullOrWhitespace(this string? value)
    {
        return string.IsNullOrWhiteSpace(value);
    }

    public static string Truncate(this string value, int maxLength)
    {
        return value?.Length <= maxLength ? value : value?[..maxLength];
    }
}

// Usage
string text = "Hello World";
bool empty = text.IsNullOrWhitespace();
string truncated = text.Truncate(5);  // "Hello"
```

## Thread Safety with Static Members
```csharp
public class ThreadSafeCounter
{
    private static int _count;
    private static readonly object _lock = new();

    public static int Increment()
    {
        lock (_lock)
        {
            return ++_count;
        }
    }

    // Thread-safe with Interlocked
    private static int _atomicCount;
    public static int AtomicIncrement()
    {
        return Interlocked.Increment(ref _atomicCount);
    }

    // Thread-safe with Lazy<T>
    private static readonly Lazy<ExpensiveResource> _resource =
        new(() => new ExpensiveResource(), LazyThreadSafetyMode.ExecutionAndPublication);

    public static ExpensiveResource Resource => _resource.Value;
}
```

## Static Local Functions (C# 8+)
```csharp
public class Demo
{
    public void Process()
    {
        int localVar = 10;

        // Static local function - cannot capture local variables
        static int Double(int x) => x * 2;

        Console.WriteLine(Double(localVar));  // Pass explicitly
    }
}
```

## When to Use Static
| Scenario | Static? |
|----------|---------|
| Utility/helper methods | Yes |
| Extension methods | Yes (required) |
| Factory methods | Yes |
| Singleton pattern | Yes |
| Constants | Yes |
| Application-wide config | Yes |
| Stateful service | No (use DI) |
| Instance-specific data | No |
| Polymorphic behavior | No |

## Common Static Types in .NET
```csharp
// Utility classes
Math.Max(10, 20);
Path.Combine("dir", "file.txt");
File.ReadAllText("file.txt");
Convert.ToInt32("123");
string.IsNullOrEmpty(text);
Array.Sort(array);
Console.WriteLine("Hello");
```
