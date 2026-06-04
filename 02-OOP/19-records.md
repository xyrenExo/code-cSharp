# Records

## Introduction (C# 9+)
Records are reference types (or value types with record struct) with built-in value equality, immutability, and non-destructive mutation.

## Record Declaration

### Positional Records
```csharp
// Simplest form - primary constructor creates init-only properties
public record Person(string FirstName, string LastName);

// Equivalent expanded form:
public record Person
{
    public string FirstName { get; init; }
    public string LastName { get; init; }

    public Person(string firstName, string lastName)
    {
        FirstName = firstName;
        LastName = lastName;
    }

    public void Deconstruct(out string firstName, out string lastName)
    {
        firstName = FirstName;
        lastName = LastName;
    }
}
```

### Nominal Records
```csharp
public record Employee
{
    public string Name { get; init; }
    public string Department { get; set; }  // Mutable property
    public decimal Salary { get; init; }
}
```

## Key Features

### Value Equality
```csharp
var p1 = new Person("Alice", "Smith");
var p2 = new Person("Alice", "Smith");

Console.WriteLine(p1 == p2);          // true (value equality)
Console.WriteLine(p1.Equals(p2));     // true
Console.WriteLine(p1.GetHashCode() == p2.GetHashCode()); // true
Console.WriteLine(ReferenceEquals(p1, p2)); // false

// Classes: reference equality by default
```

### Non-Destructive Mutation (with expression)
```csharp
var original = new Person("Alice", "Smith");
var modified = original with { LastName = "Johnson" };

Console.WriteLine(original);  // Person { FirstName = Alice, LastName = Smith }
Console.WriteLine(modified);  // Person { FirstName = Alice, LastName = Johnson }
```

### Deconstruction
```csharp
var person = new Person("Bob", "Brown");
var (first, last) = person;

Console.WriteLine(first);  // Bob
Console.WriteLine(last);   // Brown
```

### ToString Override
```csharp
var person = new Person("Charlie", "Davis");
Console.WriteLine(person);
// Output: Person { FirstName = Charlie, LastName = Davis }
```

## Record Inheritance
```csharp
public abstract record Animal(string Name);

public record Dog(string Name, string Breed) : Animal(Name);

public record Cat(string Name, bool IsIndoor) : Animal(Name);

// Usage
var dog = new Dog("Rex", "German Shepherd");
var cat = new Cat("Whiskers", true);

Console.WriteLine(dog);
// Output: Dog { Name = Rex, Breed = German Shepherd }

// Value equality works with inheritance
var dog2 = new Dog("Rex", "German Shepherd");
Console.WriteLine(dog == dog2);  // true
```

## Record Struct (C# 10+)
```csharp
// Value-type record
public readonly record struct Point(double X, double Y);

// Mutable record struct
public record struct Measurement(double Value)
{
    public string Unit { get; set; }  // Mutable
}

var p1 = new Point(1.0, 2.0);
var p2 = new Point(1.0, 2.0);
Console.WriteLine(p1 == p2);  // true (value equality)

var m = new Measurement(10) { Unit = "kg" };
m.Unit = "lbs";  // OK - mutable property
```

## Records with Validation
```csharp
public record Product
{
    private string _name = string.Empty;

    public required string Name
    {
        get => _name;
        init => _name = value ?? throw new ArgumentNullException(nameof(value));
    }

    public required decimal Price { get; init; }

    // Validation in constructor
    public Product
    {
        if (Price < 0)
            throw new ArgumentException("Price cannot be negative");
    }
}
```

## Records vs Classes vs Structs

| Feature | Record (class) | Record Struct | Class | Struct |
|---------|---------------|---------------|-------|--------|
| Type | Reference | Value | Reference | Value |
| Equality | Value | Value | Reference | Value |
| Immutability | Yes (init) | Configurable | Configurable | Configurable |
| `with` expression | Yes | Yes | Manual | Manual |
| Inheritance | Yes | No | Yes | No |
| Deconstruction | Yes | Yes | Manual | Manual |
| `ToString` override | Auto | Auto | Manual | Manual |

## When to Use Records

### Good For
```csharp
// DTOs and API responses
public record ApiResponse<T>(bool Success, T Data, string? Error);

// Value objects
public record Money(decimal Amount, string Currency);

// Command/Query objects (CQRS)
public record CreateOrderCommand(
    string CustomerId,
    List<OrderItemDto> Items,
    ShippingAddress Address
);

// Immutable configuration
public record AppConfig(
    string DatabaseConnectionString,
    string RedisEndpoint,
    int MaxRetries
);
```

### Not Ideal For
- Objects requiring reference identity (entities in EF Core)
- Objects with complex behavior/encapsulation
- Large objects that change frequently
- Service classes (use regular classes)

## Sealed Records
```csharp
// Sealed record - cannot be inherited
public sealed record Configuration(string Key, string Value);
```
