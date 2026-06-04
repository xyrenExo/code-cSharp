# Abstract Classes and Methods

## Abstract Class Definition
```csharp
public abstract class DatabaseProvider
{
    // Abstract property - must be overridden
    public abstract string ConnectionString { get; protected set; }

    // Abstract method - no implementation
    public abstract void Connect();
    public abstract void Disconnect();

    // Virtual method - optional override
    public virtual void Log(string message)
    {
        Console.WriteLine($"[{GetType().Name}] {message}");
    }

    // Concrete method - inherited as-is
    public void ExecuteWithLogging(string command)
    {
        Log($"Executing: {command}");
        Connect();
        ExecuteCommand(command);
        Disconnect();
    }

    // Protected abstract method
    protected abstract void ExecuteCommand(string command);
}
```

## Concrete Implementation
```csharp
public class SqlServerProvider : DatabaseProvider
{
    private SqlConnection _connection;

    public override string ConnectionString { get; protected set; }

    public SqlServerProvider(string connectionString)
    {
        ConnectionString = connectionString;
    }

    public override void Connect()
    {
        _connection = new SqlConnection(ConnectionString);
        _connection.Open();
        Log("Connected to SQL Server");
    }

    public override void Disconnect()
    {
        _connection?.Close();
        _connection?.Dispose();
        Log("Disconnected from SQL Server");
    }

    protected override void ExecuteCommand(string command)
    {
        using var cmd = new SqlCommand(command, _connection);
        cmd.ExecuteNonQuery();
    }
}

public class PostgreSqlProvider : DatabaseProvider
{
    private NpgsqlConnection _connection;

    public override string ConnectionString { get; protected set; }

    public PostgreSqlProvider(string connectionString)
    {
        ConnectionString = connectionString;
    }

    public override void Connect()
    {
        _connection = new NpgsqlConnection(ConnectionString);
        _connection.Open();
        Log("Connected to PostgreSQL");
    }

    public override void Disconnect()
    {
        _connection?.Close();
        Log("Disconnected from PostgreSQL");
    }

    protected override void ExecuteCommand(string command)
    {
        using var cmd = new NpgsqlCommand(command, _connection);
        cmd.ExecuteNonQuery();
    }
}
```

## Abstract vs Virtual Methods
```csharp
public abstract class Shape
{
    // Abstract: must be overridden, no implementation
    public abstract double GetArea();

    // Virtual: can be overridden, has default implementation
    public virtual string GetDescription()
    {
        return $"A shape with area {GetArea()}";
    }

    // Concrete: cannot be overridden
    public void Display()
    {
        Console.WriteLine(GetDescription());
    }
}
```

## Abstract Properties
```csharp
public abstract class Report
{
    public abstract string Title { get; }
    public abstract DateTime GeneratedAt { get; }
    public abstract string Format { get; }

    public void PrintHeader()
    {
        Console.WriteLine($"{Title} - {GeneratedAt}");
    }
}

public class SalesReport : Report
{
    public override string Title => "Sales Report";
    public override DateTime GeneratedAt => DateTime.Now;
    public override string Format => "PDF";
}
```

## Abstract Class vs Interface

| Feature | Abstract Class | Interface |
|---------|---------------|-----------|
| Fields | Yes | No (C# 11: static fields) |
| Constructors | Yes | No |
| Destructors | Yes | No |
| Access modifiers | All | Public (default) |
| Implementation | Partial | None (or default methods) |
| Multiple inheritance | No | Yes |
| State | Yes | No |
| When to use | "Is-a" relationship | "Can-do" capability |

## Choosing Between Abstract Class and Interface

### Use Abstract Class When:
```csharp
// Shared state and behavior
public abstract class Animal
{
    protected string Name { get; set; }  // Shared state
    protected int Age { get; set; }

    public void Eat()                    // Shared behavior
    {
        Console.WriteLine($"{Name} is eating");
    }

    public abstract void MakeSound();    // Custom behavior
}
```

### Use Interface When:
```csharp
// Capability contract
public interface IFlyable
{
    void Fly();
}

public interface ISwimmable
{
    void Swim();
}

// A bird can both fly and swim
public class Duck : IFlyable, ISwimmable
{
    public void Fly() => Console.WriteLine("Duck flies");
    public void Swim() => Console.WriteLine("Duck swims");
}
```

## Template Method Pattern with Abstract Classes
```csharp
public abstract class DataProcessor
{
    // Template method - defines algorithm skeleton
    public async Task ProcessData(string source)
    {
        var data = await LoadData(source);
        var cleaned = CleanData(data);
        var transformed = TransformData(cleaned);
        await SaveData(transformed);
        NotifyComplete();
    }

    protected abstract Task<string> LoadData(string source);
    protected abstract string CleanData(string data);
    protected abstract string TransformData(string data);
    protected abstract Task SaveData(string data);

    protected virtual void NotifyComplete()
    {
        Console.WriteLine("Processing complete");
    }
}
```
