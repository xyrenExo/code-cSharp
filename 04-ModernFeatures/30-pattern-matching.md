# Pattern Matching

## Type Pattern
```csharp
object value = "Hello World";

// C# 7+: is with pattern
if (value is string text)
{
    Console.WriteLine(text.Length);  // text is in scope
}

// C# 9+: not pattern
if (value is not null)
{
    Console.WriteLine("Not null");
}
```

## Constant Pattern
```csharp
string? input = "yes";

if (input is "yes")
    Console.WriteLine("Affirmative");

if (input is not null)
    Console.WriteLine("Has value");

// In switch
string result = input switch
{
    "yes" => "Yes!",
    "no" => "No!",
    _ => "Unknown"
};
```

## Relational Pattern (C# 9+)
```csharp
int score = 85;

string grade = score switch
{
    >= 90 => "A",
    >= 80 => "B",
    >= 70 => "C",
    >= 60 => "D",
    < 60 => "F"
};

// Combined
string category = score switch
{
    >= 90 and <= 100 => "Excellent",
    >= 80 and < 90 => "Good",
    >= 0 and < 80 => "Needs Improvement",
    _ => "Invalid"
};
```

## Logical Pattern (C# 9+)
```csharp
bool isWeekend = day switch
{
    DayOfWeek.Saturday or DayOfWeek.Sunday => true,
    _ => false
};

// and, or, not
int age = 25;
string group = age switch
{
    < 13 => "Child",
    >= 13 and < 20 => "Teen",
    >= 20 and < 65 => "Adult",
    >= 65 => "Senior"
};

// not pattern
if (value is not null)
    Process(value);
```

## Property Pattern
```csharp
public record Address(string Street, string City, string ZipCode);
public record Person(string Name, int Age, Address? Address);

var person = new Person("Alice", 30, new Address("123 Main", "NYC", "10001"));

// Check nested properties
if (person is { Age: >= 18, Address: { City: "NYC" } })
{
    Console.WriteLine("Adult in NYC");
}

// In switch
string location = person switch
{
    { Address: { City: "NYC" } } => "New Yorker",
    { Address: { City: "LA" } } => "Angeleno",
    { Address: null } => "No address",
    _ => "Other"
};

// Nested property patterns
if (person is { Address.City: "NYC", Age: > 21 })
{
    Console.WriteLine("Can drink in NYC");
}

// Extended property pattern (C# 10+)
if (person is Person { Address.City: "NYC", Age: > 18 })
{
    // Explicit type with properties
}
```

## Positional Pattern
```csharp
public record Point(int X, int Y);
public record Segment(Point Start, Point End);

var point = new Point(3, 4);

// Deconstruct and match
string location = point switch
{
    (0, 0) => "Origin",
    (0, _) => "On Y-axis",
    (_, 0) => "On X-axis",
    (var x, var y) when x == y => "On diagonal",
    _ => "Somewhere"
};

// Nested positional
var segment = new Segment(new(0, 0), new(1, 1));
string desc = segment switch
{
    ((0, 0), (1, 1)) => "Unit diagonal",
    ((0, 0), _) => "From origin",
    _ => "Other"
};
```

## List Pattern (C# 11)
```csharp
int[] numbers = { 1, 2, 3, 4, 5 };

// Match beginning and end
if (numbers is [1, 2, .., 5])
    Console.WriteLine("Starts with 1,2 and ends with 5");

// Discard elements
if (numbers is [_, _, _, _, _])
    Console.WriteLine("Has 5 elements");

// Slice pattern
if (numbers is [.., 5])
    Console.WriteLine("Last element is 5");

// In switch
string desc = numbers switch
{
    [] => "Empty",
    [1, 2, ..] => "Starts with 1,2",
    [.., 3, 4, 5] => "Ends with 3,4,5",
    [var first, .. var rest] => $"First: {first}, Rest: {rest.Length}"
};

// Nested patterns
int[][] matrix = [[1, 0], [0, 1]];
if (matrix is [[1, 0], [0, 1]])
    Console.WriteLine("Identity matrix");
```

## var Pattern
```csharp
if (GetValue() is var result)
{
    // result is available, even on null
    Console.WriteLine(result?.ToString() ?? "null");
}

// In switch for temporary variable
string description = point switch
{
    var (x, y) when x == y => $"On diagonal at {x}",
    var (x, y) => $"At ({x}, {y})"
};
```

## Discard Pattern
```csharp
// _ discards the value
switch (obj)
{
    case int _:
        Console.WriteLine("It's an int");
        break;
    case string _:
        Console.WriteLine("It's a string");
        break;
}

// C# 9+ discard pattern in switch expression
string type = obj switch
{
    int _ => "int",
    string _ => "string",
    _ => "other"
};
```

## Switch Expression Advanced
```csharp
public decimal CalculateDiscount(Order order)
    => (order.CustomerType, order.Total) switch
    {
        ("Premium", > 1000) => order.Total * 0.20m,
        ("Premium", _) => order.Total * 0.15m,
        ("Regular", > 500) => order.Total * 0.10m,
        ("Regular", _) => 0,
        (_, _) => 0
    };
```

## Pattern Matching in Catch
```csharp
try
{
    DoSomething();
}
catch (HttpRequestException ex) when (ex.StatusCode is HttpStatusCode.NotFound)
{
    // Handle 404
}
catch (Exception ex) when (ex is InvalidOperationException or ArgumentException)
{
    // Handle specific exceptions
}
```
