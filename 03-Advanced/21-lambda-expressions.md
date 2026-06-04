# Lambda Expressions

## Lambda Basics

### Syntax
```csharp
// Expression lambda: (parameters) => expression
Func<int, int> square = x => x * x;

// Statement lambda: (parameters) => { statements }
Func<int, int> square = x =>
{
    Console.WriteLine($"Squaring {x}");
    return x * x;
};
```

### Parameter Variations
```csharp
// No parameters
Action greet = () => Console.WriteLine("Hello");

// One parameter (parentheses optional)
Func<int, int> doubleIt = x => x * 2;

// Multiple parameters
Func<int, int, int> add = (a, b) => a + b;

// Explicit types
Func<int, int, int> sum = (int a, int b) => a + b;

// Discard parameters (C# 9+)
Func<int, int, int> ignoreFirst = (_, b) => b;
```

## Lambdas with Collections

### LINQ Method Syntax
```csharp
var numbers = new List<int> { 1, 2, 3, 4, 5, 6 };

var evens = numbers.Where(n => n % 2 == 0);
var squared = numbers.Select(n => n * n);
var sum = numbers.Aggregate((acc, n) => acc + n);
var sorted = numbers.OrderByDescending(n => n);
var grouped = numbers.GroupBy(n => n % 2 == 0 ? "Even" : "Odd");
```

### Complex Lambdas
```csharp
var people = new List<Person>
{
    new("Alice", 30, "Engineering"),
    new("Bob", 25, "Marketing"),
    new("Charlie", 35, "Engineering")
};

// Multiple operations
var result = people
    .Where(p => p.Age > 25)
    .OrderBy(p => p.Name)
    .Select(p => new { p.Name, p.Department })
    .ToList();

// Grouping
var byDepartment = people
    .GroupBy(p => p.Department)
    .Select(g => new
    {
        Department = g.Key,
        Count = g.Count(),
        AverageAge = g.Average(p => p.Age)
    });
```

## Capturing Variables (Closures)
```csharp
int factor = 3;
Func<int, int> multiplier = x => x * factor;  // Captures 'factor'

Console.WriteLine(multiplier(5));  // 15

factor = 10;
Console.WriteLine(multiplier(5));  // 50 (captured variable, not value!)
```

### Capture Pitfalls
```csharp
// Classic foreach capture bug (pre C# 5)
var actions = new List<Action>();
for (int i = 0; i < 5; i++)
{
    actions.Add(() => Console.WriteLine(i));
}
foreach (var action in actions)
    action();
// Output: 5, 5, 5, 5, 5 (all capture same variable!)

// Fix: copy to local
for (int i = 0; i < 5; i++)
{
    int copy = i;
    actions.Add(() => Console.WriteLine(copy));
}
// Output: 0, 1, 2, 3, 4
```

## Async Lambdas
```csharp
// Async lambda
Func<Task> asyncAction = async () =>
{
    await Task.Delay(1000);
    Console.WriteLine("Done");
};

// Async with parameter
Func<int, Task<int>> asyncFunc = async x =>
{
    await Task.Delay(100);
    return x * 2;
};

// Event handler with async void
button.Clicked += async (sender, e) =>
{
    await Task.Delay(100);
    Console.WriteLine("Async handler");
};
```

## Expression-Bodied Members
```csharp
public class Person
{
    // Constructor
    public Person(string name) => Name = name;

    // Property
    public string Name { get; }

    // Method
    public string Greet() => $"Hello, {Name}!";

    // Read-only property (computed)
    public int NameLength => Name.Length;

    // Indexer
    public char this[int i] => Name[i];

    // Finalizer
    ~Person() => Console.WriteLine("Finalized");
}
```

## Local Functions vs Lambdas
```csharp
public void Process(int[] numbers)
{
    // Local function (named, can be recursive, generic)
    int Double(int x) => x * 2;  // Static (C# 8+ captures nothing)

    static bool IsEven(int x) => x % 2 == 0;

    // Lambda (anonymous)
    Func<int, int> doubleLambda = x => x * 2;

    // Local functions are more efficient (no delegate allocation)
    foreach (var n in numbers)
    {
        if (IsEven(n))
            Console.WriteLine(Double(n));
    }
}
```

## Lambda in Expression Trees
```csharp
using System.Linq.Expressions;

// Expression tree (not delegate)
Expression<Func<int, int>> exprTree = x => x * x + 1;

// Can analyze and compile
var compiled = exprTree.Compile();
Console.WriteLine(compiled(5));  // 26

// Expression trees are used by EF Core, etc.
Expression<Func<Person, bool>> predicate = p => p.Age > 18;
```

## Performance Considerations
```csharp
// Cache lambdas to avoid allocations
public class Processor
{
    private static readonly Func<int, int> _square = x => x * x;

    public void Process()
    {
        // Reuses cached delegate
        var result = _square(5);
    }
}
```

## Lambda for Capturing Context
```csharp
// Inline lambda for deferred execution
public IEnumerable<int> GetFiltered(Func<int, bool> predicate)
{
    return GetAllNumbers().Where(predicate);
}

// Closure for configuration
Func<string> CreateGreeter(string greeting)
{
    return () => $"{greeting}, World!";
}

var greet = CreateGreeter("Hello");
Console.WriteLine(greet());  // "Hello, World!"
```
