# Interfaces

## Interface Definition
```csharp
public interface IPaymentProcessor
{
    // Method signatures
    Task<PaymentResult> ProcessPayment(decimal amount, Currency currency);

    // Properties
    string ProcessorName { get; }

    // Default implementation (C# 8+)
    void Log(string message)
    {
        Console.WriteLine($"[{ProcessorName}] {message}");
    }
}
```

## Implementing Interfaces
```csharp
public class StripeProcessor : IPaymentProcessor
{
    public string ProcessorName => "Stripe";

    public async Task<PaymentResult> ProcessPayment(decimal amount, Currency currency)
    {
        // Stripe-specific implementation
        var charge = await StripeClient.CreateCharge(amount, currency);
        return new PaymentResult(charge.Id, charge.Status);
    }
}

public class PayPalProcessor : IPaymentProcessor
{
    public string ProcessorName => "PayPal";

    public async Task<PaymentResult> ProcessPayment(decimal amount, Currency currency)
    {
        // PayPal-specific implementation
        var payment = await PayPalClient.ExecutePayment(amount, currency);
        return new PaymentResult(payment.Id, payment.Status);
    }
}
```

## Multiple Interface Implementation
```csharp
public interface ISerializable
{
    string Serialize();
}

public interface IValidatable
{
    bool Validate();
}

public class Product : ISerializable, IValidatable
{
    public string Name { get; set; }
    public decimal Price { get; set; }

    public string Serialize() => JsonSerializer.Serialize(this);

    public bool Validate() => !string.IsNullOrEmpty(Name) && Price > 0;
}
```

## Explicit Interface Implementation
```csharp
public interface IFileLogger
{
    void Log(string message);
}

public interface IDatabaseLogger
{
    void Log(string message);
}

public class Logger : IFileLogger, IDatabaseLogger
{
    // Explicit implementation
    void IFileLogger.Log(string message)
    {
        File.AppendAllText("log.txt", message);
    }

    void IDatabaseLogger.Log(string message)
    {
        // Save to database
    }

    // Public method
    public void Log(string message)
    {
        Console.WriteLine(message);
    }
}

// Usage
var logger = new Logger();
logger.Log("Test");           // Console

IFileLogger fileLogger = logger;
fileLogger.Log("Test");       // File

IDatabaseLogger dbLogger = logger;
dbLogger.Log("Test");         // Database
```

## Interface Inheritance
```csharp
public interface IRepository
{
    object GetById(int id);
}

public interface IUserRepository : IRepository
{
    User GetByEmail(string email);
}

public interface IAdminRepository : IUserRepository
{
    void DeleteUser(int id);
}

public class UserRepository : IAdminRepository
{
    public object GetById(int id) => /* ... */;
    public User GetByEmail(string email) => /* ... */;
    public void DeleteUser(int id) => /* ... */;
}
```

## Generic Interfaces
```csharp
public interface ICrudRepository<T, TId>
{
    Task<T> GetById(TId id);
    Task<IEnumerable<T>> GetAll();
    Task<T> Add(T entity);
    Task<T> Update(T entity);
    Task Delete(TId id);
}

public class UserRepository : ICrudRepository<User, int>
{
    public async Task<User> GetById(int id) { /* ... */ }
    public async Task<IEnumerable<User>> GetAll() { /* ... */ }
    public async Task<User> Add(User entity) { /* ... */ }
    public async Task<User> Update(User entity) { /* ... */ }
    public async Task Delete(int id) { /* ... */ }
}
```

## Covariance and Contravariance

### Covariance (out)
```csharp
// Covariant interface - T only in output positions
public interface IProducer<out T>
{
    T Produce();
}

IProducer<Dog> dogProducer = new DogProducer();
IProducer<Animal> animalProducer = dogProducer;  // OK: Dog → Animal
```

### Contravariance (in)
```csharp
// Contravariant interface - T only in input positions
public interface IConsumer<in T>
{
    void Consume(T item);
}

IConsumer<Animal> animalConsumer = new AnimalConsumer();
IConsumer<Dog> dogConsumer = animalConsumer;  // OK: Animal → Dog
```

### Built-in Covariant/Contravariant Interfaces
```csharp
// Covariant
IEnumerable<object> objs = new List<string>();  // string → object
IReadOnlyList<object> readonlyList = new List<string>();

// Contravariant
IComparer<object> objectComparer = new MyComparer();
IComparer<string> stringComparer = objectComparer;  // object → string

Comparison<object> objComparison = (a, b) => 0;
Comparison<string> strComparison = objComparison;
```

## Default Interface Methods (C# 8+)
```csharp
public interface INotification
{
    void Send(string message);

    // Default implementation
    void SendWithLog(string message)
    {
        Console.WriteLine("Sending...");
        Send(message);
        Console.WriteLine("Sent!");
    }

    // Static members in interfaces (C# 11)
    static INotification CreateDefault() => new EmailNotification();
}
```

## IComparable and IEquatable
```csharp
public class Product : IComparable<Product>, IEquatable<Product>
{
    public string Name { get; set; }
    public decimal Price { get; set; }

    public int CompareTo(Product? other)
    {
        if (other is null) return 1;
        return Price.CompareTo(other.Price);
    }

    public bool Equals(Product? other)
    {
        if (other is null) return false;
        return Name == other.Name && Price == other.Price;
    }

    public override bool Equals(object obj) => Equals(obj as Product);
    public override int GetHashCode() => HashCode.Combine(Name, Price);
}
```

## IDisposable Interface
```csharp
public class DatabaseConnection : IDisposable
{
    private SqlConnection _connection;

    public void Dispose()
    {
        _connection?.Dispose();
        GC.SuppressFinalize(this);
    }
}
```

## Common .NET Interfaces
| Interface | Purpose |
|-----------|---------|
| `IDisposable` | Resource cleanup |
| `IComparable<T>` | Sorting/comparison |
| `IEquatable<T>` | Equality comparison |
| `IEnumerable<T>` | Iteration (foreach) |
| `ICollection<T>` | Collection operations |
| `IList<T>` | Indexed list operations |
| `IDictionary<K,V>` | Key-value storage |
| `INotifyPropertyChanged` | Property change notification |
| `ICloneable` | Object cloning |
| `ISerializable` | Custom serialization |
