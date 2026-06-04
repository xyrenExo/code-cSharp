# Nullable Reference Types

## Enabling Nullable Context
```csharp
// Per project (.csproj)
<Nullable>enable</Nullable>

// Per file
#nullable enable

// Disable per file
#nullable disable

// Restore to project default
#nullable restore
```

## Nullable vs Non-Nullable References
```csharp
#nullable enable

// Non-nullable (compiler warns if could be null)
string name = "Alice";
string mustNotBeNull = GetValue();  // Warning if GetValue() can return null

// Nullable (explicitly allows null)
string? maybeNull = null;
string? couldBeNull = GetNullableValue();
```

## Nullable Annotations in APIs
```csharp
public class CustomerService
{
    // Returns non-nullable
    public Customer GetCustomer(int id) { /* ... */ }

    // Returns nullable
    public Customer? FindCustomer(string email) { /* ... */ }

    // Parameters
    public void UpdateCustomer(string name, string? middleName, Customer customer)
    {
        // name: non-nullable
        // middleName: nullable
        // customer: non-nullable
    }
}
```

## Null State Analysis
```csharp
string? nullable = GetValue();

// Dereference warning
Console.WriteLine(nullable.Length);  // Warning!

// Check before use
if (nullable is not null)
    Console.WriteLine(nullable.Length);  // OK

// Null-conditional
int? length = nullable?.Length;  // OK

// Null-forgiving operator
Console.WriteLine(nullable!.Length);  // Suppress warning (use carefully)

// Pattern matching
if (nullable is string s)
    Console.WriteLine(s.Length);  // OK
```

## Null-Forgiving Operator (!)
```csharp
// When you know a value isn't null but compiler disagrees
string name = GetValue()!;  // Suppress warning

// After null check in different scope
string? temp = GetValue();
ProcessWithNonNull(temp!);  // I know it's not null here

// Test scenarios
string? value = MethodThatShouldNotReturnNull();
Assert.NotNull(value);
Console.WriteLine(value!.Length);  // Safe after assertion
```

## Nullable Attributes
```csharp
using System.Diagnostics.CodeAnalysis;

public class Parser
{
    // [NotNullWhen(true)] - if returns true, out param is not null
    public static bool TryParse(string input, [NotNullWhen(true)] out Guid? result)
    {
        if (Guid.TryParse(input, out var guid))
        {
            result = guid;
            return true;
        }
        result = null;
        return false;
    }

    // [MaybeNull] - return might be null even if type says non-null
    [return: MaybeNull]
    public T GetValueOrDefault<T>(string key)
    {
        if (_dictionary.TryGetValue(key, out var value))
            return value;
        return default;  // Could be null for reference types
    }

    // [NotNull] - return is never null
    [return: NotNull]
    public string GetConfig(string key)
    {
        return _config[key] ?? throw new KeyNotFoundException();
    }

    // [DisallowNull] - input should never be null
    public void SetName([DisallowNull] string? name)
    {
        // name can be null in signature, but shouldn't be
        _name = name!;
    }
}
```

## Generic Types and Nullability
```csharp
// Generic with nullable context
public class Container<T>
{
    // T could be string (non-null) or string? (nullable)
    public T? GetDefault()
    {
        return default;
    }
}

// Constraint for non-nullable
public class NonNullContainer<T> where T : notnull
{
    public void Process(T value) { }
}

// MaybeNull attribute for generics
public static class Extensions
{
    [return: MaybeNull]
    public static T FirstOrDefault<T>(this IEnumerable<T> source)
    {
        foreach (var item in source)
            return item;
        return default;
    }
}
```

## Nullable in Inherited Interfaces
```csharp
public interface IRepository<T> where T : class
{
    T? Find(int id);
    T Get(int id);
}

public class Repository<T> : IRepository<T> where T : class
{
    public T? Find(int id) { /* can return null */ }
    public T Get(int id) { /* never returns null */ }
}
```

## Migration from Non-Nullable
```csharp
// Step 1: Enable nullable and address warnings
#nullable enable

// Step 2: Annotate APIs correctly
public Customer? FindCustomer(int id)  // Actually can be null
{
    return _customers.GetValueOrDefault(id);
}

// Step 3: Handle nulls properly
var customer = FindCustomer(1);
if (customer is null)
{
    // Handle missing customer
}
```

## Common Patterns
```csharp
// Null-coalescing assignment
public class Settings
{
    private string? _cachedValue;
    public string GetValue()
    {
        return _cachedValue ??= LoadValue();
    }
}

// Required initialization
public class Model
{
    public string Name { get; set; } = string.Empty;  // No warning
    public string? Description { get; set; }  // Can be null
}
```

## Nullable Warnings
| Warning | Meaning | Fix |
|---------|---------|-----|
| CS8600 | Converting null literal to non-nullable | Change type to nullable |
| CS8602 | Dereference of possibly null reference | Add null check |
| CS8603 | Possible null reference return | Fix return or use nullable |
| CS8604 | Possible null reference argument | Add null check |
| CS8618 | Non-nullable field not initialized | Initialize or make nullable |
| CS8625 | Cannot convert null literal to non-nullable | Use nullable type |
