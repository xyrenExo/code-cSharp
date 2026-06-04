# Generics

## Generic Classes
```csharp
// Generic class with type parameter T
public class Repository<T>
{
    private readonly List<T> _items = new();

    public void Add(T item) => _items.Add(item);

    public T Get(int index) => _items[index];

    public IEnumerable<T> GetAll() => _items;

    public int Count => _items.Count;
}

// Usage
var intRepo = new Repository<int>();
intRepo.Add(42);

var stringRepo = new Repository<string>();
stringRepo.Add("Hello");
```

## Generic Methods
```csharp
public class Utilities
{
    // Generic method
    public static T Max<T>(T a, T b) where T : IComparable<T>
    {
        return a.CompareTo(b) > 0 ? a : b;
    }

    // Multiple type parameters
    public static Dictionary<TKey, TValue> CreateDictionary<TKey, TValue>()
        where TKey : notnull
    {
        return new Dictionary<TKey, TValue>();
    }

    // Generic with type inference
    public static T Create<T>() where T : new()
    {
        return new T();
    }
}

// Type inference
var max = Utilities.Max(5, 10);        // T inferred as int
var maxStr = Utilities.Max("A", "B");  // T inferred as string
```

## Generic Constraints

| Constraint | Description |
|-----------|-------------|
| `where T : class` | Reference type |
| `where T : struct` | Value type (non-nullable) |
| `where T : notnull` | Non-nullable (C# 8+) |
| `where T : new()` | Parameterless constructor |
| `where T : BaseClass` | Inherits from BaseClass |
| `where T : IInterface` | Implements interface |
| `where T : U` | Same as another type parameter |
| `where T : unmanaged` | Unmanaged type (C# 7.3+) |
| `where T : Enum` | Enum type (C# 7.3+) |
| `where T : Delegate` | Delegate type (C# 7.3+) |

### Examples
```csharp
// Multiple constraints
public class EntityService<T>
    where T : class, IEntity, new()
{
    public T Create()
    {
        var entity = new T();
        entity.CreatedAt = DateTime.UtcNow;
        return entity;
    }
}

// Constraint combinations
public class Processor<T>
    where T : class, IComparable<T>, new()
{
    // T must be: reference type + IComparable + parameterless ctor
}

// Enum constraint
public static TEnum ParseEnum<TEnum>(string value)
    where TEnum : struct, Enum
{
    return Enum.Parse<TEnum>(value, ignoreCase: true);
}

// Delegate constraint
public static TDelegate Combine<TDelegate>(TDelegate a, TDelegate b)
    where TDelegate : Delegate
{
    return (TDelegate)Delegate.Combine(a, b);
}
```

## Generic Interfaces
```csharp
// Generic interface
public interface ICrudRepository<T, TId>
{
    Task<T?> GetById(TId id);
    Task<IEnumerable<T>> GetAll();
    Task<T> Add(T entity);
    Task<T> Update(T entity);
    Task Delete(TId id);
}

// Implementation
public class UserRepository : ICrudRepository<User, Guid>
{
    public async Task<User?> GetById(Guid id) { /* ... */ }
    public async Task<IEnumerable<User>> GetAll() { /* ... */ }
    public async Task<User> Add(User entity) { /* ... */ }
    public async Task<User> Update(User entity) { /* ... */ }
    public async Task Delete(Guid id) { /* ... */ }
}
```

## Generic Delegates
```csharp
// Generic delegate definition
public delegate T Transformer<T>(T input);

// Usage
Transformer<int> square = x => x * x;
Transformer<string> addExclamation = s => s + "!";

// Built-in generics
Func<T, TResult>    // Generic function
Action<T>           // Generic action
Predicate<T>        // Generic predicate
```

## Variance in Generics

### Covariance (out)
```csharp
// Covariant: T only in output positions
public interface IReadOnlyCollection<out T>
{
    T Get(int index);
    IEnumerable<T> GetAll();
}

IReadOnlyCollection<string> strings = new StringCollection();
IReadOnlyCollection<object> objects = strings;  // OK: string → object
```

### Contravariance (in)
```csharp
// Contravariant: T only in input positions
public interface IComparer<in T>
{
    int Compare(T x, T y);
}

IComparer<object> objectComparer = new ObjectComparer();
IComparer<string> stringComparer = objectComparer;  // OK: object → string
```

## Generic Static Members
```csharp
public class GenericCounter<T>
{
    // Static per closed generic type
    public static int Count { get; set; }
}

GenericCounter<int>.Count = 10;
GenericCounter<string>.Count = 20;

Console.WriteLine(GenericCounter<int>.Count);     // 10 (separate!)
Console.WriteLine(GenericCounter<string>.Count);  // 20 (separate!)
```

## Type Parameters Naming Conventions
```csharp
// Single letter
T            // Type
TKey, TValue // Key/value pairs

// Descriptive
TEntity      // Entity type
TId          // Identifier type
TRequest     // Request type
TResponse    // Response type

// Prefix with T
TSource, TResult  // Transformation
```

## Open vs Closed Generic Types
```csharp
// Open generic type (not fully specified)
Type openType = typeof(List<>);

// Closed generic type (all type args specified)
Type closedType = typeof(List<int>);

// Creating at runtime
Type closed = openType.MakeGenericType(typeof(string));
var list = (List<string>)Activator.CreateInstance(closed)!;
```

## Generic Math (C# 11+)
```csharp
public static T Add<T>(T left, T right) where T : INumber<T>
{
    return left + right;
}

// Usage
int sum = Add(5, 10);
double sumD = Add(3.14, 2.86);
decimal sumM = Add(19.99m, 0.01m);
```
