# Encapsulation and Properties

## Encapsulation Principle
Encapsulation hides internal state and requires all interaction to be through well-defined interfaces (methods/properties).

## Fields (Private Data)
```csharp
public class BankAccount
{
    // Private fields - internal state
    private string _accountNumber;
    private decimal _balance;
    private readonly string _accountType;  // Only set in constructor
    private static int _totalAccounts;
    private const decimal MinimumBalance = 100m;
}
```

## Properties
### Auto-Implemented Properties
```csharp
public class Customer
{
    // Compiler generates backing field
    public string FirstName { get; set; }           // Read/write
    public string LastName { get; init; }            // Init-only (C# 9+)
    public string MiddleName { get; }                // Read-only
    public string FullName => $"{FirstName} {LastName}";  // Computed (expression-bodied)
}
```

### Full Property with Backing Field
```csharp
private decimal _price;

public decimal Price
{
    get => _price;
    set
    {
        if (value < 0)
            throw new ArgumentException("Price cannot be negative");
        _price = value;
    }
}
```

### Property Access Modifiers
```csharp
public class Employee
{
    public string Name { get; private set; }     // Public get, private set
    public int Salary { get; protected set; }     // Public get, protected set
    internal string Department { get; set; }      // Internal access
    public string SSN { get; private init; }      // Private init (C# 9+)
}
```

### Computed Properties
```csharp
public class Order
{
    public List<OrderItem> Items { get; set; } = new();
    public decimal TaxRate { get; set; } = 0.08m;

    // Computed (no backing field)
    public decimal Subtotal => Items.Sum(i => i.Price * i.Quantity);

    public decimal Tax => Subtotal * TaxRate;

    public decimal Total => Subtotal + Tax;

    // Expression-bodied property
    public int ItemCount => Items.Count;
}
```

## Init-Only Properties (C# 9+)
```csharp
public class Product
{
    public string Name { get; init; }
    public decimal Price { get; init; }

    // Property validation in init accessor
    private readonly int _id;
    public int Id
    {
        get => _id;
        init
        {
            if (value <= 0)
                throw new ArgumentException("Id must be positive");
            _id = value;
        }
    }
}

// Can only set during initialization
var product = new Product
{
    Name = "Laptop",
    Price = 999.99m,
    Id = 1
};
// product.Name = "New";  // Error! Init-only
```

## Required Properties (C# 11)
```csharp
public class User
{
    public required string Username { get; set; }
    public required string Email { get; set; }
    public string? DisplayName { get; set; }
}

// Must set required properties
var user = new User
{
    Username = "alice",
    Email = "alice@example.com"
};
// Error if Username or Email omitted
```

## Field Keyword (C# 12 Preview)
```csharp
// Future C# feature - accessing backing field in property
public class Demo
{
    public string Name
    {
        get => field;
        init => field = value ?? throw new ArgumentNullException();
    }
}
```

## Indexers
```csharp
public class ShoppingCart
{
    private List<Product> _products = new();

    public Product this[int index]
    {
        get => _products[index];
        set => _products[index] = value;
    }

    public Product? this[string name]
    {
        get => _products.FirstOrDefault(p => p.Name == name);
    }

    public int Count => _products.Count;
}

var cart = new ShoppingCart();
var item = cart[0];       // Uses int indexer
var found = cart["Laptop"];  // Uses string indexer
```

## Records (Immutable Data)
```csharp
public record Person(string FirstName, string LastName);

// Equivalent to:
public class Person : IEquatable<Person>
{
    public string FirstName { get; init; }
    public string LastName { get; init; }
    // Plus value equality, deconstruction, etc.
}

public record Employee(string Name, string Department) : Person(Name, "");
```

## Access Modifier Guidelines
| Situation | Recommendation |
|-----------|---------------|
| Internal state | `private` |
| Internal state with inheritance | `protected` |
| Internal to assembly | `internal` |
| Public API | `public` with properties |
| Immutable after creation | `init` accessors |
| Required construction | `required` modifier |
| Constant values | `const` or `static readonly` |
