# Extension Methods

## Basic Syntax
Extension methods allow adding methods to existing types without modifying them, creating derived types, or recompiling.

```csharp
// Must be in a static class
public static class StringExtensions
{
    // First parameter with 'this' keyword is the extended type
    public static bool IsNullOrWhitespace(this string? value)
    {
        return string.IsNullOrWhiteSpace(value);
    }

    public static string Truncate(this string value, int maxLength)
    {
        return value?.Length <= maxLength ? value : value?[..maxLength];
    }

    public static string Reverse(this string value)
    {
        char[] chars = value.ToCharArray();
        Array.Reverse(chars);
        return new string(chars);
    }
}

// Usage
string text = "Hello World";
bool empty = text.IsNullOrWhitespace();  // Called like instance method!
string truncated = text.Truncate(5);
string reversed = text.Reverse();
```

## Extension Methods on Interfaces
```csharp
public static class EnumerableExtensions
{
    public static void ForEach<T>(this IEnumerable<T> source, Action<T> action)
    {
        foreach (var item in source)
            action(item);
    }

    public static string JoinString<T>(this IEnumerable<T> source, string separator)
    {
        return string.Join(separator, source);
    }

    public static bool IsNullOrEmpty<T>(this IEnumerable<T>? source)
    {
        return source is null || !source.Any();
    }
}

// Usage
var numbers = new[] { 1, 2, 3, 4, 5 };
numbers.ForEach(n => Console.WriteLine(n));
var csv = numbers.JoinString(", ");  // "1, 2, 3, 4, 5"
```

## Generic Extension Methods
```csharp
public static class GenericExtensions
{
    public static T? NullIfDefault<T>(this T value) where T : struct
    {
        return EqualityComparer<T>.Default.Equals(value, default) ? null : value;
    }

    public static T Tap<T>(this T source, Action<T> action)
    {
        action(source);
        return source;
    }

    public static T2 Pipe<T1, T2>(this T1 source, Func<T1, T2> func)
    {
        return func(source);
    }
}

// Usage
int? maybeNull = 0.NullIfDefault();  // null
int? notNull = 5.NullIfDefault();    // 5

var result = new List<int> { 3, 1, 2 }
    .Tap(list => list.Sort())
    .Pipe(list => list.JoinString(","));
// "1,2,3"
```

## Extension Methods on Specific Types
```csharp
public static class IntExtensions
{
    public static bool IsEven(this int value) => value % 2 == 0;
    public static bool IsOdd(this int value) => value % 2 != 0;
    public static int Square(this int value) => value * value;
}

public static class DateTimeExtensions
{
    public static bool IsWeekend(this DateTime date)
        => date.DayOfWeek is DayOfWeek.Saturday or DayOfWeek.Sunday;

    public static DateTime StartOfMonth(this DateTime date)
        => new(date.Year, date.Month, 1);

    public static DateTime EndOfMonth(this DateTime date)
        => date.StartOfMonth().AddMonths(1).AddDays(-1);
}

// Usage
int x = 42;
bool even = x.IsEven();
int square = 5.Square();

DateTime today = DateTime.Now;
bool weekend = today.IsWeekend();
DateTime monthStart = today.StartOfMonth();
```

## Method Resolution and Priority
```csharp
public class MyClass
{
    public void Method() => Console.WriteLine("Instance");
}

public static class MyExtensions
{
    public static void Method(this MyClass obj)
        => Console.WriteLine("Extension");  // Never called for instance
}

// Instance method always wins over extension method
new MyClass().Method();  // "Instance"

// Extension method called when instance method resolution fails
string text = "hello";
var reversed = text.Reverse();  // Extension method (no instance Reverse())
```

## Chaining Extension Methods
```csharp
public static class StringChainExtensions
{
    public static string WithPrefix(this string value, string prefix)
        => $"{prefix}{value}";

    public static string WithSuffix(this string value, string suffix)
        => $"{value}{suffix}";

    public static string ToCsv(this IEnumerable<string> values)
        => string.Join(", ", values);
}

// Chaining
string result = "world"
    .WithPrefix("Hello, ")
    .WithSuffix("!")
    .ToUpper();

// LINQ is the most common example of chaining
var query = numbers
    .Where(n => n > 5)
    .OrderBy(n => n)
    .Select(n => n * 2)
    .ToList();
```

## Common Use Cases

### Fluent Configuration
```csharp
public static class ConfigurationExtensions
{
    public static IServiceCollection AddMyServices(this IServiceCollection services)
    {
        services.AddScoped<IUserService, UserService>();
        services.AddScoped<IOrderService, OrderService>();
        return services;
    }

    public static IApplicationBuilder UseCustomMiddleware(this IApplicationBuilder app)
    {
        app.UseMiddleware<RequestLoggingMiddleware>();
        app.UseMiddleware<ErrorHandlingMiddleware>();
        return app;
    }
}
```

### Null-conditional Chaining
```csharp
public static class ObjectExtensions
{
    public static TOut? Map<TIn, TOut>(this TIn? source, Func<TIn, TOut> mapper)
        where TIn : class
        where TOut : class
    {
        return source is not null ? mapper(source) : null;
    }
}

// Usage
string? name = GetName();
int? length = name.Map(n => n.Length);
```

## Important Rules
1. Extension methods must be in a non-nested, non-generic static class
2. The first parameter with `this` specifies the extended type
3. Extension methods are in scope when the namespace is imported with `using`
4. Instance methods always take priority over extension methods
5. Extension methods cannot access private members of the extended type
