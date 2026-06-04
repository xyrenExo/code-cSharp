# Classes and Objects

## Class Definition
```csharp
public class Person
{
    // Fields (data)
    private string _name;
    private int _age;

    // Constructor
    public Person(string name, int age)
    {
        _name = name;
        _age = age;
    }

    // Properties
    public string Name
    {
        get => _name;
        set => _name = value ?? throw new ArgumentNullException(nameof(value));
    }

    public int Age
    {
        get => _age;
        set => _age = value >= 0 ? value : throw new ArgumentException("Age must be positive");
    }

    // Auto-property (compiler generates backing field)
    public string Email { get; set; }

    // Read-only property
    public bool IsAdult => Age >= 18;

    // Methods
    public void Introduce()
    {
        Console.WriteLine($"Hi, I'm {Name} and I'm {Age} years old.");
    }

    // Method with return value
    public string GetGreeting() => $"Hello, I'm {Name}!";

    // Static method
    public static Person CreateChild(string name) => new Person(name, 0);
}
```

## Object Instantiation
```csharp
// Default constructor (if no other constructors defined)
var person1 = new Person();

// Parameterized constructor
var person2 = new Person("Alice", 30);

// Object initializer
var person3 = new Person
{
    Name = "Bob",
    Age = 25,
    Email = "bob@example.com"
};

// Object initializer with constructor
var person4 = new Person("Charlie", 35)
{
    Email = "charlie@example.com"
};

// Target-typed new (C# 9+)
Person person5 = new("Diana", 28);
```

## Constructors

### Constructor Overloading
```csharp
public class Product
{
    public string Name { get; set; }
    public decimal Price { get; set; }
    public string Category { get; set; }

    // Primary constructor (C# 12)
    public class Product(string name, decimal price) 
    {
        public string Name { get; set; } = name;
        public decimal Price { get; set; } = price;
    }

    // Chaining constructors
    public Product() : this("Unknown", 0) { }

    public Product(string name) : this(name, 0) { }

    public Product(string name, decimal price)
    {
        Name = name;
        Price = price;
        Category = "General";
    }
}
```

### Static Constructor
```csharp
public class Database
{
    public static string ConnectionString { get; }

    // Called once, before any instance is created or static member accessed
    static Database()
    {
        ConnectionString = ConfigurationManager.ConnectionStrings["Default"];
        Console.WriteLine("Static constructor executed");
    }
}
```

### Copy Constructor
```csharp
public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }

    // Copy constructor
    public Person(Person other)
    {
        Name = other.Name;
        Age = other.Age;
    }
}

var original = new Person { Name = "Alice", Age = 30 };
var copy = new Person(original);  // Deep copy
```

## `this` Keyword
```csharp
public class Employee
{
    private string name;

    public Employee(string name)
    {
        this.name = name;  // Disambiguate field from parameter
    }

    public Employee() : this("Unknown") { }  // Call another constructor

    public Employee SetName(string name)
    {
        this.name = name;
        return this;  // Method chaining
    }
}
```

## Access Modifiers
| Modifier | Visibility |
|----------|------------|
| `public` | Any code |
| `private` | Same class only |
| `protected` | Same class + derived classes |
| `internal` | Same assembly |
| `protected internal` | Same assembly OR derived classes |
| `private protected` | Same class OR derived classes in same assembly |

## Partial Classes
```csharp
// File1.cs
public partial class Customer
{
    public string Name { get; set; }
}

// File2.cs
public partial class Customer
{
    public string Email { get; set; }
}

// File3.cs
public partial class Customer
{
    public void Save()
    {
        // Combined into one class at compile time
    }
}
```

## Nested Classes
```csharp
public class Outer
{
    private int _outerValue;

    public class Inner
    {
        public void DoSomething()
        {
            // Can access private members of Outer?
            // No, unless given a reference
        }
    }
}

var inner = new Outer.Inner();
```

## Anonymous Types
```csharp
var person = new { Name = "Alice", Age = 30 };
Console.WriteLine(person.Name);  // Strongly typed

var people = new[]
{
    new { Name = "Bob", Age = 25 },
    new { Name = "Charlie", Age = 35 }
};
```

## Object Lifecycle and Garbage Collection
```csharp
public class Resource : IDisposable
{
    private bool _disposed;

    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);
    }

    protected virtual void Dispose(bool disposing)
    {
        if (_disposed) return;
        if (disposing)
            // Free managed resources
        // Free unmanaged resources
        _disposed = true;
    }

    ~Resource()
    {
        Dispose(false);
    }
}

// Using statement handles Dispose
using var resource = new Resource();
```
